[Partie 5] - Personnalisation de la vue : extensions Twig, barre lat�rale et Assetic
====================================================================================

Introduction
------------

Dans ce chapitre, nous allons continuer � construire la partie utilisateur de Symblog. Nous allons am�liore la page d'accueil pour aficher des informations sur les commentaires associ�s aux articles, ainsi qu'am�liorer les r�sultats potentiels de recherche via SEO (Search Engine Optimization : optimisation pour les moteurs de recherches) en ajoutant le titre des articles dans l'URL. Nous allons �galement commencer � travailler sur la barre lat�rale, en lui ajoutant 2 composants classiques; un nuage de tags et une section "Derniers commentaires".
Nous allons �galement explorer les diff�rents environnements dans Symfony2 et apprendre comment lancer Symblog dans l'environnement de production.  Le moteur de template Twig va �tre �tendu afin de proposer un nouveau filtre, et nous allons pr�senter Assetic pour la gestion des ressources externes.

La page d'accueil - Articles et commentaires
--------------------------------------------

Pour le moment, la page d'accueil se contente d'afficher les articles mais ne fournit pas d'informations concernant les commentaires qui leurs sont associ�s. Maintenant que nous avons construit une entit� ``Comment``, nous pouvons mettre � jour la page d'accueil afin d'ajouter ces informations.
Comme nous avons d�j� �tabli un lien entre les entit�s ``Blog`` et ``Comment``, nous savons que Doctrine 2 est capable de retrouver les commentaires associ�s � un article (souvenez vous que nous avons ajout� un membre ``$comments`` dans l'entit� ``Blog``). Mettons � jour le template de la page d'accueil dans ``src/Blogger/BlogBundle/Resources/views/Page/index.html.twig`` avec ce qui suit.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/index.html.twig #}

    {# .. #}
    
    <footer class="meta">
        <p>Comments: {{ blog.comments|length }}</p>
        <p>Posted by <span class="highlight">{{ blog.author }}</span> at {{ blog.created|date('h:iA') }}</p>
        <p>Tags: <span class="highlight">{{ blog.tags }}</span></p>
    </footer>
    
    {# .. #}

Nous avons utilis� le getter ``comments`` afin de r�cup�rer les commentaires de l'article, et avons ensuite pass� la liste dans le filtre Twig ``length``. Si vous regardez maintenant la page d'accueil via ``http://symblog.dev/app_dev.php/``, vous pourrez voir que le nombre de commentaires de chaque article est affich�.

Comme expliqu� plus haut, nous avons d�j� inform� Doctrine 2 que le membre ``$comments`` de l'entit� ``Blog`` estn associ� � l'entit� ``Comment``. Nous avons r�alis� cela dans le chapitre pr�c�dent avec la m�tadonn�e suivante dans l'entit� ``Blog``, dans ``src/Blogger/BlogBundle/Entity/Blog.php``.

.. code-block:: php

    // src/Blogger/BlogBundle/Entity/Blog.php

    /**
     * @ORM\OneToMany(targetEntity="Comment", mappedBy="blog")
     */
    protected $comments;

Nous savons ainsi que Doctrine 2 est conscient de la relation entre articles et commentaires, mais comment a-t-il remplit le membre ``$comments`` avec les entit�s ``Comment`` correspondantes ? Si vous vous souvenez de la m�thode que nous avons cr�� pour le ``BlogRepository`` (voir ci-dessous), vous pourrez voir que nous n'avons fait aucune s�lection des commentaires pour r�cup�rer les articles de la page d'accueil.

.. code-block:: php

    // src/Blogger/BlogBundle/Repository/BlogRepository.php
    
    public function getLatestBlogs($limit = null)
    {
        $qb = $this->createQueryBuilder('b')
                   ->select('b')
                   ->addOrderBy('b.created', 'DESC');

        if (false === is_null($limit))
            $qb->setMaxResults($limit);

        return $qb->getQuery()
                  ->getResult();
    }

N�anmoins, Doctrine 2 utilise un processus appel� chargement feignant (lazy loading) o� les entit�s ``Comment`` sont cherch�es dans la base de donn�e lorsque c'est n�cessaire, dans le cas pr�sent lors de l'appel � ``{{ blog.comments|length }}``. Nous pouvons d�montrer ce processus � l'aide de la abrre d'outils pour d�veloppeurs. Nous avons d�j� commenc� � parler de cet outil, et il est maintenant temps d'aborder l'une des ses fonctionnalit�s les plus puissantes, le profiler pour Doctrine 2. On se rend dans le profiler Doctrine 2 en cliquant sur le dernier icone de la barre d'outils. Le chiffre � c�t� indique le nombre de requ�tes ex�cut�es sur la base de donn�es pour l'actuelle requ�te HTTP.

.. image:: /_static/images/part_5/doctrine_2_toolbar_icon.jpg
    :align: center
    :alt: Barre d'outils pour d�veloppeurs - Ic�ne Doctrine 2

Si vous cliquez sur l'ic�ne Doctrine 2, des informations sur les requ�tes qui ont �t� ex�cut�es par Doctrine 2 sur la base de donn�es vous seront pr�sent�es.

.. image:: /_static/images/part_5/doctrine_2_toolbar_queries.jpg
    :align: center
    :alt: Barre d'outils pour d�veloppeurs - Requ�tes Doctrine 2

Comme vous pouvez le voir dans la capture d'�cran ci-dessus, il y a plusieurs requ�tes vers la base de donn�e qui sont execut�es lorsque la page d'accueil est charg�e. La seconde requ�te r�cup�re les articles dans la base de donn�e, et est ex�cut�e en r�ponse � l'appel de la m�thode
``getLatestBlogs()`` de la classe ``BlogRepository``. Apr�s cette requ�te, vous pouvez trouver plusieurs requ�tes qui extraient les commentaires depuis la base de donn�e, un article � la fois. On peut le voir gr�ce � ``WHERE t0.blog_id = ?`` dans chacune des requ�tes, o� le ``?`` est remplat� par la valeur du param�tre (l'identifiant de l'article). Chacune de ces requ�tes est li�e � un appel de ``{{ blog.comments }}`` dans le template de la page d'accueil. Chaque fois que cette fonction est effectu�e, Doctrine 2 va charger, parce que c'est n�cessaire ici et pas avant, et donc de mani�re feignante, les entit�s ``Comment`` associ�es � une entit� ``Blog``.

Bien que le lazy loading soit tr�s efficace pour r�cup�rer des entit�s depuis la base de donn�es, ce n'est pas toujours la mani�re la plus efficace de proc�der. Doctrine 2 fournit la possibilit� de ``joindre`` des entit�s reli�es entre elles lorsqu'une requ�te a lieu sur la base de donn�es. De cette mani�re, on peut extraire les entit� ``Blog`` et leurs entit�s ``Comment`` associ�es en une seule requ�te.
Mettez � jour le code du ``QueryBuilder`` de la classe ``BlogRepository`` dans ``src/Blogger/BlogBundle/Repository/BlogRepositoy.php`` pour joindre les commentaires.

.. code-block:: php

    // src/Blogger/BlogBundle/Repository/BlogRepositoy.php

    public function getLatestBlogs($limit = null)
    {
        $qb = $this->createQueryBuilder('b')
                   ->select('b, c')
                   ->leftJoin('b.comments', 'c')
                   ->addOrderBy('b.created', 'DESC');

        if (false === is_null($limit))
            $qb->setMaxResults($limit);

        return $qb->getQuery()
                  ->getResult();
    }

Si maintenant vous raffraichissez la page d'accueil et allez examiner la sortie de Doctrine 2 dans la barre d'outils, vous allez remarquer que le nombre de requ�tes a chut� de mani�re drastique. Vous pouvez �galement voir que la table de commentaires a �t� jointe � la table d'articles.

Le lazy loading et la jonction d'entit�s qui sont li�es sont deux concepts tr�s puissants, mais qui doivent �tre utilis�s correctement. L'�quilibre entre les deux doit �tre trouv� afin de permettre aux applications de fonctionner aussi efficacement que possible. Au premier abord, il simble attrayant de joindre toutes les entit�s li�es afin de ne jamais avoir � faire du lazy loading et � conserver un nombre faible de requ�tes vers la base de donn�es. Il est n�anmoins important de se souvenir que plus il y a d'informations � aller chercher dans la base de donn�es, plus les traitements � effectuer par Doctrine 2 pour cr�er les objets associ�s aux entit�s sont lourds. Plus de donn�es signifie �galement plus d'utilisation m�moire par le serveur pour stocker les objets.

Avant d'avancer, faisant un ajout mineur au template de la page d'accueil. Mettez � jour le template de la page d'accueil dans ``src/Blogger/BlogBundle/Resources/views/Page/index.html.twig`` pour ajouter un lien vers l'affichage des commentaires de l'article.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/index.html.twig #}

    {# .. #}
    
    <footer class="meta">
        <p>Comments: <a href="{{ path('BloggerBlogBundle_blog_show', { 'id': blog.id }) }}#comments">{{ blog.comments|length }}</a></p>
        <p>Posted by <span class="highlight">{{ blog.author }}</span> at {{ blog.created|date('h:iA') }}</p>
        <p>Tags: <span class="highlight">{{ blog.tags }}</span></p>
    </footer>
    
    {# .. #}
            
La barre lat�rale.
------------------

Actuellement, la barre lat�rale de Symblog est un peu vide. Nous allons la mettre � jour en lui ajoutant 2 composants, un nuage de tags et une liste des derniers commentaires.

Le nuage de tags
~~~~~~~~~~~~~~~~

Le nuage de tags montre les tags des articles, les plus populaires ayant plus d'importance visuelle � travers un affichage plus gros. Pour cel�, il nous faut un moyen de r�cup�rer tous les articles de tous les articles. Cr�ons de nouvelles m�thodes dans la classe ``BlogRepository`` pour cela. Mettez � jour la classe ``BlogRepository`` dans ``src/Blogger/BlogBundle/Repository/BlogRepository.php`` avec ce qui suit.

.. code-block:: php

    // src/Blogger/BlogBundle/Repository/BlogRepository.php

    public function getTags()
    {
        $blogTags = $this->createQueryBuilder('b')
                         ->select('b.tags')
                         ->getQuery()
                         ->getResult();

        $tags = array();
        foreach ($blogTags as $blogTag)
        {
            $tags = array_merge(explode(",", $blogTag['tags']), $tags);
        }

        foreach ($tags as &$tag)
        {
            $tag = trim($tag);
        }

        return $tags;
    }

    public function getTagWeights($tags)
    {
        $tagWeights = array();
        if (empty($tags))
            return $tagWeights;
        
        foreach ($tags as $tag)
        {
            $tagWeights[$tag] = (isset($tagWeights[$tag])) ? $tagWeights[$tag] + 1 : 1;
        }
        // Shuffle the tags
        uksort($tagWeights, function() {
            return rand() > rand();
        });
        
        $max = max($tagWeights);
        
        // Max of 5 weights
        $multiplier = ($max > 5) ? 5 / $max : 1;
        foreach ($tagWeights as &$tag)
        {
            $tag = ceil($tag * $multiplier);
        }
    
        return $tagWeights;
    }

Comme les tags sont stock�s dans la base de donn�e au format CSV (comma separated values, c'est � dire que chaque valeur est s�par�e de la pr�c�dente par une virgule), il nous faut un moyen de s�parer et de renvoyer le r�sultat sous la forme d'un tableau. C'est le r�le de ``getTags()``.
La m�thode ``getTagWeights()`` se sert ensuite du tableau de tafs pour calculer le poids (weight) de chaque tag � partir de son nombre d'occurences dans le tableau. Les tags sont �galement m�lang�s afin d'ajouter un peu d'al�atoire � leur affichage.

Maintenant que nous sommes capable de g�n�rer un nuage de tags, il faut l'afficher. Cr�ez une nouvelle action dans le ``PageController`` dans le fichier ``src/Blogger/BlogBundle/Controller/PageController.php`` pour g�rer la barre lat�rale.

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    
    public function sidebarAction()
    {
        $em = $this->getDoctrine()
                   ->getEntityManager();

        $tags = $em->getRepository('BloggerBlogBundle:Blog')
                   ->getTags();

        $tagWeights = $em->getRepository('BloggerBlogBundle:Blog')
                         ->getTagWeights($tags);

        return $this->render('BloggerBlogBundle:Page:sidebar.html.twig', array(
            'tags' => $tagWeights
        ));
    }

Cette action est tr�s simple, elle utilise les 2 nouvelles m�thodes du ``BlogRepository`` pour g�n�rer le nuage de tags, qu'elle passe ensuite en param�tres � la vue. Il nous faut maintenant cr�er cette vue, dans ``src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig``.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig #}
    
    <section class="section">
        <header>
            <h3>Tag Cloud</h3>
        </header>
        <p class="tags">
            {% for tag, weight in tags %}
                <span class="weight-{{ weight }}">{{ tag }}</span>
            {% else %}
                <p>There are no tags</p>
            {% endfor %}
        </p>
    </section>

Le template est �galement tr�s simple. Il traverse les diff�rents tags, en leur associant une classe CSS en fonction de leur poids. Dans cette boucle ``for`` un peu particuli�re, on acc�de aux couples ``cl�/valeur`` du tableau avec ``tag`` pour la cl� et ``weight`` comme valeur. Il existe plusieurs variations de comment utiliser une boucle ```for`` avec Twig disponible dans la ``documentation <http://twig.sensiolabs.org/doc/templates.html#for>`_.

Si vous regardez le principal template du ``BloggerBlogBundle`` dans ``src/Blogger/BlogBundle/Resources/views/layout.html.twig``, vous pourrez remarquer que nous avions plac� un �l�ment temporaire pour le bloc de la barre lat�rale. On peut maintenant le remplacer, en affichant la nouvelle action de la barre lat�rale. Souvenez vous que la fonction Twig ``render`` permet d'afficher le contenu d'une action d'un controlleur, dans le cas pr�sent l'action ``sidebar`` du controlleur ``Page``.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}

    {# .. #}

    {% block sidebar %}
        {% render "BloggerBlogBundle:Page:sidebar" %}
    {% endblock %}

Enfin, ajoutons de la CSS au nuage de tags. Cr�ez la nouvelle feuille de style dans 
``src/Blogger/BlogBundle/Resources/public/css/sidebar.css``.

.. code-block:: css

    .sidebar .section { margin-bottom: 20px; }
    .sidebar h3 { line-height: 1.2em; font-size: 20px; margin-bottom: 10px; font-weight: normal; background: #eee; padding: 5px;  }
    .sidebar p { line-height: 1.5em; margin-bottom: 20px; }
    .sidebar ul { list-style: none }
    .sidebar ul li { line-height: 1.5em }
    .sidebar .small { font-size: 12px; }
    .sidebar .comment p { margin-bottom: 5px; }
    .sidebar .comment { margin-bottom: 10px; padding-bottom: 10px; }
    .sidebar .tags { font-weight: bold; }
    .sidebar .tags span { color: #000; font-size: 12px; }
    .sidebar .tags .weight-1 { font-size: 12px; }
    .sidebar .tags .weight-2 { font-size: 15px; }
    .sidebar .tags .weight-3 { font-size: 18px; }
    .sidebar .tags .weight-4 { font-size: 21px; }
    .sidebar .tags .weight-5 { font-size: 24px; }

Comme nous avons ajout� une nouvelle feuille de style, il faut l'inclure. Mettez � jour le template principale du ``BloggerBlogBundle`` dans
``src/Blogger/BlogBundle/Resources/views/layout.html.twig`` avec ce qui suit.

.. code-block:: html
    
    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}

    {# .. #}
    
    {% block stylesheets %}
        {{ parent() }}
        <link href="{{ asset('bundles/bloggerblog/css/blog.css') }}" type="text/css" rel="stylesheet" />
        <link href="{{ asset('bundles/bloggerblog/css/sidebar.css') }}" type="text/css" rel="stylesheet" />
    {% endblock %}
    
    {# .. #}

.. note::

    Si vous n'utilisez pas les liens symboliques pour r�f�rencer les fichiers externes dans le r�pertoire ``web``, vous devez relancer la commande suivante afin de copier les nouveaux fichiers CSS.

    .. code-block:: bash

        $ php app/console assets:install web
        
Si vous mettez maintenant � jour la page d'accueil de Symblog, vous verrez que le nuage de tags est affich� dans la barre lat�rale. Afin que les tags soient affich�s avec diff�rents poids, vous devrez modifier les tags factices afin que certains soient plus utilis�s que d'autres.

Commentaires r�cents.
~~~~~~~~~~~~~~~~~~~~~

Maintenant que le nuage de tags est en place, ajoutons un composant pour les derniers commentaires � la barre lat�rale.

Il nous faut tout d'abord un moyen de r�cup�rer les derniers commentaires des articles. Nous allons pour cela ajouter une m�thode dans le ``CommentRepository`` situ� dans ``src/Blogger/BlogBundle/Repository/CommentRepository.php``.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Repository/CommentRepository.php

    public function getLatestComments($limit = 10)
    {
        $qb = $this->createQueryBuilder('c')
                    ->select('c')
                    ->addOrderBy('c.id', 'DESC');

        if (false === is_null($limit))
            $qb->setMaxResults($limit);

        return $qb->getQuery()
                  ->getResult();
    }

Maintenant, mettez � jour l'action de la barre lat�rale dans ``src/Blogger/BlogBundle/Controller/PageController.php`` afin de r�cup�rer les derniers commentaires et les fournir � la vue.

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/PageController.php
    
    public function sidebarAction()
    {
        // ..

        $commentLimit   = $this->container
                               ->getParameter('blogger_blog.comments.latest_comment_limit');
        $latestComments = $em->getRepository('BloggerBlogBundle:Comment')
                             ->getLatestComments($commentLimit);
    
        return $this->render('BloggerBlogBundle:Page:sidebar.html.twig', array(
            'latestComments'    => $latestComments,
            'tags'              => $tagWeights
        ));
    }

Vous remarquerez �galement que nous avons utilis� un nouveau param�tre appel� ``blogger_blog.comments.latest_comment_limit`` afin de limiter le nombre de commentaires � afficher. Pour cr�er ce param�tre, mettez � jour le fichier de configuration dans ``src/Blogger/BlogBundle/Resources/config/config.yml`` avec ce qui suit.

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/config.yml
    
    parameters:
        # ..

        # Blogger max latest comments
        blogger_blog.comments.latest_comment_limit: 10

Il faut enfin afficher les derniers commentaires dans le template de la barre lat�rale. Mettez � jour le template dans  ``src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig`` en y ajoutant ce qui suit.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig #}

    {# .. #}

    <section class="section">
        <header>
            <h3>Latest Comments</h3>
        </header>
        {% for comment in latestComments %}
            <article class="comment">
                <header>
                    <p class="small"><span class="highlight">{{ comment.user }}</span> commented on
                        <a href="{{ path('BloggerBlogBundle_blog_show', { 'id': comment.blog.id }) }}#comment-{{ comment.id }}">
                            {{ comment.blog.title }}
                        </a>
                        [<em><time datetime="{{ comment.created|date('c') }}">{{ comment.created|date('Y-m-d h:iA') }}</time></em>]
                    </p>
                </header>
                <p>{{ comment.comment }}</p>
                </p>
            </article>
        {% else %}
            <p>There are no recent comments</p>
        {% endfor %}
    </section>

Si vous mettez maintenant � jour le site, vous verrez que les derniers commentaires sont affich�s dans la barre lat�rale, juste en dessous du nuage de tags.

.. image:: /_static/images/part_5/sidebar.jpg
    :align: center
    :alt: Barre lat�rale - Nuage de tags et derniers commentaires.

Extensions Twig 
---------------

Pour le moment nous avons affich� les dates dans un format de date standard tel que `2011-04-21`. Une approche bien plus sympa serait d'afficher depuis combien de temps les commentaires ont �t� ajout�s, tel que `post� il y a 3 heures`. Nous pourrions ajouter une m�thode dans l'entit� ``Comment`` afin de r�aliser cela et changer les templates pour utiliser cette m�thode au lieu de ``{{ comment.created|date('Y-m-d h:iA') }}``.

Comme il est possible que l'on veuille utiliser cette fonctionnalit� � d'autres endroits, il est logique de sortir le code de l'entit� ``Comment``. Comme transformer la date est une t�che sp�cifique � la vue, nous devrions l'impl�menter en utilisant le moteur de template Twig. Twig nous permet en effet cela gr�ce � ses possibilit�s d'extensions.

Nous pouvons utiliser l'interface d'`extension <http://www.twig-project.org/doc/extensions.html>`_ de Twig pour �tendre les fonctionnalit�s par d�faut qu'il propose. Nous allons cr�er une extension qui nous fournira un nouveau filtre qui s'utilisera de la mani�re suivante :

.. code-block:: html
    
    {{ comment.created|created_ago }}
    
Cela affichera une date de cr�ation du commentaire de type `posted 2 days ago` pour `Post� il y a 2 jours`.

L'extension
~~~~~~~~~~~

Cr�ez un fichier pour l'extension Twig dans ``src/Blogger/BlogBundle/Twig/Extensions/BloggerBlogExtension.php`` et mettez le � jour avec le contenu suivant.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Twig/Extensions/BloggerBlogExtension.php

    namespace Blogger\BlogBundle\Twig\Extensions;

    class BloggerBlogExtension extends \Twig_Extension
    {
        public function getFilters()
        {
            return array(
                'created_ago' => new \Twig_Filter_Method($this, 'createdAgo'),
            );
        }

        public function createdAgo(\DateTime $dateTime)
        {
            $delta = time() - $dateTime->getTimestamp();
            if ($delta < 0)
                throw new \Exception("createdAgo is unable to handle dates in the future");

            $duration = "";
            if ($delta < 60)
            {
                // Seconds
                $time = $delta;
                $duration = $time . " second" . (($time > 1) ? "s" : "") . " ago";
            }
            else if ($delta <= 3600)
            {
                // Mins
                $time = floor($delta / 60);
                $duration = $time . " minute" . (($time > 1) ? "s" : "") . " ago";
            }
            else if ($delta <= 86400)
            {
                // Hours
                $time = floor($delta / 3600);
                $duration = $time . " hour" . (($time > 1) ? "s" : "") . " ago";
            }
            else
            {
                // Days
                $time = floor($delta / 86400);
                $duration = $time . " day" . (($time > 1) ? "s" : "") . " ago";
            }

            return $duration;
        }

        public function getName()
        {
            return 'blogger_blog_extension';
        }
    }

Cr�er l'extension est assez simple. On surcharge la m�thode ``getFilters()`` pour renvoyer autant de filtres que l'on souhaite. Dans le cas pr�sent, on a cr�� le filtre ``created_ago``. Ce filtre est ensuite enregistr� de mani�re � appeler la m�thode ``createdAgo``, qui se charge simplement de transformer un objet ``DateTime`` en une chaine de caract�res qui repr�sente la dur�e �coul�e depuis la valeur stock�e dans l'objet ``DateTime``.

Enregistrer l'extension
~~~~~~~~~~~~~~~~~~~~~~~

Pour rendre l'extension Twig disponible, il faut mettre � jour le fichier de services dans ``src/Blogger/BlogBundle/Resources/config/services.yml`` avec ce qui suit.

.. code-block:: yaml

    services:
        blogger_blog.twig.extension:
            class: Blogger\BlogBundle\Twig\Extensions\BloggerBlogExtension
            tags:
                - { name: twig.extension }

Vous pouvez voir que cel� enregistre un nouveau service en utilisant la classe d'extension ``BloggerBlogExtension`` que nous venons de cr�er.

Mettre � jour la vue
~~~~~~~~~~~~~~~~~~~~

Le nouveau filtre Twig est d�sormais pr�t � �tre utilis�. Mettons � jour la section des derniers commentaires de la barre lat�rale pour nous en servir. Mettez � jour le contenu du template de la barre lat�rale dans ``src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig`` avec ce qui suit :

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig #}

    {# .. #}
    
    <section class="section">
        <header>
            <h3>Latest Comments</h3>
        </header>
        {% for comment in latestComments %}
            {# .. #}
            <em><time datetime="{{ comment.created|date('c') }}">{{ comment.created|created_ago }}</time></em>
            {# .. #}
        {% endfor %}
    </section>

Si vous vous rendez maintenant sur la page d'accueil ``http://symblog.dev/app_dev.php/``, vous allez voir que les dates des derniers commentaires utilisent le filtre Twig pour afficher les dur�es depuis lesquelles ils ont �t� post�s.

Nous allons �galement mettre � jour les commentaires de la page d'affichage des articles afin d'utiliser l� aussi le nouveau filtre. Remplacez le contenu du template dans ``src/Blogger/BlogBundle/Resources/views/Comment/index.html.twig`` avec ce qui suit.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Comment/index.html.twig #}

    {% for comment in comments %}
        <article class="comment {{ cycle(['odd', 'even'], loop.index0) }}" id="comment-{{ comment.id }}">
            <header>
                <p><span class="highlight">{{ comment.user }}</span> commented <time datetime="{{ comment.created|date('c') }}">{{ comment.created|created_ago }}</time></p>
            </header>
            <p>{{ comment.comment }}</p>
        </article>
    {% else %}
        <p>There are no comments for this post. Be the first to comment...</p>
    {% endfor %}

.. tip::

    Il y a plusieurs extensions Twig utiles disponibles via la librarie 
    `Twig-Extensions <https://github.com/fabpot/Twig-extensions>`_  sur GitHub.
    Si vous cr�ez une extension utile, proposez une proposition d'ajout (pull request) dans ce d�p�t et il est possible qu'elle soit incluse afin que d'autres puissent s'en servir.

Slugification de l'URL
----------------------

Actuellement, l'URL de chaque article montre seulement l'identifiant de l'article. Bien que ce soit parfaitement acceptable d'un point de vue fonctionnel, c'est pas terrible d'un point de vue SEO (Search Engine Optimization: optimisation pour les moteurs de recherche). Par exemple, l'URL ``http://symblog.dev/1`` ne donne aucune information sur le contenu de l'article, alors que quelquechose comme ``http://symblog.dev/1/a-day-with-symfony2`` est beaucoup mieux de ce point de vue. Pour r�aliser cel�, il nous faut slugifier le titre des articles et nous en servir comme �l�ment de l'adresse. Slugifier le titre revient � enlever tous les caract�res non ASCII et les remplacer par un ``-``.

Mise � jour de la route
~~~~~~~~~~~~~~~~~~~~~~~

Pour commencer, modifions les r�gles de routage pour la page d'affichage des articles afin d'ajouter sa nouvelle composante ``slug``. Mettez � jour les r�gles de rouatge dans ``src/Blogger/BlogBundle/Resources/config/routing.yml``

.. code-block:: yaml

    # src/Blogger/BlogBundle/Resources/config/routing.yml
    
    BloggerBlogBundle_blog_show:
        pattern:  /{id}/{slug}
        defaults: { _controller: BloggerBlogBundle:Blog:show }
        requirements:
            _method:  GET
            id: \d+

Le controlleur
~~~~~~~~~~~~~~

Comme avec le composant d�j� existant ``id``, le nouvel �l�ment ``slug`` va �tre pass� � l'action du controlleur en argument. Il faut donc mettre � jour le controlleur dans ``src/Blogger/BlogBundle/Controller/BlogController.php`` afin de r�percuter ce changement.

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/BlogController.php

    public function showAction($id, $slug)
    {
        // ..
    }

.. tip::

    L'ordre dans lequel les arguments sont pass�s � l'action du controlleur n'a pas d'importance, seul leur nom compte. Symfony2 est capable d'associer les param�tres de routage avec la liste de param�tres pour nous. Bien que nous n'ayons pas utilis� pour le moment de valeurs par d�faut, cela vaut le coup de les mentionner ici. Si nous ajoutions un nouveau composant � la r�gle de routage, nous pourrions tr�s bien lui sp�cifier �galement une valeur par d�faut, � l'aide de l'option ``defauts``.

    .. code-block:: yaml

        BloggerBlogBundle_blog_show:
            pattern:  /{id}/{slug}/{comments}
            defaults: { _controller: BloggerBlogBundle:Blog:show, comments: true }
            requirements:
                _method:  GET
                id: \d+

    .. code-block:: php

        public function showAction($id, $slug, $comments)
        {
            // ..
        }

    En utilisant cette m�thode, les requ�tes � l'adresse ``http://symblog.dev/1/symfony2-blog`` m�neraient � avoir ``$comments`` � true dans ``showAction``.

Slugification du titre
~~~~~~~~~~~~~~~~~~~~~~

Comme on veut g�n�rer le slug � partir du titre de l'article, nous allons g�n�rer automatiquement cette valeur. Nous pourrions r�aliser cel� automatiquement � l'ex�cution sur le titre de l'article, mais � la place nous allons plut�t stocker le slug dans l'entit� ``Blog`` et le stocker dans la base de donn�es.

Mise � jour de l'entit� Blog
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ajoutons un nouveau membre � l'entit� ``Blog`` pour stocker le slug. Mettez � jour l'entit� ``Blog`` dans ``src/Blogger/BlogBundle/Entity/Blog.php``

.. code-block:: php

    // src/Blogger/BlogBundle/Entity/Blog.php

    class Blog
    {
        // ..

        /**
         * @ORM\Column(type="string")
         */
        protected $slug;

        // ..
    }

G�n�rez maintenant les accesseurs pour le nouveau membre ``$slug``. Comme avant, lancez la t�che :

.. code-block:: bash

    $ php app/console doctrine:generate:entities Blogger

Il est ensuite temps de mettre � jour le sch�ma de base de donn�e :

.. code-block:: bash

    $ php app/console doctrine:migrations:diff
    $ php app/console doctrine:migrations:migrate

Pour g�n�rer la valeur du slug, nous allons utiliser la m�thode slugify du Tutorial Symfony 1
`Jobeet <http://www.symfony-project.org/jobeet/1_4/Propel/en/08>`_. Ajoutez la m�thode ``slugify`` dans l'entit� ``Blog`` situ� dans ``src/Blogger/BlogBundle/Entity/Blog.php``

.. code-block:: php

    // src/Blogger/BlogBundle/Entity/Blog.php

    public function slugify($text)
    {
        // replace non letter or digits by -
        $text = preg_replace('#[^\\pL\d]+#u', '-', $text);

        // trim
        $text = trim($text, '-');

        // transliterate
        if (function_exists('iconv'))
        {
            $text = iconv('utf-8', 'us-ascii//TRANSLIT', $text);
        }

        // lowercase
        $text = strtolower($text);

        // remove unwanted characters
        $text = preg_replace('#[^-\w]+#', '', $text);

        if (empty($text))
        {
            return 'n-a';
        }

        return $text;
    }

Comme nous voulons g�n�rer automatiquement le slug � partir du titre, on peut g�n�rer le slug lorsque la valeur du titre est affect�e. Pour cel�, on peut mettre � jour l'accesseur ``setTitle`` pour mettre �galement � jour la valeur du slug. Mettez � jour l'entit� ``Blog`` dans
``src/Blogger/BlogBundle/Entity/Blog.php`` avec ce qui suit.

.. code-block:: php

    // src/Blogger/BlogBundle/Entity/Blog.php

    public function setTitle($title)
    {
        $this->title = $title;

        $this->setSlug($this->title);
    }

Maintenant mettez � jour la m�thode ``setSlug`` afin d'affecter une valeur `slugifi�e` � l'attribut slug.

.. code-block:: php

    // src/Blogger/BlogBundle/Entity/Blog.php

    public function setSlug($slug)
    {
        $this->slug = $this->slugify($slug);
    }

Maintenant rechargez les donn�es factices pour g�n�rer les slugs des articles.

.. code-block:: bash

    $ php app/console doctrine:fixtures:load

Mise � jour des routes g�n�r�es
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Il faut enfin mettre � jour les appels d�j� existants � la g�n�ration de route vers la page d'affichage des articles. Il y a plusieurs endroits o� cel� doit �tre mis � jour.

Ouvrez le template de la page d'accueil dans ``src/Blogger/BlogBundle/Resources/views/Page/index.html.twig`` et remplacez son contenu avec ce qui suit. Il y a 3 modifications de la route 
``BloggerBlogBundle_blog_show`` dans ce template. Les modifications ajoutent simplement le slug des titres des articles en param�tre de la fonction ``path``. 

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/index.html.twig #}

    {% extends 'BloggerBlogBundle::layout.html.twig' %}

    {% block body %}
        {% for blog in blogs %}
            <article class="blog">
                <div class="date"><time datetime="{{ blog.created|date('c') }}">{{ blog.created|date('l, F j, Y') }}</time></div>
                <header>
                    <h2><a href="{{ path('BloggerBlogBundle_blog_show', { 'id': blog.id, 'slug': blog.slug }) }}">{{ blog.title }}</a></h2>
                </header>
    
                <img src="{{ asset(['images/', blog.image]|join) }}" />
                <div class="snippet">
                    <p>{{ blog.blog(500) }}</p>
                    <p class="continue"><a href="{{ path('BloggerBlogBundle_blog_show', { 'id': blog.id, 'slug': blog.slug }) }}">Continue reading...</a></p>
                </div>
    
                <footer class="meta">
                    <p>Comments: <a href="{{ path('BloggerBlogBundle_blog_show', { 'id': blog.id, 'slug': blog.slug }) }}#comments">{{ blog.comments|length }}</a></p>
                    <p>Posted by <span class="highlight">{{ blog.author }}</span> at {{ blog.created|date('h:iA') }}</p>
                    <p>Tags: <span class="highlight">{{ blog.tags }}</span></p>
                </footer>
            </article>
        {% else %}
            <p>There are no blog entries for symblog</p>
        {% endfor %}
    {% endblock %}

De plus, une mise � jour doit �tre faite � la section Derniers commentaires de la barre lat�rale dans le template ``src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig``.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/Page/sidebar.html.twig #}

    {# .. #}

    <a href="{{ path('BloggerBlogBundle_blog_show', { 'id': comment.blog.id, 'slug': comment.blog.slug }) }}#comment-{{ comment.id }}">
        {{ comment.blog.title }}
    </a>

    {# .. #}

Enfin, l'action ``createAction`` du ``CommentController`` doit �tre mise � jour lorsqu'elle redirige vers la page d'affichage d'un article lorsqu'un commentaire a �t� post�. Mettez � jour le ``CommentController`` situ� dans ``src/Blogger/BlogBundle/Controller/CommentController.php`` avec ce qui suit.

.. code-block:: php

    // src/Blogger/BlogBundle/Controller/CommentController.php
    
    public function createAction($blog_id)
    {
        // ..

        if ($form->isValid()) {
            // ..
                
            return $this->redirect($this->generateUrl('BloggerBlogBundle_blog_show', array(
                'id'    => $comment->getBlog()->getId(),
                'slug'  => $comment->getBlog()->getSlug())) .
                '#comment-' . $comment->getId()
            );
        }

        // ..
    }

Maintenant si vous allez sur la page d'accueil ``http://symblog.dev/app_dev.php/`` et cliquez sur un des titres des articles, vous verrez que le slug des titres des articles est maintenant pr�sent � la fin de l'URL.

Environnements
------------

Les environnements sont � la fois une fonctionnalit� tr�s simple et tr�s puissante de Symfony2.  Vous n'en �tes peut �tre pas conscient, mais vous vous en servez depuis le tout premier chapitre de ce tutoriel. Avec les environnements, on peut configurer diff�rents aspects de Symfony2 et de l'application pour qu'elle tourne diff�remment selon des besoins sp�cifiques au cours du cycle de vie de l'application. Par d�faut, Symfony2 est configur� avec 3 environnement :

1. ``dev`` - Developpement
2. ``test`` - Test
3. ``prod`` - Production

Le r�le de ces environnements est inclus dans leur nom. Lorsque l'on d�veloppe une application, il est utile d'avoir la barre de d�bug � l'acran afin d'avoir des erreurs et des exceptions d�taill�es, alors qu'en production on ne veut rien de tout cela. En fait, afficher ces informations serait m�me une faille de s�curit� car de nombreux d�tails relatifs au comportement interne de l'application et du serveur seraient disponibles. En production, il serait plus judicieux d'afficher des pages d'erreur personnalis�es avec des messages simples, tout en stockant discr�tement les messages d'erreurs dans un fichier log. Il peut �galement �tre utile d'activer le cache afin que l'application tourne au maximum de ses capacit�s. En d�bug, l'activer serait un v�ritable cauchemar car il faudrait vider le cache � chaque modification ou presque, ce qui fait au final perdre plus de temps qu'il n'en fait gagner et peut �tre source d'erreurs.

Le dernier environnement, c'est l'environnement de test. Il est utilis� pour effectuer des tests sur l'application, tel que des tests unitaires ou fonctionnels. Nous n'avons pas parl� des tests pour le moment, mais ils seront abord�s en d�tails dans le chapitre suivant.

Controlleur de facade
~~~~~~~~~~~~~~~~~~~~~

Pour le moment dans ce tutoriel, nous avons uniquement utilis� l'environnement de ``d�veloppement``, ce qui nous avons pr�cis� en utilisant le controlleur de facade ``app_dev.php`` lorsque nous avons fait des requ�tes vers symblog, par exemple ``http://symblog.dev/app_dev.php/about``. 
Si vous regardez le contenu du controlleur de facade de l'environnement de d�veloppement dans ``web/app_dev.php``, vous y verrez la ligne suivante :

.. code-block:: php

    $kernel = new AppKernel('dev', true);

Cette ligne est celle qui fait d�marrer Symfony2. Elle cr�e une nouvelle instance de l'``AppKernel`` de Symfony2, et opte pour l'environnement ``dev``.

En comparaison, si vous regardez le controlleur de fa�ade de l'environnement de ``production`` dans ``web/app.php``, vous y verrez :

.. code-block:: php

    $kernel = new AppKernel('prod', false);

Vous pouvez voir que l'environnement  ``prod`` est fourni en param�tre � l'``AppKernel`` dans cette instance.

L'environnement de test n'a pas de controlleur de fa�ade, car il n'est pas cens� �tre utilis� dans un navigateur. C'est pourquoi il n'y a pas de fichier ``app_test.php``.

Param�tres de configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Nous avons vu plus haut comment les controlleurs de fa�ade sont utilis�s pour changer l'environnement dans lequel l'application tourne. Nous allons maintenant regarder comment les diff�rents param�tres sont modifi�s lorsque l'on utilise tel ou tel environnement. Si vous regardez les fichiers dans ``app/config``, vous y verrez plusieurs fichiers ``config.yml``. Plus pr�cis�mment, il y a un fichier de configuration principal, ``config.yml``, et 3 autres qui sont suffix�s du nom de l'environnement; ``config_dev.yml``, ``config_test.yml`` et ``config_prod.yml``.
Chacun de ces fichiers est charg� selon l'environnement courant. Si nous ouvrons le fichier ``config_dev.yml``, nous y verrons les lignes suivantes en ent�te :

.. code-block:: yaml

    imports:
        - { resource: config.yml }

La directive ``imports`` va permettre d'importer le contenu du fichier ``config.yml`` � l'int�rieur de celui l�. La m�me directive ``import`` peut �tre trouv�e au d�but des 2 autres fichiers de configuration ``config_test.yml`` et ``config_prod.yml``. 
L'inclusion d'un ensemble commun de param�tres de configuration d�finis dans ``config.yml`` permet d'avoir des valeurs sp�cifiques pour ces param�tres selon les environnements. On peut voir dans le fichier de configuration de l'environnement de d�veloppement ``app/config/config_dev.yml`` les lignes suivantes, qui configurent l'utilisation de la barre de d�bug :

.. code-block:: yaml

    # app/config/config_dev.yml
    
    web_profiler:
        toolbar: true

Ce param�tre est absent dans le fichier de configuration de l'environnement de production car nous ne voulons pas que la barre d'outils soit affich�e.

Fonctionner en production
~~~~~~~~~~~~~~~~~~~~~~~~~

Nous allons maintenant voir notre site tourner dans l'environnement de production. Pour cela, il faut tout d'abord vider le cache, � l'aide d'une commande Symfony2 :

.. code-block:: bash

    $ php app/console cache:clear --env=prod

Maintenant rendez vous � l'adresse ``http://symblog.dev/``. Remarquez qu'il manque le controlleur de fa�ade ``app_dev.php``.

.. note::
    
    Pour ceux qui utilisent les hotes dynamiques virtuels comme dans le lien de la partie 1, il faudra ajouter ce qui suit dans le fichier .htaccess dans ``web/.htaccess``.
    
    .. code-block:: text
    
        <IfModule mod_rewrite.c>
            RewriteBase /
            # ..
        </IfModule>
        

Vous allez remarquer que le site est presque identique, mais un certain nombre d'�l�ments sont diff�rents. La barre de d�bug a disparue et les messages d'erreur d�taill�s ne sont plus affich�s : essayez de vous rendre � l'adresse ``http://symblog.dev/999`` pour vous en assurer.

.. image:: /_static/images/part_5/production_error.jpg
    :align: center
    :alt: Production - Erreur 404
    
Les messages d'exceptions d�taill�s ont �t� remplaces par un message plus simple, qui informe l'utilisateur qu'un probl�me a eu lieu. Ces �crans d'exceptions peuvent �tre configur�s pour s'accorder avec le th�me visuel de votre application. Nous reviendrons sur ce sujet dans un futur chapitre.

Vous pouvez �galement remarquer que le fichier ``app/logs/prod.log`` se remplit avec des informations sur l'ex�cution de l'application. C'est un aspect int�ressant lorsque vous aurez des probl�mes en production mais qu'il n'y aura plus les erreurs et exceptions de l'environnement de d�veloppement.

.. tip::

    Comment la requ�te depuis ``http://symblog.dev/`` a r�ussi � emmener jusqu'au fichier ``app.php``? Je suis s�r que vous avez tous d�j� cr�� des fichiers tels que ``index.html`` et ``index.php`` comme index de sites, mais ``app.php`` est moins courant; c'est gr�ce � une des r�gles du fichier ``web/.htaccess`` :

    .. code-block:: text

        RewriteRule ^(.*)$ app.php [QSA,L]

    On peut voir que cette line contient une expression r�guli�re qui associe n'importe quel texte via ``^(.*)$`` et le fournit � ``app.php``.

    Vous �tes peut �tre sur un serveur Apache qui ne dispose pas de ``mod_rewrite.c`` activ�. Dans ce cas, vous pouvez simplement ajouter ``app.php`` � l'URL, tel que ``http://symblog.dev/app.php/``.

Bien que nous ayions couvert les bases de l'environnement de production, nous n'avons pas parl� de plusieurs �l�ments li�s � l'environnement de production, tel que la personnalisation des pages d'erreurs et le d�ploiement vers un serveur de production � l'aide d'outils tel que `capifony <http://capifony.org/>`_. Nous reviendrons plus tard sur ces sujets dans un chapitre ult�rieur.

Cr�ation de nouveaux environnements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Il est enfin int�ressant de savoir que vous pouvez cr�er vos propres environnements facilement dans Symfony2. Par exemple, vous pouvez avoir envie d'avoir un environnement qui tourne sur le serveur de production mais affiche certaines informations de d�bug tel que les exceptions. Cela permettrait � la plateforme d'�tre test�e manuellement sur le serveur de production, car les configurations des serveurs de d�veloppement et de production peuvent (et c'est souvent le cas) �tre diff�rentes.

Bien que la cr�ation d'un nouvel environnement soit une t�che simple, elle va au del� du cadre de ce tutoriel. Il y a un excellent article
`article <http://symfony.com/doc/current/cookbook/configuration/environments.html>`_ dans le livre de recettes de Symfony2 qui couvre ce sujet.

Assetic
-------

La distribution standard de Symfony2 est accompagn�e d'une librairie de gestion des fichiers externes (les assets) appel�e `Assetic <https://github.com/kriswallsmith/assetic>`_. Cette librairie a �t� d�velopp�e par `Kris Wallsmith <https://twitter.com/#!/kriswallsmith>`_ et a �t� inspir�e par la librairie Python `webassets
<http://elsdoerfer.name/files/docs/webassets/>`_.


Assetic se charge de 2 aspects de la gestion des fichiers externes, les assets tels que images, feuilles de style ou fichiers JavaScript, et les filtres qui peuvent �tre appliqu�s sur ces assets. Ces filtre permettent de r�aliser des t�ches utiles tel que la minification des fichiers CSS ou JavaScript, ou bien passer les fichiers `CoffeeScript <http://jashkenas.github.com/coffee-script/>`_ � travers un compilateur, et comibiner les assets ensemble afin de r�duire le nombre de requ�tes HTTP faites vers le serveur.

Nous avons jusqu'� pr�sent utilis� la fonction Twig ``asset`` afin d'inclure les fichiers externes, de la mani�re suivante :

.. code-block:: html
    
    <link href="{{ asset('bundles/bloggerblog/css/blog.css') }}" type="text/css" rel="stylesheet" />

Ces appels � la fonction ``asset`` vont �tre remplac�s par Assetic.

Assets
~~~~~~

La librairie Assetic d�crit un asset de la mani�re suivante :

`Un asset Assetic est quelquechose avec un contenu filtrable qui peut �tre charg� et d�charg�. Cela inclus �galement les m�tadonn�es, certaines pouvant �tre manipul�es et certaines �tant fix�es.`

Plus simplement, les assets sont des ressources que l'application utilise tel que les feuilles de style et les images.

Feuilles de style
.................

Commen�ons par remplacer les appels actuels � la fonction ``asset`` pour les feuilles de styles dans le template principal du  ``BloggerBlogBundle``. Mettez � jour le contenu du template situ� dans ``src/Blogger/BlogBundle/Resources/views/layout.html.twig`` avec ce qui suit :

.. code-block:: html
    
    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}
    
    {# .. #}

    {% block stylesheets %}
        {{ parent () }}
        
        {% stylesheets 
            '@BloggerBlogBundle/Resources/public/css/*'
        %}
            <link href="{{ asset_url }}" rel="stylesheet" media="screen" />
        {% endstylesheets %}
    {% endblock %}
    
    {# .. #}

Nous avons remplac� les 2 pr�c�dents liens vers les fichiers CSS avec des fonctionnalit�s Assetic. En utilisant ``stylesheets`` depuis Assetic, nous avons pr�cis� que toutes les feuilles de style dans ``src/Blogger/BlogBundle/Resources/public/css`` doivent �tre combin�es en un fichier avant d'�tre incluses. Combiner plusieurs fichiers est une mani�re simple mais efficace d'opitmiser le nombre de fichiers n�cessaires � votre site web. Moins de fichiers signifie moins de requ�tes HTTP vers le serveur. Bien que nous ayons utilis� ``*`` pour pr�ciser tous les fichiers du r�pertoire ``css``, nous aurions �galement pu lister chaque fichier individuellement :

.. code-block:: html
    
    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}
    
    {# .. #}

    {% block stylesheets %}
        {{ parent () }}
        
        {% stylesheets 
            '@BloggerBlogBundle/Resources/public/css/blog.css'
            '@BloggerBlogBundle/Resources/public/css/sidebar.css'
        %}
            <link href="{{ asset_url }}" rel="stylesheet" media="screen" />
        {% endstylesheets %}
    {% endblock %}

    {# .. #}
    
Le r�sultat final dans les 2 cas est le m�me. La premi�re option qui utilise ``*`` assure que les nouveaux fichiers CSS ajout�s dans le r�pertoire seront ajout�s et combin�s dans le fichier CSS d'Assetic. Cela n'est toutefois pas forc�ment le comportement que l'on souhaite avoir, donc utilisez l'une ou l'autre des m�thodes selon vos besoins.
    
Si vous regardez la sortie HTML via ``http://symblog.dev/app_dev.php/``, vous verrez que les fichiers CSS ont �t� inclus de la mani�re suivante (remarquez que nous sommes retourn� dans l'environnement de d�veloppement).

.. code-block:: html
    
    <link href="/app_dev.php/css/d8f44a4_part_1_blog_1.css" rel="stylesheet" media="screen" />
    <link href="/app_dev.php/css/d8f44a4_part_1_sidebar_2.css" rel="stylesheet" media="screen" />

Au premier abord, vous vous demandez peut �tre quels sont ces 2 fichiers, car nous avons dit plus haut qu'Assetic combinerait les fichiers en 1 fichier. C'est parce que nous sommes dans l'environnement de ``developpement``. On peut demander � Assetic de fonctionner en mode non d�bug We can ask Assetic to run in non-debug mode en mettant le param�tre ``debug`` � false de la mani�re suivante :

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}
    
    {# .. #}
    
    {% stylesheets 
        '@BloggerBlogBundle/Resources/public/css/*'
        debug=false
    %}
        <link href="{{ asset_url }}" rel="stylesheet" media="screen" />
    {% endstylesheets %}

    {# .. #}
    
Si vous regardez maintenant le HTML, vous y verrez ceci :

.. code-block:: html

    <link href="/app_dev.php/css/3c7da45.css" rel="stylesheet" media="screen" />

Si vous regardez le contenu de ce fichier, vous verrez que les 2 fichiers CSS ``blog.css`` et ``sidebar.css`` ont �t� combin�s en 1 fichier. Le nom de fichier utilis� pour le fichier g�n�r� est produit al�atoirement par Assetic. Si vous voulez controller le nom du fichier g�n�r�, utilisez l'option ``output`` comme suit :

.. code-block:: html

    {% stylesheets 
        '@BloggerBlogBundle/Resources/public/css/*'
        output='css/blogger.css'
    %}
        <link href="{{ asset_url }}" rel="stylesheet" media="screen" />
    {% endstylesheets %}

Avant de continuer, supprimez le param�tre ``debug`` de l'exemple pr�c�dent, car nous voulons revenir au comportement par d�faut sur les assets.

Nous devons �galement mettre � jour le template de base de l'application, dans ``app/Resources/views/base.html.twig``.

.. code-block:: html

    {# app/Resources/views/base.html.twig #}
    
    {# .. #}
    
    {% block stylesheets %}
        <link href='http://fonts.googleapis.com/css?family=Irish+Grover' rel='stylesheet' type='text/css'>
        <link href='http://fonts.googleapis.com/css?family=La+Belle+Aurore' rel='stylesheet' type='text/css'>
        {% stylesheets 
            'css/*'
        %}
            <link href="{{ asset_url }}" rel="stylesheet" media="screen" />
        {% endstylesheets %}
    {% endblock %}
    
    {# .. #}
    
JavaScripts
...........

Bien que nous n'ayions pas actuellement de fichiers JavaScript dans notre application, leur utilisation via Assetic est tr�s semblable � celle des feuilles de style :

.. code-block:: html

    {% javascripts 
        '@BloggerBlogBundle/Resources/public/js/*'
    %}
        <script type="text/javascript" src="{{ asset_url }}"></script>
    {% endjavascripts %}

Filtres
~~~~~~~

La vrai puissance d'Assetic vient de ses filtres. Les filtres peuvent �tre appliqu�s � des assets ou � un ensemble d'assets. Il y a un grand nombre de filtres � l'int�rieur de la librairie de base, qui r�alisent les taches courantes suivantes :

1. ``CssMinFilter``: minifaction de la CSS
2. ``JpegoptimFilter``: optimisation des fichiers JPEGs
3. ``Yui\CssCompressorFilter``: compression de fichiers CSS � l'aide de l'outil YUI compressor
4. ``Yui\JsCompressorFilter``: compression de fichiers  JavaScript � l'aide de l'outil YUI compressor
5. ``CoffeeScriptFilter``: compile CoffeeScript en JavaScript

Une liste compl�te des filtres disponible se trouve dans le 
`Readme Assetic <https://github.com/kriswallsmith/assetic/blob/master/README.md>`_.

Plusieurs de ces filtres passent en fait la main � un autre programme ou � une autre librairie, tel que YUI Compressor, donc il est possible que vous ayiez � installer ou configurer les librairies n�cessaires pour utiliser certains filtres.

T�l�chargez `YUI Compressor <http://yuilibrary.com/download/yuicompressor/>`_, d�compressez l'archive et copiez les fichiers du r�pertoire ``build`` dans
``app/Resources/java/yuicompressor-2.4.6.jar``. Cela suppose que vous ayiez t�l�charg� la version ``2.4.6``, sinon changez le num�ro de version en cons�quences.

Nous allons ensuite configurer un filtre Assetic pour minifier la CSS � l'aide de YUI Compressor. Mettez � jour la configuration de l'application dans  ``app/config/config.yml`` avec le contenu suivant :

.. code-block:: yaml
    
    # app/config/config.yml
    
    # ..

    assetic:
        filters:
            yui_css:
                jar: %kernel.root_dir%/Resources/java/yuicompressor-2.4.6.jar
    
    # ..
    
Nous venons de configurer un filtre ``yui_css`` qui va utiliser l'ex�cutable Java de l'outil YUI Compressor, que nous allons placer dans le r�pertoire des ressources de l'application. Afin d'utiliser ce filtre, il faut lui pr�ciser avec quels assets s'en servir. Mettez � jour le template dans ``src/Blogger/BlogBundle/Resources/views/layout.html.twig`` pour utiliser le filtre ``yui_css``.

.. code-block:: html

    {# src/Blogger/BlogBundle/Resources/views/layout.html.twig #}

    {# .. #}
    
    {% stylesheets 
        '@BloggerBlogBundle/Resources/public/css/*'
        output='css/blogger.css'
        filter='yui_css'
    %}
        <link href="{{ asset_url }}" rel="stylesheet" media="screen" />
    {% endstylesheets %}

    {# .. #}

Si vous rafraichissez la page d'accueil du site Symblog et regardez les fichiers g�n�r�s par Assetic, vous verrez qu'ils ont �t� minifi�s. Bien que la minification soit une bonne id�e sur un serveur de production, elle peut rendre le d�buggage difficile, en particulier lorsque le Javascript est minifi�. On peut la d�sactiver pour l'environnement ``development`` en pr�fixant le filtre avec un  ``?`` de la mani�re suivante.

.. code-block:: html
    
    {% stylesheets 
        '@BloggerBlogBundle/Resources/public/css/*'
        output='css/blogger.css'
        filter='?yui_css'
    %}
        <link href="{{ asset_url }}" rel="stylesheet" media="screen" />
    {% endstylesheets %}

G�n�ration des assets pour la production
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

En production, on peut g�n�rer les fichiers d'assets gr�ce � Assetic afin qu'ils deviennent de vrais fichiers pr�ts � �tre utilis�s sur le serveur web. Le processus de cr�ation des assets avec Assetic pour chaque adresse peut �tre assez long, en particulier si des filtres sont appliqu�s aux assets. Le sauvegarder de mani�re d�finitive pour la production assure qu'Assetic ne sera pas utilis� pour manipuler les assets, mais seulement pour fournir les assets pr�-trait�s. Lancez la commande suivante pour conserver les fichiers assets trait�s sur le disque :

.. code-block:: bash

    $ app/console --env=prod assetic:dump

Vous pouvez remarquer que plusieurs fichiers CSS ont �t� g�n�r�s dans le r�pertoire ``web/css``. Si vous lancez Symblog dans l'environnement de production, vous verrez que les fichiers proviennent directement de ce r�pertoire.

.. note::

    Si vous stockez les fichiers assets sur le disque mais souhaitez retourner dans l'environnement de d�veloppement, vous devrez supprimer les fichiers cr�� dans le r�pertoire  ``web/`` pour permettre � Assetic de les recr�er.

Lecture additionnelle    
~~~~~~~~~~~~~~~~~~~~~

Nous avons seulement abord� une fraction des possibilit�s offertes par Assetic. Il y a plus de ressources en ligne, en particulier dans le livre de recettes de Symfony2, en particulier (mais en anglais) :

`How to Use Assetic for Asset Management <http://symfony.com/doc/current/cookbook/assetic/asset_management.html>`_

`How to Minify JavaScripts and Stylesheets with YUI Compressor <http://symfony.com/doc/current/cookbook/assetic/yuicompressor.html>`_

`How to Use Assetic For Image Optimization with Twig Functions <http://symfony.com/doc/current/cookbook/assetic/jpeg_optimize.html>`_

`How to Apply an Assetic Filter to a Specific File Extension <http://symfony.com/doc/current/cookbook/assetic/apply_to_option.html>`_

Il y a �galement plusieurs bons articles de `Richard Miller <https://twitter.com/#!/mr_r_miller>`_ tel que :

`Symfony2: Using CoffeeScript with Assetic <http://miller.limethinking.co.uk/2011/05/16/symfony2-using-coffeescript-with-assetic/>`_

`Symfony2: A Few Assetic Notes <http://miller.limethinking.co.uk/2011/06/02/symfony2-a-few-assetic-notes/>`_

`Symfony2: Assetic Twig Functions <http://miller.limethinking.co.uk/2011/06/23/symfony2-assetic-twig-functions/>`_

.. tip::

    Il est � noter �galement que Richard Miller a �galement de nombreux articles tr�s int�ressant dans de nombreux dommaines de Symfony2, tel que l'injection de d�pendances, les services ainsi que les d�j� mentionn�s guides sur Assetic. Cherchez les articles tagg�s avec `symfony2 <http://miller.limethinking.co.uk/tag/symfony2/>`_

Conclusion
----------

Nous avons couvert plusieurs nouveaux dommaines de Symfony2, tel que les environnements et comment utiliser la librairie Assetic. Nous avons �galement am�lior� la page d'accueil, et ajout� plusieurs composants � la barre lat�rale.

Dans le prochain chapitre, nous aller passer aux tests. Nous parlerons � la fois des tests unitaires et des tests fonctionnels avec PHPUnit. Nous verrons comment Symfony2 aide grandement � l'�criture des tests avec plusieurs classes pour faciliter l'�criture des tests fonctionnels qui simulent des requ�tes, permettre de remplir les formulaires, cliquent sur les liens et nous permettent d'inspecter les r�ponses obtenues.
