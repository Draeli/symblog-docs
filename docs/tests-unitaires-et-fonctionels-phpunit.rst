[Partie 6] - Les tests unitaires et fonctionnels avec PHPUnit
====================================================

Introduction
--------

Jusqu'� pr�sent, nous avons explor� une grande quantit� d'�l�ments de base essentiels de Symfony2. Avant de continuer � ajouter des fonctionnalit�s � Symblog, il est temps de commencer � parler des tests. Nous allons regarder comment tester les fonctions individuellement gr�ce aux tests unitaires, puis regarder comment nous assurer que plusieurs composants fonctionnent bien ensemble gr�ce aux tests unitaires. Nous allons utiliser la librairie de tests `PHPUnit <http://www.phpunit.de/manual/current/en/>`_ car elle est au centre des tests dans Symfony2. Comme le sujet des tests est tr�s large, il sera abord� � nouveau dans d'autres chapitres par la suite. A la fin de ce chapitre nous aurons �crit plusieurs tests, � la fois unitaires et fonctionnels. Nous aurons simul� des requ�tes du navigateur, rempli des formulaires, et v�rifi� la r�ponse pour nous assurer que les pages s'affichent correctement. Nous allons �galement regarder quel pourcentage du code de l'application couvrent ces tests.


Les tests dans Symfony2
-------------------

`PHPUnit <http://www.phpunit.de/manual/current/en/>`_  est devenu le standard pour l'�criture des tests en PHP, donc si vous ne connaissez pas cette librairie, l'apprendre vous sera �galement utile pour vos autres projets PHP. N'oubliez �galement pas que la plupart des concepts abord�s dans ce chapitre sont ind�pendant du langage, et pourront ainsi �tre transf�r�s dans les autres langages que vous pourrez �tre amen�s � utiliser.

.. tip::

    Si vous comptez �crire vos propres bundles open source pour Symfony2, vous avez plus de chances de gagner en popularit� si votre bundle est bien test� (et document�). Regardez les bundles d�j� existant pour Symfony2 chez `KnpBundles <http://bundles.knplabs.org/fr>`_.

Tests unitaires
~~~~~~~~~~~~~~~

Les tests unitaires ont pour mission d'assurer que des unit�s individuelles de code fonctionnent correctement lorsqu'elles sont utilis�es de mani�re isol�e. Dans un cadre objet tel que celui de Symfony2, une unit� serait une classe et ses m�thodes. Par exemple, nous pourrions �crire les test pour les classes des entit�s ``Blog`` et ``Comment``. Lorsque l'on �crit des tests unitaires, les tests devraient �tre �crits ind�pendemment des autres tests, c'est � dire que le r�sultat du test B ne devrait pas d�pendre du r�sultat du test A. Il est utile, lorsque l'on fait des tests unitaires, de pouvoir cr�er des faux objets (des objets mock) qui facilitent les tests unitaires lorsqu'il y a des d�pendances. L'utilisation des mocks permet �galement de simuler un appel de fonction plut�t que l'ex�cuter; c'est par exemple utile pour simuler une classe qui fait appel � une librairie ext�rieure. La classe de l'API peut utiliser une couche de transport pour communiquer avec la librairie externe. On peut "mocker" la m�thode de requ�te de la couche de transport pour simuler les r�sultats que l'on veut, plut�t que de faire appel � la librairie ext�rieur (dont l'utilisateur dans un cadre isol� peut �tre p�nible � mettre en place, en plus d'�tre hors sujet, et peut mener � des erreurs lors des mises � jour de librairies, et ainsi de suite).
Les tests unitaires ne testent pas que les composants fonctionnent bien ensemble, c'est un domaine couvert par le th�me suivant, les tests fonctionnels.

Tests fonctionnels
~~~~~~~~~~~~~~~~~~

Les tests fonctionnels v�rifient l'int�gration des diff�rents composants � l'int�rieur de l'application, tel que le routage, les controlleurs et les vues. Les tests fonctionnels sont similaires aux tests manuels que vous pouvez effectuer vous m�me en lan�ant le navigateur sur la page d'accueil du site, en cliquant sur le lien d'un article et en v�rifiant que c'est le bon article qui s'affiche.
Les tests fonctionnels fournissent la possibilit� d'automatiser ce processus. Symfony2 propose plusieurs classes tr�s utiles pour les tests fonctionnels, tel qu'un ``Client`` capable d'effectuer des requ�tes vers les pages et soumettre des formulaire, et un navigateur de DOM que l'on peut utiliser pour analyser la r�ponse du client.

.. tip::

    Il y a plusieurs processus de d�veloppement logiciel dirig�s par les tests. On peut citer le TDD (Test Driven Development, d�veloppement orient� tests) et BDD (Behavioral Driven Development, d�veloppement orient� comportement). Bien que ces th�mes aillent au dela du but de ce tutoriel, il est bon que vous ayiez connaissance de la librairie �crit� par `everzet <https://twitter.com/#!/everzet>`_,  `Behat <http://behat.org/>`_, qui facilite le BDD. Il y a �galement le `BehatBundle <http://docs.behat.org/bundle/index.html>`_ disponible pour Symfony2, qui permet d'int�grer facilement Behat dans vos projets Symfony2.

PHPUnit
-------

Comme dit plus haut, les tests sont �crits dans Symfony2 avec PHPUnit. Vous devrez l'installer afin de pouvoir lancer ces tests et les tests de ce chapitre. Pour des instructions  d'`installation d�taill�e <http://www.phpunit.de/manual/current/en/installation.html>`_ rendez vous sur la documentation officielle du site de PHPUnit. Pour lancer les tests dans Symfony2, il vaut faut PHPUnit 3.5.11 ou une version plus r�cente. PHPUnit  est une librairie de tests tr�s compl�te, donc des r�f�rences � la documentation officielle auront lieu lorsque des d�tails suppl�mentaires peuvent �tre apport�s.

Assertions
~~~~~~~~~~

L'�criture de tests s'occupe de v�rifier que le r�sultat test du test est bien �gal � celui attendu. Il y a plusieurs m�thodes d'assertion disponibles dans PHPUnit pour nous assister dans cette t�che. Quelques unes des assertions les plus courantes sont list�es ci-dessous.

.. code-block:: php

    // Check 1 === 1 is true
    $this->assertTrue(1 === 1);

    // Check 1 === 2 is false
    $this->assertFalse(1 === 2);

    // Check 'Hello' equals 'Hello'
    $this->assertEquals('Hello', 'Hello');

    // Check array has key 'language'
    $this->assertArrayHasKey('language', array('language' => 'php', 'size' => '1024'));

    // Check array contains value 'php'
    $this->assertContains('php', array('php', 'ruby', 'c++', 'JavaScript'));

Une liste compl�te des 
`assertions <http://www.phpunit.de/manual/current/en/writing-tests-for-phpunit.html#writing-tests-for-phpunit.assertions>`_
est disponible dans la documentation de PHPUnit.

Lancer les tests de Symfony2
----------------------

Avant de commencer � �crire des tests, regardons comment lancer des tests dans Symfony2. On peut sp�cifier � PHPUnit de se lancer avec un fichier de configuration particulier. Dans notre projet Symfony2, ce fichier est ``app/phpunit.xml.dist``. Comme le nom du fichier est suffix� avec ``.dist``, vous devez copier son contenu dans un fichier ``app/phpunit.xml``.

.. tip::

    Si vous utilisez un gestionnaire de version tel que Git, vous devriez ajouter le nouveau fichier ``app/phpunit.xml`` dans la liste des fichiers � ignorer.

Si vous regardez maintenant le contenu du fichier de configuration PHPUnit, vous y verrez ce qui suit :

.. code-block:: xml

    <!-- app/phpunit.xml -->
    
    <testsuites>
        <testsuite name="Project Test Suite">
            <directory>../src/*/*Bundle/Tests</directory>
            <directory>../src/*/Bundle/*Bundle/Tests</directory>
        </testsuite>
    </testsuites>

Ces param�tres configurent des r�pertoires qui font partie de notre ensemble de test. Lorsqu'on lance PHPUnit, il va chercher dans ces r�pertoire s'il existe des tests � lancer. On peut �galement lui fournir des param�tres de ligne de commande additionnels pour lancer les tests seulement sur un r�pertoire particulier, plut�t que sur une suite de tests compl�te. Nous verrons comment faire cel� un peu plus tard.

Vous pouvez �galement remarquer que ce fichier sp�cifie le fichier de d�marrage ``app/bootstrap.php.cache``. Ce fichier est utilis� par PHPUnit pour obtenir des param�tres de l'environnement de test.

.. code-block:: xml

    <!-- app/phpunit.xml -->
    
    <phpunit
        bootstrap                   = "bootstrap.php.cache" >

.. tip::

    Pour plus d'informations concernant la configuration de PHPUnit � l'aide d'un fichier XML, reportez vous � la 
    `documentation de PHPUnit <http://www.phpunit.de/manual/current/en/organizing-tests.html#organizing-tests.xml-configuration>`_.

Lancer les tests
----------------

Comme nous avons utilis� le g�n�rateur de Symfony2 pour cr�er le ``BloggerBlogBundle`` dans le chapitre 1, une classe de test pour le controlleur par d�faut  ``DefaultController`` a �galement �t� cr�e. On peut ex�cuter ce test en lan�ant la commande suivante depuis le r�pertoire racine du projet. L'option ``-c`` pr�cise que PHPUnit doit charger sa configuration depuis le r�pertoire ``app``.

.. code-block:: bash

    $ phpunit -c app

Une fois que le test est termin�, vous devriez �tre averti que les tests ont �chou�. Si vous regardez dans la classe ``DefaultControllerTest`` dans ``src/Blogger/BlogBundle/Tests/Controller/DefaultControllerTest.php``, vous y trouverez le contenu suivant :

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Controller/DefaultControllerTest.php

    namespace Blogger\BlogBundle\Tests\Controller;

    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

    class DefaultControllerTest extends WebTestCase
    {
        public function testIndex()
        {
            $client = static::createClient();

            $crawler = $client->request('GET', '/hello/Fabien');

            $this->assertTrue($crawler->filter('html:contains("Hello Fabien")')->count() > 0);
        }
    }

C'est un test fonctionnel pour la classe ``DefaultController`` que Symfony2 a g�n�r�. Si vous vous souvenez du chapitre 1, ce controlleur avait une action qui g�rait les requ�tes vers ``/hello/{name}``. Puisque nous avons enlev� cette classe, le test �choue. Essayez de vous rendre � l'adresse ``http://symblog.dev/app_dev.php/hello/Fabien`` avec un navigateur. Symfony2 devrait vous informer que la route n'a pas pu �tre trouv�e. Comme ce test fait appel � la m�me adresse, il obtient la m�me r�ponse, d'o� l'�chec du test. Les tests fonctionnels vont couvrir une grosse partie de ce chapitre et seront abord�s en d�tails par la suite.

COmme nous avons supprim� la classe ``DefaultController``, vous pouvez �galement supprimer cette classe de test. Supprimez la classe  ``DefaultControllerTest`` situ�e dans ``src/Blogger/BlogBundle/Tests/Controller/DefaultControllerTest.php``.

Tests unitaires
---------------

Comme expliqu� pr�c�demment, les tests unitaires ont pour mission de tester des unit�s de l'application, individuellement, en isolation. Lorsque vous �crivez des tests unitaires, il est conseill� de reproduire la structure du Bundle dans le r�pertoire Tests. Si par exemple vous voulez tester la classe de l'entit� ``Blog``, situ�e dans ``src/Blogger/BlogBundle/Entity/Blog.php``, le fichier de test devrait se trouver dans
``src/Blogger/BlogBundle/Tests/Entity/BlogTest.php``. Une structure de r�pertoire pourrait �tre la suivante :

.. code-block:: text

    src/Blogger/BlogBundle/
                    Entity/
                        Blog.php
                        Comment.php
                    Controller/
                        PageController.php
                    Twig/
                        Extensions/
                            BloggerBlogExtension.php
                    Tests/
                        Entity/
                            BlogTest.php
                            CommentTest.php
                        Controller/
                            PageControllerTest.php
                        Twig/
                            Extensions/
                                BloggerBlogExtensionTest.php

Vous pouvez remarquer que tous les fichiers de tests sont suffix�s par Test.

Testing the Blog Entity - Slugify method
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Nous allons commencer par tester la m�thode slugify de l'entit� ``Blog``. Ecrivons quelques tests afin de nous assurer qu'elle fonctionne correctement. Cr�ez un nouveau fichier dans ``src/Blogger/BlogBundle/Tests/Entity/BlogTest.php`` et ajoutez-y le contenu suivant :

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    namespace Blogger\BlogBundle\Tests\Entity;

    use Blogger\BlogBundle\Entity\Blog;

    class BlogTest extends \PHPUnit_Framework_TestCase
    {

    }

Nous avons cr�� une classe de test pour l'entit� ``Blog``. Remarquez que la localisation de se fichier est conforme � la structure de r�pertoire �voqu�e plus haut. La classe ``BlogTest`` extends la classe de base de PHPUnit ``PHPUnit_Framework_TestCase``. Tous les tests que vous �crirez pour PHPUnit h�riteront de cette classe. Vous vous souvenez peut �tre depuis un chapitre pr�c�dent que le ``\`` doit �tre plac� devant le nom de la classe ``PHPUnit_Framework_TestCase`` car elle est d�clar�e dans l'espace de nom public de PHP.

Nous avons un squelette de la classe de test pour l'entit� ``Blog``, �crivons donc maintenant un sc�nario de test. Les sc�narios de test sont, avec PHPUnit, des m�thodes de la classe de test pr�fix�es par ``test``, tel que ``testSlugify()``. Mettez � jour la classe ``BlogTest`` dans
``src/Blogger/BlogBundle/Tests/Entity/BlogTest.php`` avec ce qui suit.

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    // ..

    class BlogTest extends \PHPUnit_Framework_TestCase
    {
        public function testSlugify()
        {
            $blog = new Blog();

            $this->assertEquals('hello-world', $blog->slugify('Hello World'));
        }
    }

C'est un sc�nario tr�s simple. On instancie une nouvelle entit� ``Blog`` et lance un ``assertEquals()`` sur le r�sultat de la m�thode ``slugify``. La m�thode ``assertEquals()`` prend au moins 2 arguments en param�tres, le r�sultat attendu et celui obtenu. Un 3eme argument permet de choisir le message � afficher en cas d'�chec.

Lan�ons notre nouveau test unitaire. Lancez la commande suivante.

.. code-block:: bash

    $ phpunit -c app

Vous devriez voir la sortie suivante :

.. code-block :: bash

    PHPUnit 3.5.11 by Sebastian Bergmann.

    .

    Time: 1 second, Memory: 4.25Mb

    OK (1 test, 1 assertion)

La sortie de l'ex�cution de PHPUnit est tr�s simple � comprendre. Le programme commence par afficher certaines informations � propos de PHPUnit, et affiche un ``.`` pour chaque test lanc�. Dans notre cas il y a seulement 1 test, donc seulement 1 ``.`` est affich�. La derni�re ligne nous informe du r�sultat du tests. Pour notre ``BlogTest`` nous avons seulement lanc� 1 test avec 1 assertion. Si vous avez l'affichage en couleur sur votre ligne de commande, vous verrez �galement la derni�re ligne en vert, ce qui montre que tout s'est bien ex�cut�.
Modifions la m�thode ``testSlugify()`` pour voir ce qui se passe lorsque le test �choue.

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    // ..

    public function testSlugify()
    {
        $blog = new Blog();

        $this->assertEquals('hello-world', $blog->slugify('Hello World'));
        $this->assertEquals('a day with symfony2', $blog->slugify('A Day With Symfony2'));
    }

Relancez le test unitaire comme pr�c�dement. Vous aurez alors la sortie suivante :

.. code-block :: bash

    PHPUnit 3.5.11 by Sebastian Bergmann.

    F

    Time: 0 seconds, Memory: 4.25Mb

    There was 1 failure:

    1) Blogger\BlogBundle\Tests\Entity\BlogTest::testSlugify
    Failed asserting that two strings are equal.
    --- Expected
    +++ Actual
    @@ @@
    -a day with symfony2
    +a-day-with-symfony2

    /var/www/html/symblog/symblog/src/Blogger/BlogBundle/Tests/Entity/BlogTest.php:15

    FAILURES!
    Tests: 1, Assertions: 2, Failures: 1.

La sortie est un peu plus �volu�e cette fois ci. On peut voir que les ``.`` ont laiss� place � un ``F``, qui nous indique qu'un des tests a �chou�. Vous verrez �galement ``E`` lorsque le test contient des erreurs. PHPUnit nous informe ensuite des d�tails des �checs, donc dans le cas pr�sent, de l'�chec. On peut voir que la m�thode ``Blogger\BlogBundle\Tests\Entity\BlogTest::testSlugify`` a �chou� car les valeurs attendues et obtenues sont diff�rentes. Si vous avez l'affichage en couleur dans la console, vous verrez la derni�re ligne en rouge, indiquant des �checs dans les tests. Corrigez la m�thode ``testSlugify()`` afin de r�ussir le test.

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    // ..

    public function testSlugify()
    {
        $blog = new Blog();

        $this->assertEquals('hello-world', $blog->slugify('Hello World'));
        $this->assertEquals('a-day-with-symfony2', $blog->slugify('A Day With Symfony2'));
    }

Avant d'avancer, ajoutons quelques autres tests pour la m�thode ``slugify()``.

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    // ..

    public function testSlugify()
    {
        $blog = new Blog();

        $this->assertEquals('hello-world', $blog->slugify('Hello World'));
        $this->assertEquals('a-day-with-symfony2', $blog->slugify('A Day With Symfony2'));
        $this->assertEquals('hello-world', $blog->slugify('Hello    world'));
        $this->assertEquals('symblog', $blog->slugify('symblog '));
        $this->assertEquals('symblog', $blog->slugify(' symblog'));
    }

Maintenant que nous avons test� la m�thode slugify de l'entit�  ``Blog``, il faut nous assurer que le membre 
``$slug`` est correctement affect� lorsque le membre ``$title`` de ``Blog`` est mis � jour. Ajoutez la m�thode suivante dans la classe ``BlogTest`` du fichier ``src/Blogger/BlogBundle/Tests/Entity/BlogTest.php``.

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Entity/BlogTest.php

    // ..

    public function testSetSlug()
    {
        $blog = new Blog();

        $blog->setSlug('Symfony2 Blog');
        $this->assertEquals('symfony2-blog', $blog->getSlug());
    }

    public function testSetTitle()
    {
        $blog = new Blog();

        $blog->setTitle('Hello World');
        $this->assertEquals('hello-world', $blog->getSlug());
    }

On commence par tester la m�thode ``setSlug`` pour s'assurer que le membre ``$slug`` est correctement slugifi� lorsqu'il est mis � jour. Ensuite on v�rifie que le membre ``$slug`` est correctement mis � jour lorsque la m�thode ``setTitle`` de l'entit� ``Blog`` est appel�e.

Lancez ces tests pour v�rifier que l'entit� ``Blog`` fonctionne correctement.

Test de l'extension Twig
~~~~~~~~~~~~~~~~~~~~~~~~

Dans le chapitre pr�c�dent, nous avons cr�� une extension Twig pour convertir une instance d'un objet ``\DateTime`` en une chaine de caract�res contenant la dur�e �coul�e depuis. Nous allons tester que cette m�thode se comporte bien comme nous l'attendons.
Cr�ez un nouveau fichier de test dans ``src/Blogger/BlogBundle/Tests/Twig/Extensions/BloggerBlogExtensionTest.php`` et mettez-y le ccontenu suivant :

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Twig/Extensions/BloggerBlogExtensionTest.php

    namespace Blogger\BlogBundle\Tests\Twig\Extensions;

    use Blogger\BlogBundle\Twig\Extensions\BloggerBlogExtension;

    class BloggerBlogExtensionTest extends \PHPUnit_Framework_TestCase
    {
        public function testCreatedAgo()
        {
            $blog = new BloggerBlogExtension();

            $this->assertEquals("0 seconds ago", $blog->createdAgo(new \DateTime()));
            $this->assertEquals("34 seconds ago", $blog->createdAgo($this->getDateTime(-34)));
            $this->assertEquals("1 minute ago", $blog->createdAgo($this->getDateTime(-60)));
            $this->assertEquals("2 minutes ago", $blog->createdAgo($this->getDateTime(-120)));
            $this->assertEquals("1 hour ago", $blog->createdAgo($this->getDateTime(-3600)));
            $this->assertEquals("1 hour ago", $blog->createdAgo($this->getDateTime(-3601)));
            $this->assertEquals("2 hours ago", $blog->createdAgo($this->getDateTime(-7200)));

            // Cannot create time in the future
            $this->setExpectedException('\Exception');
            $blog->createdAgo($this->getDateTime(60));
        }

        protected function getDateTime($delta)
        {
            return new \DateTime(date("Y-m-d H:i:s", time()+$delta));
        }
    }

La classe est construite de la m�me mani�re que pr�c�demment, et une m�thode ``testCreatedAgo()`` teste l'extension Twig. Nous utilisons une nouvelle m�thode de PHPUnit ici, la m�thode ``setExpectedException()``. Cette m�thode doit �tre appel�e avant une m�thode dont on attend qu'elle lance une exception. Nous savons que la m�thode ``createdAgo`` ne peut g�rer des dur�es dans le futur, et que si cela arrive, elle va lancer une ``\Exception``. La m�thode ``getDateTime()`` est simplement une petite fonction auxilliaire pour cr�er une instance de ``\DateTime`` facilement. Vous pouvez remarquer que cette m�thode n'est pas pr�fix�e par ``test``, donc PHPUnit n'essayera pas de l'ex�cuter. Ouvrez une ligne de commande et lancez les tests pour ce fichier. On pourrait simplement lancer le test comme avant, mais nous pouvons �galement pr�ciser � PHPUnit de lancer les tests sur tout un r�pertoire (et ses sous-r�pertoires) ou bien sur un seul fichier. Lancez la commande suivante :

.. code-block:: bash

    $ phpunit -c app src/Blogger/BlogBundle/Tests/Twig/Extensions/BloggerBlogExtensionTest.php

Cela va lancer les tests uniquement pour les fichiers de tests de notre extension. PHPUnit va alors nous informer que les tests ont �chou�. Regardons la sortie, et essayons de comprendre pourquoi :

.. code-block:: bash

    1) Blogger\BlogBundle\Tests\Twig\Extension\BloggerBlogExtensionTest::testCreatedAgo
    Failed asserting that two strings are equal.
    --- Expected
    +++ Actual
    @@ @@
    -0 seconds ago
    +0 second ago

    /var/www/html/symblog/symblog/src/Blogger/BlogBundle/Tests/Twig/Extensions/BloggerBlogExtensionTest.php:14

Nous voulions que la premi�re assertion renvoie ``0 seconds ago`` mais ce n'est pas arriv�, le mot ``second`` n'�tait pas au pluriel. Mettons � jour l'extension Twig dans ``src/Blogger/BlogBundle/Twig/Extensions/BloggerBlogBundle.php`` pour corriger cela.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Twig/Extensions/BloggerBlogBundle.php

    namespace Blogger\BlogBundle\Twig\Extensions;

    class BloggerBlogExtension extends \Twig_Extension
    {
        // ..

        public function createdAgo(\DateTime $dateTime)
        {
            // ..
            if ($delta < 60)
            {
                // Seconds
                $time = $delta;
                $duration = $time . " second" . (($time === 0 || $time > 1) ? "s" : "") . " ago";
            }
            // ..
        }

        // ..
    }

Relancez les tests. La premi�re assertion passe d�sormais, mais le jeu de test �choue plus loin quand m�me. Regardons � nouveau pourquoi :

.. code-block:: bash

    1) Blogger\BlogBundle\Tests\Twig\Extension\BloggerBlogExtensionTest::testCreatedAgo
    Failed asserting that two strings are equal.
    --- Expected
    +++ Actual
    @@ @@
    -1 hour ago
    +60 minutes ago

    /var/www/html/symblog/symblog/src/Blogger/BlogBundle/Tests/Twig/Extensions/BloggerBlogExtensionTest.php:18

Nous pouvons maintenant remarquer que c'est la 5�me assertion qui �choue. Prenez note du 18 � la fin de la sortie, qui nous indique la ligne dans le fichie o� l'assertion a �chou�e. En regardant le jeu de tests, on peut voir que l'extension ne se comporte pas comme attendu : l� o� il aurait fallu donner ``1 hour ago`` (il y a 1 heure), il a �t� r�pondu ``60 minutes ago`` (Il y a 60 minutes). Nous pouvons en trouver la raison en examinant le code du ``BloggerBlogExtension``. On compare les dur�es non strictement, c'est � dire avec ``<=`` plut�t ``<``. Nous le faisons �galement lorsque nous v�rifions les heures. Mettez � jour le code de ``src/Blogger/BlogBundle/Twig/Extensions/BloggerBlogBundle.php`` pour corriger cel�.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Twig/Extensions/BloggerBlogBundle.php

    namespace Blogger\BlogBundle\Twig\Extensions;

    class BloggerBlogExtension extends \Twig_Extension
    {
        // ..

        public function createdAgo(\DateTime $dateTime)
        {
            // ..

            else if ($delta < 3600)
            {
                // Mins
                $time = floor($delta / 60);
                $duration = $time . " minute" . (($time > 1) ? "s" : "") . " ago";
            }
            else if ($delta < 86400)
            {
                // Hours
                $time = floor($delta / 3600);
                $duration = $time . " hour" . (($time > 1) ? "s" : "") . " ago";
            }

            // ..
        }

        // ..
    }

Relancez � nouveau tous nos tests avec la commande suivante :

.. code-block:: bash

    $ phpunit -c app

Cela lance tous nos tests, et montre qu'ils passent tous sans erreurs. Bien que nous n'avons �crit que quelques tests unitaires, vous devriez d�j� sentir � quel point il est important et puissant de tester unitairement son code. Bien que les tests ci-dessus soient mineurs, il s'agit tout de m�me d'erreurs. Tester permet �galement de s'assurer que les nouvelles fonctionnalit�s ajout�es au projet ne cassent pas ce qui est d�j� en place. Cela conclut la page sur les tests unitaires pour cette fois-ci, mais nous y reviendrons par la suite. En attendant, vous pouvez essayer d'ajouter vos propres tests unitaires pour tester ce qui ne l'a pas encore �t�.

Tests fonctionnels
------------------

Maintenant que nous avons �crit quelques tests unitaires, passons aux tests de plusieurs composants � la fois. La premi�re section des tests fonctionnels va nous faire simuler des requ�tes dans un navigateur, afin d'analyser la r�ponse qui est g�n�r�e.

Test de la page A propos
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Nous commen�ons par tester la classe ``PageController`` pour la page ``A propos``. C'est un bon point de d�part, car cette page est tr�s simple. Cr�ez un nouveau fichier dans ``src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php`` et ajoutez-y le contenu suivant.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    namespace Blogger\BlogBundle\Tests\Controller;

    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

    class PageControllerTest extends WebTestCase
    {
        public function testAbout()
        {
            $client = static::createClient();

            $crawler = $client->request('GET', '/about');

            $this->assertEquals(1, $crawler->filter('h1:contains("About symblog")')->count());
        }
    }

Nous avons d�j� vu un test de controlleur tr�s similaire lorsque nous avons bri�vement jet� un oeil � la classe ``DefaultControllerTest``. Pour tester la page ``� propos``, on v�rifie que la chaine ``About symblog`` est pr�sente dans le HTML qui est g�n�r�, plus pr�cis�mment � l'int�rieur d'un tag ``H1``. La classe ``PageControllerTest`` n'�tend pas le ``\PHPUnit_Framework_TestCase`` comme nous l'avons vu dans les exemples des tests unitaires, elle �tend � la place la classe ``WebTestCase``. Cette classe fait partie du FrameworkBundle, un bundle de Symfony2.

Comme expliqu� plus t�t, les classes de tests de PHPUnit doivent �tendre ``\PHPUnit_Framework_TestCase``, mais lorsque des fonctionnalit�s communes ou suppl�mentaires sont n�cessaires dans plusieurs jeux de tests, il est utile de les encapsuler dans une classe � part et de faire que les classes de test �tendent alors cette classe. C'est ce que fait ``WebTestCase``, qui fournit plusieurs m�thode utiles aux tests fonctionnels dans Symfony2. Ouvrez la d�finition de ``WebTestCase`` dans
``vendor/symfony/src/Symfony/Bundle/FrameworkBundle/Test/WebTestCase.php``, vous y verrez que cette classe �tend en fait la classe ``\PHPUnit_Framework_TestCase``.

.. code-block:: php

    // vendor/symfony/src/Symfony/Bundle/FrameworkBundle/Test/WebTestCase.php

    abstract class WebTestCase extends \PHPUnit_Framework_TestCase
    {
        // ..
    }

Si vous regardez la m�thode ``createClient()`` de la classe ``WebTestCase``, vous verrez qu'elle instancie un kernel (noyau) de Symfony2. EN suivant les m�thodes, vous pouvez �galement voir que l' ``environment`` est d�fini � ``test``, � moins qu'il soit surd�fini comme argument � ``createClient()``. Il s'agit l� de l'environnement de test dont nous parlions dans le chapitre pr�c�dent.
En revenant � notre classe de test, on peut voir que la m�thode ``createClient()`` est appel�e pour mettre en marche le test. On appelle en suite ``request()`` sur le client pour simuler une requ�te  HTTP de type GET de la part d'un navigateur vers la page ``/about``, exactement comme si nous nous rendions sur ``http://symblog.dev/about`` dans un navigateur). La requ�te nous fournit en retour un objet ``Crawler`` qui contient la  ``Response``. La classe  ``Crawler`` est tr�s utile car elle nous laisse traverser le HTML qui nous est renvoy�. Nous utilisons le l'instance du  ``Crawler`` pour v�rifier que le tag ``H1`` de la r�ponse HTML contient les mots ``About symblog``. Vous remarquerez que, bien que nous �tendons d�sormais la classe ``WebTestCase``, nous continuons � utiliser les m�thodes d'assertions comme avant : souvenez vous que la classe ``PageControllerTest`` h�rite de ``\PHPUnit_Framework_TestCase`` .

Lan�ons le ``PageControllerTest`` avec la commande suivante. Lorsque l'on �crit des tests, il est utile de ne lancer que les tests sur le fichier sur lequel on est en train de travailler : lorsque les tests deviennent nombreux, tous les ex�cuter peut alors prendre beaucoup de temps.

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

Vous devriez recevoir le message ``OK (1 test, 1 assertion)``, qui nous informe qu'un test (``testAboutIndex()``) a tourn� avec une assertion (``assertEquals()``) et que le r�sultat est celui attendu. Tout va pour le mieux !

Essayez de changer la chaine ``About symblog`` pour ``Contact``, et lancez � nouveau le test. Vous verrez alors le test �chouter car ``Contact`` n'est pas trouv�, et le ``assertEquals`` donne alors une erreur.

.. code-block:: bash

    1) Blogger\BlogBundle\Tests\Controller\PageControllerTest::testAboutIndex
    Failed asserting that <boolean:false> is true.

Remettez le test � ``About symblog`` avant d'avancer.

L'instance du  ``Crawler`` que nous avons utilis� nous permet de traverser les documents HTML ou XML, ce qui signifie qu'il ne va marcher que pour des r�ponses de ce type. Nous pouvons utiliser le ``Crawler`` pour traverser les r�ponses g�n�r�es � l'aide de m�thodes telles que  ``filter()``, ``first()``, ``last()``, et ``parents()``. Si vous avez d�j� utilis� `jQuery <http://jquery.com/>`_ auparavant, vous ne serez pas perdu avec la classe ``Crawler``. Une liste compl�te des m�thodes de travers�e support�es par le ``Crawler`` se trouve dans le chapitre sur les `tests
<http://symfony.com/doc/current/book/testing.html#traversing>`_ du livre Symfony2. Nous allons en �voquer quelques uns en avan�ant.

Page d'accueil
~~~~~~~~~~~~~~

Bien que le test de la page ``A propos`` ait �t� tr�s simple, il nous a permis de mettre en avant les principes de base des tests fonctionnels des pages du site :

 1. Cr�er le client
 2. Effectuer une requ�te sur une page
 3. V�rifier la r�ponse

C'est une pr�sentation simple du processus, car il y a en fait plusieurs autres �tapes qui peuvent s'ajouter, comme cliquer sur les liens, ou remplir et soumettre des formulaires.

Cr�ons une m�thode pour tester la page d'accueil. Nous savons que la page d'accueil est disponible via l'URL ``/`` et qu'elle doit afficher les derniers articles. Ajoutez une nouvelle m�thode ``testIndex()`` � la classe ``PageControllerTest`` dans le fichier
``src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php``, avec le contenu suivant :

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    public function testIndex()
    {
        $client = static::createClient();

        $crawler = $client->request('GET', '/');

        // Check there are some blog entries on the page
        $this->assertTrue($crawler->filter('article.blog')->count() > 0);
    }

Vous pouvez voir que les m�mes �taptes ont lieu que pour les tests de la page ``A propos``. Lancez le test pour v�rifier que tout fonctionne comme pr�vu.

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

Allons maintenant un peu plus loin dans les tests. Une partie des tests fonctionnels consiste � r�pliquer ce qu'un utilisateur pourrait faire sur le site. Les utilisateurs cliquent sur les liens pour naviguer entre les pages. Simulons maintenant cette action pour tester que les liens vers la page d'affichage des articles fonctionnent am�nent bien vers la bonne page lorsque l'on clique sur les titres des articles.

Mettez � jour la m�thode ``testIndex()`` de la classe ``PageControllerTest`` avec le contenu suivant :

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    public function testIndex()
    {
        // ..

        // Find the first link, get the title, ensure this is loaded on the next page
        $blogLink   = $crawler->filter('article.blog h2 a')->first();
        $blogTitle  = $blogLink->text();
        $crawler    = $client->click($blogLink->link());

        // Check the h2 has the blog title in it
        $this->assertEquals(1, $crawler->filter('h2:contains("' . $blogTitle .'")')->count());
    }

La premi�re chose que nous faisons, c'est utiliser le ``Crawler`` pour extraitre le texte du premier lien vers un article, � l'aide du filtre ``article.blog h2 a``. Ce filtre est utilis� pour renvoyer un tag ``a`` dans un tag ``H2`` de la section ``article.blog``. Pour mieux comprendre cela, regardez le code HTML utilis� pour afficher les articles sur la page d'accueil.

.. code-block:: html

    <article class="blog">
        <div class="date"><time datetime="2011-09-05T21:06:19+01:00">Monday, September 5, 2011</time></div>
        <header>
            <h2><a href="/app_dev.php/1/a-day-with-symfony2">A day with Symfony2</a></h2>
        </header>

        <!-- .. -->
    </article>
    <article class="blog">
        <div class="date"><time datetime="2011-09-05T21:06:19+01:00">Monday, September 5, 2011</time></div>
        <header>
            <h2><a href="/app_dev.php/2/the-pool-on-the-roof-must-have-a-leak">The pool on the roof must have a leak</a></h2>
        </header>

        <!-- .. -->
    </article>

Vous pouvez voir le filtre ``article.blog h2 a`` en place dans le code de la page d'accueil. Vous pouvez �galement remarquer qu'il y a plus d'une section ``<article class="blog">`` dans le code, ce qui signifie que le filtre du ``Crawler`` va nous renvoyer une collection. Comme nous voulons seulement le premier lien, nous utilisons la m�thode ``first()`` sur la collection. Nous utilisons enfin la m�thode ``text()`` pour extraire le texte du lien, dans ce cas pr�cis il s'agit de ``A day with Symfony2``. On clique ensuite sur le lien du titre de l'article pour naviguer vers la page d'affichage des articles. La m�thode ``click()`` du client prend un objet lien en param�tre et renvoit la ``Response`` dans une instance de ``Crawler``. Vous devriez maintenant remarquer que l'objet ``Crawler`` est un �l�ment essentiel des tests fonctionnels.

L'objet ``Crawler`` contient maintenant la r�ponse vers la page d'affichage des articles. Nous devons tester que la page vers laquelle nous avons navigu� est bien la bonne. Nous pouvons utiliser la variable ``$blogTitle`` que nous avons r�cup�r� auparavant pour v�rifier le titre de la r�ponse.

Lancez les tests pour v�rifier que la navigation entre la page d'accueil et la page d'affichage des articles fonctionne correctement.

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

Vous savez maintenant comment naviguer entre les pages du site � travers les tests fonctionnels. Apprenons maintenant � tester les formulaires.

Test de la page de contact
~~~~~~~~~~~~~~~~~~~~~~~~~~

Les utilisateurs de Symblog peuvent envoyer des demandes de contact en utilisant le formulaire sur la page de contacts ``http://symblog.dev/contact``. Testons que la soumission de ce formulaire marche comme il faut. Nous devons tout d'abord savoir ce qui doit arriver si le formulaire est correctement soumis, ce qui, dans le cas pr�sent, doit arriver lorsqu'il n'y a pas d'erreurs dans le formulaire. Le processus est le suivant :

 1. Se rendre sur la page de contact
 2. Remplir les �l�ments du formulaire avec des valeurs
 3. Soumettre le formulaire
 4. V�rifier que l'email a �t� envoy� � symblog
 5. V�rifier que la r�ponse au client contient une notification de r�ussite d'envoi
 
Pour le moment, nous en savons assez pour faire seulement les �tapes 1 et 5. Nous allons maintenant regarder comment faire les 3 �tapes interm�diaires.

Ajoutez une nouvelle m�thode ``testContact()`` � la classe ``PageControllerTest`` dans
``src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php``.

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    public function testContact()
    {
        $client = static::createClient();

        $crawler = $client->request('GET', '/contact');

        $this->assertEquals(1, $crawler->filter('h1:contains("Contact symblog")')->count());

        // Select based on button value, or id or name for buttons
        $form = $crawler->selectButton('Submit')->form();

        $form['blogger_blogbundle_enquirytype[name]']       = 'name';
        $form['blogger_blogbundle_enquirytype[email]']      = 'email@email.com';
        $form['blogger_blogbundle_enquirytype[subject]']    = 'Subject';
        $form['blogger_blogbundle_enquirytype[body]']       = 'The comment body must be at least 50 characters long as there is a validation constrain on the Enquiry entity';

        $crawler = $client->submit($form);

        $this->assertEquals(1, $crawler->filter('.blogger-notice:contains("Your contact enquiry was successfully sent. Thank you!")')->count());
    }

On commence de la mani�re habituelle, en effectuant une requ�te vers l'URL ``/contact``, et en v�rifiant que la page contient le bon titre ``H1``. On utilise ensuite le ``Crawler`` pour choisir le bouton de soumission du formulaire. A partir du bouton, il est ensuite possible de retrouver le formulaire; on remplit ensuite les �l�ments du formulaire en utilisant la notation de tableau ``[]``. Le formulaire est ensuite fourni � la m�thode du client ``submit()`` pour soumettre le formulaire. Comme d'hbaitude, on re�oit en retour une instance de ``Crawler``. On utilise cette r�ponse pour v�rifier que le message flash est pr�sent dans la r�ponse qui nous est renvoy�e. Lancez le test pour v�rifier que tout fonctionne correctement.

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

Le test �choue, PHPUnit nous donne la sortie suivante :

.. code-block:: bash

    1) Blogger\BlogBundle\Tests\Controller\PageControllerTest::testContact
    Failed asserting that <integer:0> matches expected <integer:1>.

    /var/www/html/symblog/symblog/src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php:53

    FAILURES!
    Tests: 3, Assertions: 5, Failures: 1.

La sortie nous informe que le message flash n'a pas pu �tre trouv� dans la r�ponse obtenue apr�s soumission du formulaire. C'est parce que dans l'environnement de test, les redirections ne sont pas suivies. Lorsque le formulaire est effectivement valid� dans la classe ``PageController``, une redirection a lieu, mais n'est pas suivie. Nous devons dire explicitement que la redirection doit �tre suivie. La raison en est simple : il est possible que l'on veuille v�rifier la r�ponse actuelle d'abord. C'est ce que l'on verra bient�t pour v�rifier que l'envoi de l'email a bien eu lieu. Mettez � jour la classe ``PageControllerTest`` pour forcer le client � suivre la redirection.

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    public function testContact()
    {
        // ..

        $crawler = $client->submit($form);

        // Need to follow redirect
        $crawler = $client->followRedirect();

        $this->assertEquals(1, $crawler->filter('.blogger-notice:contains("Your contact enquiry was successfully sent. Thank you!")')->count());
    }

Les tests devraient maintenant passer dans PHPUnit. Regardons maintenant l'�tape finale du processus de v�rification de soumission des formulaires, qui consiste � s'assurer qu'un email a bien �t� envoy� � Symblog. On sait d�j� que les emails ne seront pas r�ellement envoy�s dans l'environnement de test, gr�ce � la configuration de cet environnement :


.. code-block:: yaml

    # app/config/config_test.yml

    swiftmailer:
        disable_delivery: true

Il est n�anmoins possible de v�rifier que les emails sont envoy�s en utilisant les informations rapport�es par la barre d'outils de debug. C'est l� o� l'importance pour le client de ne pas suivre les redirections rentre en jeu. La v�rification de l'outil de debug doit �tre r�alis�e avant que la redirection arrive, sinon les informations du profiler seraient perdues. Mettez � jour la m�thode de test ``testContact()`` avec ce qui suit :

.. code-block:: php

    // src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

    public function testContact()
    {
        // ..

        $crawler = $client->submit($form);

        // Check email has been sent
        if ($profile = $client->getProfile())
        {
            $swiftMailerProfiler = $profile->getCollector('swiftmailer');

            // Only 1 message should have been sent
            $this->assertEquals(1, $swiftMailerProfiler->getMessageCount());

            // Get the first message
            $messages = $swiftMailerProfiler->getMessages();
            $message  = array_shift($messages);

            $symblogEmail = $client->getContainer()->getParameter('blogger_blog.emails.contact_email');
            // Check message is being sent to correct address
            $this->assertArrayHasKey($symblogEmail, $message->getTo());
        }

        // Need to follow redirect
        $crawler = $client->followRedirect();

        $this->assertTrue($crawler->filter('.blogger-notice:contains("Your contact enquiry was successfully sent. Thank you!")')->count() > 0);
    }

Apr�s la soumission du formulaire, on v�rifie que le profiler est disponible, car il aurait pu �tre d�sactiv� par un param�tre de l'environnement.

.. tip::

    Souvenez vous que les tests n'ont pas besoin d'�tre r�alis�s dans l'environnement de test. Ils pourraient tr�s bien �tre lanc�s dans l'environnement de production, o� des �l�ments tel que le profiler ne sont pas disponibles.

Si nous arrivons � r�cup�rer le profiler, on effectue une requ�te pour obtenir l'espion de ``swiftmailer``. L'espion de ``swiftmailer`` se charge de surveiller, en arri�re plan, comment le service d'emails est utilis�. On peut l'utiliser pour obtenir des informations sur quels emails ont �t� envoy�s.

On utilise ensuite la m�thode ``getMessageCount()`` pour v�rifier qu'un email a �t� envoy�. C'est peut-�tre suffisant pour s'assurer qu'un email a bien �t� envoy�, mais cela ne v�rifie pas vers o� il est envoy�. Il peut �tre tr�s embarassant, voire dangereux, que des emails soient envoy�s � des adresses incorrectes. Afin de nous en assurer, on v�rifie que l'adresse du destinataire est bien correcte.

Lancez maintenant le test pour v�rifier que tout fonctionne correctement :

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/PageControllerTest.php

Test de l'ajout de commentaire d'articles
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Utilisons maintenant les connaissances que nous avons acquises dans les pr�c�dents tests de la page de contact pour tester le processus de soumission des commentaires pour un article. Voici ce qui doit arriver lorsqu'un commentaire est correctement soumis :

 1. Se rendre sur la page d'un article
 2. Remplir le formulaire de commentaire
 3. Soumettre le formulaire
 4. V�rifier que le nouveau commentaire est ajout� � la fin de la liste des commentaires
 5. V�rifier �galement dans la barre lat�rale que le commentaire ajout� est au sommet de la liste des derniers ccommentaires

Cr�ez un nouveau fichier dans ``src/Blogger/BlogBundle/Tests/Controller/BlogControllerTest.php`` et ajoutez-y le code suivant :

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Controller/BlogControllerTest.php

    namespace Blogger\BlogBundle\Tests\Controller;

    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

    class BlogControllerTest extends WebTestCase
    {
        public function testAddBlogComment()
        {
            $client = static::createClient();

            $crawler = $client->request('GET', '/1/a-day-with-symfony');

            $this->assertEquals(1, $crawler->filter('h2:contains("A day with Symfony2")')->count());

            // Select based on button value, or id or name for buttons
            $form = $crawler->selectButton('Submit')->form();

            $crawler = $client->submit($form, array(
                'blogger_blogbundle_commenttype[user]'          => 'name',
                'blogger_blogbundle_commenttype[comment]'       => 'comment',
            ));

            // Il faut suivre la redirection
            $crawler = $client->followRedirect();

            // On v�rifie que le comment s'affiche, et que c'est le dernier. Cela assure que les commentaires
            // vont du plus vieux au plus r�cent.
            $articleCrawler = $crawler->filter('section .previous-comments article')->last();

            $this->assertEquals('name', $articleCrawler->filter('header span.highlight')->text());
            $this->assertEquals('comment', $articleCrawler->filter('p')->last()->text());

            // On v�rifie que la barre lat�rale affiche bien 10 derniers articles.

            $this->assertEquals(10, $crawler->filter('aside.sidebar section')->last()
                                            ->filter('article')->count()
            );

            $this->assertEquals('name', $crawler->filter('aside.sidebar section')->last()
                                                ->filter('article')->first()
                                                ->filter('header span.highlight')->text()
            );
        }
    }

Il s'agit cette fois-ci du code entier du test directement. Avant de dissecter ce code, lancez le test pour v�rifier que tout fonctionne :

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Controller/BlogControllerTest.php

    
PHPUnit devrait vous dire qu'un test a �t� correctement ex�cut�. En regardant le code de ``testAddBlogComment()``, on peut voir que les choses se passent de la mani�re habituelle : on cr�e un client, effectue des requ�tes sur les pages et on v�rifie qu'elles contiennent bien ce qu'il faut.
On continue alors par obtenir le formulaire d'ajout des commentaires et le soumet. Cette fois-ci, nous peuplons le formulaire d'une mani�re un peu diff�rente de la pr�c�dente. Nous utilisons le 2nd argument de la m�thode ``submit()`` pour fournir les valeurs du formulaire.

.. tip::

    Nous pourrions �galement utiliser l'interface objet pour remplir les champs du formulaire, comme dans les exemples suivantes :

    .. code-block:: php

        // Cocher une case � cocher
        $form['show_emal']->tick();
        
        // Choisir une option ou un �l�ment radio
        $form['gender']->select('Male');

Apr�s avoir soumis le formulaire, on effectue la redirection du client afin de pouvoir v�rifier la r�ponse. On utilise � nouveau le ``Crawler`` afin d'obtenir le dernier commentaire de l'article, qui devrait �tre celui qui vient d'�tre soumis. On v�rifie enfin que le premier �l�ment de la liste des derniers commentaires de la barre lat�rale est bien le dernier commentaire propos�.

D�p�t d'articles
~~~~~~~~~~~~~~~~

Dans la derni�re partie sur les tests fonctionnels de ce chapitre, nous allons regarder comment tester un d�p�t Doctrine 2. Cr�ez un fichier dans
``src/Blogger/BlogBundle/Tests/Repository/BlogRepositoryTest.php`` et ajoutez-y le contenu suivante.

.. code-block:: php

    <?php
    // src/Blogger/BlogBundle/Tests/Repository/BlogRepositoryTest.php

    namespace Blogger\BlogBundle\Tests\Repository;

    use Blogger\BlogBundle\Repository\BlogRepository;
    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

    class BlogRepositoryTest extends WebTestCase
    {
        /**
         * @var \Blogger\BlogBundle\Repository\BlogRepository
         */
        private $blogRepository;

        public function setUp()
        {
            $kernel = static::createKernel();
            $kernel->boot();
            $this->blogRepository = $kernel->getContainer()
                                           ->get('doctrine.orm.entity_manager')
                                           ->getRepository('BloggerBlogBundle:Blog');
        }

        public function testGetTags()
        {
            $tags = $this->blogRepository->getTags();

            $this->assertTrue(count($tags) > 1);
            $this->assertContains('symblog', $tags);
        }

        public function testGetTagWeights()
        {
            $tagsWeight = $this->blogRepository->getTagWeights(
                array('php', 'code', 'code', 'symblog', 'blog')
            );

            $this->assertTrue(count($tagsWeight) > 1);

            // Test case where count is over max weight of 5
            $tagsWeight = $this->blogRepository->getTagWeights(
                array_fill(0, 10, 'php')
            );

            $this->assertTrue(count($tagsWeight) >= 1);

            // Test case with multiple counts over max weight of 5
            $tagsWeight = $this->blogRepository->getTagWeights(
                array_merge(array_fill(0, 10, 'php'), array_fill(0, 2, 'html'), array_fill(0, 6, 'js'))
            );

            $this->assertEquals(5, $tagsWeight['php']);
            $this->assertEquals(3, $tagsWeight['js']);
            $this->assertEquals(1, $tagsWeight['html']);

            // Test empty case
            $tagsWeight = $this->blogRepository->getTagWeights(array());

            $this->assertEmpty($tagsWeight);
        }
    }

Comme on veut effectuer des tests qui n�cessitent une connection effective vers la base de donn�e, il faut � nouveau �tendre la classe ``WebTestCase``, car cela nous permet de d�marrer le noyau Symfony2. Lancez le test sur ce fichier avec la commande suivante :

.. code-block:: bash

    $ phpunit -c app/ src/Blogger/BlogBundle/Tests/Repository/BlogRepositoryTest.php

Couverture de code
------------------

Avant d'avancer, quelques mots sur la couverture de code. Il s'agit d'un aper�u de quelles parties du code sont execut�es lorsque les tests sont lanc�s. Cela permet de savoir quelles sont les parties du code qui ne sont pas test�es, et de d�terminer s'il faut ou non �crire des test pour eux.

Afin d'obtenir l'analyse de couverture de code de votre application, lancez la commande suivante :

.. code-block:: bash

    $ phpunit --coverage-html ./phpunit-report -c app/

Cela donnera l'analyse de couverture de code dans le r�pertoire ``phpunit-report``. Lancez le fichier ``index.html`` dans votre navigateur pour obtenir les r�sultats de l'analyse.

Reportez vous au chapitre sur `l'analyse de la couverture de code <http://www.phpunit.de/manual/current/en/code-coverage-analysis.html>`_ de PHPUnit pour plus d'informations.

Conclusion
----------

Nous avons couvert un grand nombre d'aspects cl�s de la question des tests. Nous avons explor� � la fois les tests unitaires et fonctionnels pour nous assurer que notre site web fonctionne correctement. Nous avons vu comment manipuler les requ�tes les requ�tes du navigateur et comment utiliser la classe ``Crawler`` de Symfony2 pour v�rifier les r�ponses de ces requ�tes.

Dans le prochain chapitre, nous parlerons du composant de s�curit� de Symfony2, et plus sp�cifiquement de comment s'en servir pour la gestion des utilisateurs. Nous int�grerons �galement FOSUserBundle, qui va nous permettre de travailler directement sur la section d'administration de Symblog.











