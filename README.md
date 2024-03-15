# Découverte de Symfony

## Installation de Symfony

```shell
composer create-project symfony/skeleton:"7.0.*" my_project_directory
cd my_project_directory
composer require webapp
```
**`.env`** : qui va nous permettre de piloter les variables d'environnement de configurer le framework

**`assets`** : contient fichier CSS/JS ...

**`bin`** : console interagir avec le framework et phpunit qui est une sorte de pont pour interagir avec phpunit l'outil de test

```shell
# liste l'ensemble des opérations que l'ont peut faire avec symfony
php bin/console : symfony
```

**`config`** : configuration du framework et organiser en fonction des différents modules et utilise la syntaxe yaml basé sur l'indentation on a une clé: une valeur
migrations : qui contient les migrations ça a attrait à la base de données

**`public`** : qui servira de dossier racine, lorsqu'on va héberger symfony c'est celui-ci qui servira de racine à notre serveur web

**`src`** : contient l'ensemble de nos classes qui sont dans le namespace \App on trouve dans composer.json autoload on voit que le namespace App\\ est branché sur le src /

**`templates`** : va contenir nos vues HTML

**`tests`** : pour écrire nos tests

**`translations`** : va contenir nos traductions

**`var`** : va contenir des logs et des fichiers temporaires (vérifier qu'il est accessible en écriture)

**Les dossiers qui nous intéressent :**
- `public`
- `src`
- `templates`

## Lancement du serveur php en local

Pour lancer le serveur
```shell
php -S localhost:8000 -t public
```

## Notre première page

- Création d'un contrôleur, c'est une classe qui va contenir différente méthode chaque méthode correspondant en général à une URL particulière
```shell
php bin/console make:controller HomeController` # Pour générer le contrôleur de notre page d'accueil
```

```shell
 created: src/Controller/HomeController.php
 created: templates/home/index.html.twig
```

 Un contrôleur c'est une classe qui contient des méthodes et chaque méthode va correspondre à une page
 Les méthodes dans un contrôleur doivent toujours renvoyer une réponse  c'est un objet qui provient de symfony on retourne `use Symfony\Component\HttpFoundation\Response;`
 Il faudra typer Response
 Est on va retourner une nouvelle réponse c'est objet lorsqu'on le construit on lui passe en premier paramètre et en second paramètre statut puis les en-têtes

 ```php
 class HomeController
{
    function index(): Response
    {
        return new Response;
    }
}

```
Ce contrôleur HomeController il va falloir expliquer au framework que l'URL racine correspond à cette méthode là index(), ça va se faire au travers de la configuration.
Dans le dossier config on a un fichier routes.yaml
On va créer une nouvelle route

Lorsqu'on a l'URL racine `/` il faut que tu ailles chercher cette action execute la méthode index
routes.yaml
```yaml
home:
    path: /
    controller: App\Controller\HomeController::index
```

Autre méthode on rajoute un attribut au-dessus de notre méthode, elle est importée depuis le namespace use `Symfony\Component\Routing\Attribute\Route`;
En premier paramètre on spécifie le chemin, en second le nom
J'ai déclaré ma route au travers d'un attribut
`#[Route("chemin", nom)]`
```php
class HomeController
{
    #[Route("/", name: "home",)]
    function index(): Response
    {
        return new Response('Hello World !');
    }
}
```

`AbstractController` va fournir un ensemble de méthode utile pour les cas génériques le `extends AbstractController` (il va nous fournir des méthodes supplémentaires)

Par exemple on peut rediriger un utilisateur avec `return $this->redirect`
```php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;


class HomeController extends AbstractController
{
    #[Route("/", name: "home",)]
    function index(): Response
    {
        return new Response('Hello World !');
    }
}
```

Objet `Request` qui va nous permettre de gérer ce qui rentre dans le framework
Dans get on peut mettre un second paramètre nom par défaut
```php
// D'habitude on ferait en php 
return new Response('Hello' . $_GET['name']);

use Symfony\Component\HttpFoundation\Request;

class HomeController extends AbstractController
{
    #[Route("/", name: "home",)]
    function index(Request $request): Response
    {
        return new Response('Hello ' . $request->query->get('name', 'inconnu'));
    }
}
```

Dans c'est objet `Request`
- `dump()` vardump amélioré
- `dd()` vardump amélioré avec un die derrière

```php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class RecipeController extends AbstractController
{
    #[Route('/recette/{slug}-{id}', name: 'recipe.show')]
    public function index(Request $request): Response
    {
        dd($request->attributes->get('slug'), $request->attributes->get('id'));
    }
}
```
Si on veut récupérer ces attributs, on peut récupérer le slug et l'id
Beaucoup d'objets de la Request vont être de type `parameterBag` ça permet de représenter une collection de paramètres

http://localhost:8000/recette/pate-bolognaise-32
Cependant on souhaiterait que pate-bolognaise soit le slug et 32 l'id

On peut rajouter des options supplémentaires en utilisant des requirements: dans l'attribut Route
Les requirements vont nous permettre de spécifier le format attendu pour les paramètres de notre URL sous forme de tableau par exemple je souhaite que l'id soit simplement des nombres requirements: `['id' => '\d+'] \d+']` est une expression régulière et je voudrais que le slug soit des caractères alpha numériques en minuscule des nombres et un tiret.

Si on souhaite récupérer l'id sous la forme d'un entier `getInt('id')`
```php
class RecipeController extends AbstractController
{
    #[Route('/recette/{slug}-{id}', name: 'recipe.show', requirements: ['id' => '\d+'])]
    public function index(Request $request): Response
    {
        dd($request->attributes->get('slug'), $request->attributes->getInt('id'));
    }
}
```

On met les paramètres dans l'URL '`/recette/{slug}-{id}`' on spécifie les requirements (format attendu pour ces paramètres) et ensuite on peut utiliser la propriété attributes sur la requête pour récupérer les informations qui nous intéresse.

On peut lui spécifier les paramètres directement au niveau de notre méthode index je m'attends à avoir une chaine de caractère qui soit le slug et un entier qui soit l'id.
`dd($slug, $id);`
On va voir que ça va être automatiquement hydraté avec les bonnes valeurs
```php
class RecipeController extends AbstractController
{
    #[Route('/recette/{slug}-{id}', name: 'recipe.show', requirements: ['id' => '\d+'])]
    public function index(Request $request, string $slug, int $id): Response
    {
        dd($slug, $id);
    }
}
```

2 façons de retourner du json :
```php
    #[Route('/recette/{slug}-{id}', name: 'recipe.show', requirements: ['id' => '\d+', 'slug' => '[a-z0-9-]+'])]
    public function show(Request $request, string $slug, int $id): Response
    {
        // return new JsonResponse([
        //     'slug' => $slug
        // ]);
        
        return $this->json([
            'slug' => $slug
        ]);

        return new Response('Recette : ' . $slug);
    }
```

Une méthode qui renvoie une réponse
On a la possibilité de définir les routes directement de manière adjacente avec un attribut ou via le fichier `config\routes.yaml`

exemple
```yaml
controllers:
    resource:
        path: ../src/Controller/
        namespace: App\Controller
    type: attribute

home:
    path: /
    controller: App\Controller\HomeController::index
```

> [!TIP]
> Pour connaitre l'ensemble des routes définies : php bin/console debug:router
> Liste l'ensemble des routes de l'application

## Moteur de template Twig

Lorsqu'on a généré nos contrôleurs, des fichiers Twig ont été automatiquement générés dans le dossier templates

ça va nous permettre d'utiliser un tag
`extends` va étendre du fichier `base.html.twig`
```php
{% extends 'base.html.twig' %}
```

**Documentation de Twig : **
- https://twig.symfony.com/doc/

```php
{% extends 'base.html.twig' %}

{% block body %}
    Lorem ipsum dolor sit, amet consectetur adipisicing elit. Est alias cumque voluptas ad dolores possimus dignissimos excepturi, et quam assumenda dolor consectetur saepe atque beatae esse praesentium eaque mollitia maxime?
{% endblock %}

```

Si maintenant je veux rendre cette page `index.html.twig` en tapant l'URL http://localhost:8000/recette

`RecipeController.php`
```php
    #[Route('/recette', name: 'recipe.index')]
    public function index(Request $request): Response
    {
        return $this->render('recipe/index.html.twig');
    }
```

Pour changer le `<title>`
```php
{% block title %}
    Toutes les recettes
{% endblock %}
```

Méthode raccourci pour le même résultat
```php
{% block title "Toutes les recettes" %}   
```

`show.html.twig`
```php
{% extends 'base.html.twig' %}

{% block title "Recette : " ~ recipe.title %}


{% block body %}

<h1>{{ recipe.title }}</h1>
<p>{{ recipe.content | nl2br }}</p>

{% endblock %}

```
Au niveau de notre contrôleur lorsqu'on utilise la méthode render on peut lui passer en second paramètre un tableau contenant les variables que l'on souhaite envoyer à notre vue, variable que l'on pourra afficher à l'intérieur de notre fichier show.html.twig

**Source :** 
- https://symfony.com/doc/current/reference/configuration/twig.html

**Récupérer la route courante dans Twig**
```php
{{ dump(app.current_route()) }}
```

**Récupérer la request dans Twig**
```php
{{ dump(app.request()) }}
```

**Mettre `active` sur le lien courant**
```php
<li class="nav-item">
<a class="nav-link {{ app.current_route == 'home' ? 'active' : '' }}" href="{{ path('home') }}">Accueil</a>
</li>
```

> [!IMPORTANT]
> Dans le cadre de Twig c'est un double égal == pas le triple égal ===

### En résumé :

Dans templates :

On va mettre tout ce qui est commun dans `base.html.twig`

Dans nos vues on fait :
```php
{% extends 'base.html.twig' %}
```
on choisit le titre que l'on veut mettre
```php
{% block title %}Hello HomeController!{% endblock %}
```
on définit un block body
```php
{% block body %}
    Home
{% endblock %}
```

Dans notre contrôleur, `src/controller` 

On va souvent avoir à la fin de notre code une méthode render on lui passera en premier paramètre le nom du fichier Twig que l'on souhaite utiliser et en second paramètre un tableau contenant tout ce que l'on souhaite envoyer à notre vue.

```php
return $this->render('recipe/show.html.twig', [
    'slug' => $slug,
    'id' => $id,
    'htmldemo' => '<strong>mon code gras</strong>',
    'person' => [
        'firstname' => 'John',
        'lastname' => 'Doe'
    ]
]);
```

## L'ORM dans Doctrine

ORM (Object Relationnal Mapping) c'est un système qui va nous permettre d'interagir avec la bdd avec des objets.
On va avoir des objets qui vont représenter les données que nous aurons dans notre table.

Alternative à phpMyAdmin
Télécharger Adminer for MySQL :
- https://www.adminer.org/#download

Le placer dans public de notre application et le renommer adminer.php

http://localhost:8000/adminer.php

Pour connaitre la version de MySQL
```sql
SELECT VERSION();
```

Éditer le fichier `.env`
```shell
DATABASE_URL="mysql://root:root@127.0.0.1:3306/db_cook?serverVersion=8.2.0&charset=utf8mb4"
```

Création d'une table
On va créer une entité. Une entité c'est un objet php qui va permettre de représenter chaque élément dans notre table.

```shell
php bin/console make:entity

Class name of the entity to create or update (e.g. FierceKangaroo):
> Recipe
Add the ability to broadcast entity updates using Symfony UX Turbo? (yes/no) [no]:
> no
created: src/Entity/Recipe.php
created: src/Repository/RecipeRepository.php

Entity generated! Now let's add some fields!
 You can always add more fields later manually or by re-running this command.

New property name (press <return> to stop adding fields):
> title

Field type (enter ? to see all types) [string]:
> string
string

Field length [255]:
> 255

 Can this field be null in the database (nullable) (yes/no) [no]:
 > no

 updated: src/Entity/Recipe.php

 Add another property? Enter the property name (or press <return> to stop adding fields):
 >

 
  Success! 
 

 Next: When you're ready, create a migration with php bin/console make:migration
```

Pour ajouter d'autres champs directement
```shell
php bin/console make:entity Recipe
```

```shell
 Add another property? Enter the property name (or press <return> to stop adding fields):
 > createdAt

 Field type (enter ? to see all types) [datetime_immutable]:
 >

Add another property? Enter the property name (or press <return> to stop adding fields):
 > updatedAt

 Field type (enter ? to see all types) [datetime_immutable]:
 >
```

> [!TIP]
> La valeur entre crochets [no] est la valeur par défaut on peut simplement appuyer sur entrée

### Entity

Si on retourne sur notre application dans `src`
On a un nouveau dossier `Entity` qui va contenir notre recette
Recipe.php

Si on ouvre Recipe.php c'est un fichier php classique qui contient une class Recipe qui contient différentes propriétés, chaque propriété correspondant à ce que j'ai rentré dans la console

#### Getter & Setter
Il a également créé des **getters** et des **setters** : ce sont des méthodes qui permettent de récupérer ou de définir la valeur pour chacune de nos propriétés. Il fait ça automatiquement.

#### Attributs
Ce qu'on remarque il a aussi créé des **attributs**, ces attributs permettent à l'ORM de comprendre comment les données seront sauvegardées dans notre base de données

Par exemple pour le champ `title` :

On sait que le title sera une colonne dans notre base de données et aura une taille de 255 caractères
```php
#[ORM\Column(length: 255)]
    private ?string $title = null;
```

Par exemple pour le champ content :
ça va être un type Text
```php
   #[ORM\Column(type: Types::TEXT)]
    private ?string $content = null;
```

**Une entité c'est une classe classique et c'est grâce aux attributs que Doctrine va savoir comment il doit sauvegarder les informations en base de données**

### Repository

On a un nouveau dossier Repository qui contient
RecipeRepository.php

Ça, c'est une classe qui va nous permettre de récupérer des enregistrements dans cette classe on va créer les méthodes qui vont nous permettre d'extraire des données depuis notre base de données.

**Cette classe-là va contenir par défaut des méthodes qui vont nous permettre de récupérer un ou plusieurs enregistrements**.

À chaque fois on aura un **repository** qui va être **associé à une entité** c'est ce repository qui **sera chargé de récupérer les recettes**. Dès qu'on aura des méthodes spécifiques à la récupération de recettes, c'est dans ce repository qu'on le mettra.


### Création de nos tables dans la base de données
```shell
$ php bin/console make:migration
 created: migrations/Version20240313202110.php

 
  Success! 
 

 Review the new migration then run it with php bin/console doctrine:migrations:migrate
 See https://symfony.com/doc/current/bundles/DoctrineMigrationsBundle/index.html
```

Il créer un fichier dans le dossier migrations dans lequel on a une partie description ou l'on peut expliquer ce que fait cette migration

```php
    public function getDescription(): string
    {
        return '';
    }
```

Et automatiquement il a généré la requête SQL qui permet de créer notre table

```php
$this->addSql('CREATE TABLE recipe (id INT AUTO_INCREMENT NOT NULL, title VARCHAR(255) NOT NULL, slug VARCHAR(255) NOT NULL, content LONGTEXT NOT NULL, created_at DATETIME NOT NULL COMMENT \'(DC2Type:datetime_immutable)\', updated_at DATETIME NOT NULL COMMENT \'(DC2Type:datetime_immutable)\', PRIMARY KEY(id)) DEFAULT CHARACTER SET utf8mb4 COLLATE `utf8mb4_unicode_ci` ENGINE = InnoDB');
```

Ce qu'il fait il regarde l'état de notre table actuelle, il regarde nos entités et il génère automatiquement les requêtes SQL qui permettent de créer nos tables.

Pour chaque migration

On la partie up qui permet d'expliquer comment on avance
```php
public function up(Schema $schema): void
```
et on a la partie down qui permet d'expliquer comment faire le chemin inverse
```php
public function down(Schema $schema): void
```

On exécute notre migration
```shell
php bin/console doctrine:migrations:migrate
```

> [!TIP]
> En tapant cette ligne, il vous dira que la commande n'est pas définie et il vous proposera la bonne ligne de commande, c'est une astuce pour éviter de taper la commande en entier.
> `php bin/console migrate`

```shell
$ php bin/console doctrine:migrations:migrate

 WARNING! You are about to execute a migration in database "db_cook" that could result in schema changes and data loss. Are you sure you wish 
to continue? (yes/no) [yes]:
 >

[notice] Migrating up to DoctrineMigrations\Version20240313202110
[notice] finished in 137.1ms, used 24M memory, 1 migrations executed, 2 sql queries
```

La table: `doctrine_migration_versions`
Va retenir l'ensemble des migrations qui ont été exécutées


### Avantage
L'avantage de ce système de migration va nous permettre de gérer automatiquement l'évolution de notre base de données et va permettre de synchroniser la structure de notre base de données avec nos entités.


## ORM Doctrine

**Méthode Find :**

- `find($id)` : permettra de retrouver un enregistrement en particulier
- `findBy()` : permettra de spécifier un critère sous forme de tableau et de récupérer des enregistrements en fonction d'une condition sur un champ
- `findAll()` : permettra de récupérer tous les enregistrements 
- `findOneBy()` : permettra de récupérer un enregistrement en fonction d'un critère en particulier


RecipeController.php
```php
    #[Route('/recettes', name: 'recipe.index')]
    public function index(Request $request, RecipeRepository $repository): Response
    {
        $recipes = $repository->findAll();
        return $this->render('recipe/index.html.twig', [
            'recipes' => $recipes
        ]);
    }
```

Recipe.php

```php
{% block body %}
    <ul>
    {% for recipe in recipes %}
        <li>
            <a href="{{ path('recipe.show', {id: recipe.id, slug: recipe.slug}) }}">{{ recipe.title }}</a>
        </li>
    {% endfor %}
    </ul>

{% endblock %}
```

On pourrait se dire que title ne peut pas fonctionner, car c'est une propriété private.

Lorsqu'on utilise une propriété privée dans Twig, il recherche d'abord un getter associé à cette propriété. Si un getter existe, il l'utilise pour récupérer la valeur.

Donc quand on fait `recipe.title` c'est comme si l'on faisait `recipe.getTitle()`.

> [!TIP]
> Dans Twig au lieu d'écrire {{ recipe.getTitle() }} on peut l'écrire {{ recipe.title }}


```php
    #[Route('/recettes/{slug}-{id}', name: 'recipe.show', requirements: ['id' => '\d+', 'slug' => '[a-z0-9-]+'])]
    public function show(Request $request, string $slug, int $id, RecipeRepository $repository): Response
    {
        // Si l'on souhaite rechercher par l'id
        $recipe = $repository->find($id);
        // Si l'on souhaite rechercher par le slug
        $recipe = $repository->findOneBy(['slug' => $slug]);
```

La condition nous permet que si l'utilisateur se trompe dans le slug, il sera redirigé vers la bonne page qui correspond à l'id.

**Il existe 2 méthodes de redirection :**
- `redirect()` : qui permet de rediriger vers une URL
- `redirectToRoute()` : qui permet de rediriger vers une route

```php
    #[Route('/recettes/{slug}-{id}', name: 'recipe.show', requirements: ['id' => '\d+', 'slug' => '[a-z0-9-]+'])]
    public function show(Request $request, string $slug, int $id, RecipeRepository $repository): Response
    {
        $recipe = $repository->find($id);
        // Si le recipe est différent de celui qu'on a reçu dans l'URL dans ce cas on fait une redirection
        if ($recipe->getSlug() !== $slug) {
            return $this->redirectToRoute('recipe.show', ['slug' => $recipe->getSlug(), 'id' => $recipe->getId()]);
        }
        return $this->render('recipe/show.html.twig', [
            'recipe' => $recipe
        ]);
    }
```

show.html.twig
```php
{% extends 'base.html.twig' %}

{% block title "Recette : " ~ recipe.title %}


{% block body %}

<h1>{{ recipe.title }}</h1>
<p>{{ recipe.content | nl2br }}</p>

{% endblock %}
```

Le filtre `nl2br` sert à transférer les sauts de ligne.


## Requêtes personnalisées

### Récupération de données

Si l'on souhaite afficher toutes les recettes qui durent moins de 10min.

On va faire une requête personnalisée et ça va se gérer au niveau du repository dans RecipeRepository.php

On créer pour ça une nouvelle méthode
Une fonction publique `findWithDurationLowerThan()` qui prendra en paramètre un entier `$duration`, et le retour de cette fonction sera un tableau (on la type `array`).

Cette fonction fera :
Sélectionne toutes les informations depuis la recette et rajoute un WHERE duration est inférieur à quelque chose

On utilisera un queryBuilder dans le cadre de Doctrine ça permet de construire une requête SQL, l'avantage c'est lui qui va traduire ça en fonction de la bdd qu'on utilise.

`createQueryBuilder()`, on lui passe en premier paramètre un alias, c'est comme en SQL dans un SELECT quand on créer un alias pour appeler les champs plus facilement.
```php
createQueryBuilder('r')
```

On a différentes méthodes qui correspondent à ce qu'on fait en général en SQL: 
- `where()`
- `orderBy()`
- `leftJoin()`
- **limit** n'existe pas on utilisera **`setMaxResults()`**

```php
/**
 * 
 * @param int $duration 
 * @return array Recipe[]
 */
public function findWithDurationLowerThan(int $duration): array
{
    return $this->createQueryBuilder('r')
                ->where('r.duration <= :duration')
                ->orderBy('r.duration', 'ASC')
                ->setMaxResults(10)
                ->setParameter('duration', $duration)
                ->getQuery()
                ->getResult();
}
```

Quand `recipe.duration` est inférieur ou égal à `:duration` (on fait une requête préparée avec comme paramètre `:duration`)
```php
->where('r.duration <= :duration')
```

Trier par la durée de la plus rapide à la plus longue
```php
->orderBy('r.duration', 'ASC')
```

Je veux seulement 10 résultats
```php
->setMaxResults(10)
```

Correspond à un `bindParam(key,value)`
```php
->setParameter('duration', $duration)
```

On génère la requête, ça va nous donner un objet Query qui est un objet de Doctrine
```php
->getQuery()
```

Si on veut la requête SQL (dans notre cas ce n'est pas ce qui nous intéresse)
```php
->getSQL()
```

Si on veut le résultat
```php
->getResult();
```

Maintenant si sur la page d'accueil on souhaite récupérer uniquement les recettes les plus rapides

On récupèrera les recettes inférieures à 10 minutes
```php
findWithDurationLowerThan(10)
```

RecipeController.php
```php
    #[Route('/recettes', name: 'recipe.index')]
    public function index(Request $request, RecipeRepository $repository): Response
    {
        $recipes = $repository->findWithDurationLowerThan(10);
        return $this->render('recipe/index.html.twig', [
            'recipes' => $recipes
        ]);
    }

```

#### En résumé :

Dès qu'on a besoin de communiquer avec la bdd de manière plus profonde pour construire une requête particulière, on peut utiliser le Query Builder.

### Modifier un enregistrement

Pour faire les mises à jour on va avoir besoin d'une autre classe dans RecipeController.php qui est le `EntityManagerInterface` elle provient de `Doctrine\ORM`

```php
    #[Route('/recettes', name: 'recipe.index')]
    public function index(Request $request, RecipeRepository $repository, EntityManagerInterface $em): Response
```

Cette classe est responsable de la mémorisation de toutes les entités que l'on a dans notre application.
Lorsqu'un repository récupère des entités, automatiquement elles sont sauvegardées dans l'entity manager.

Dans notre cas, si on a nos 2 recettes l'entity manager à en mémoire ces 2 recettes.
Ce qu'il est possible de faire c'est de modifier une recette à la volée.

Exemple :
Je veux prendre ma recette numéro et je vais lui dire de faire un setTitle et l'appeler Pâtes bolognaises.
On pourra ainsi modifier l'ensemble de nos entités
```php
    #[Route('/recettes', name: 'recipe.index')]
    public function index(Request $request, RecipeRepository $repository, EntityManagerInterface $em): Response
    {
        $recipes = $repository->findWithDurationLowerThan(10);

        // Modification du titre de la recette
        $recipes[0]->setTitle('Pâtes bolognaise');

        // Enregistre toutes les modifications dans ma base de données
        $em->flush();

        return $this->render('recipe/index.html.twig', [
            'recipes' => $recipes
        ]);
    }

```
La méthode `flush()` va persister toutes les modifications faites de toutes les entités.

> [!NOTE]
> Il ne refera pas la requête systématiquement si la valeur n'a pas changé.

### Créer un nouvel objet (nouvelle recette)

On crée notre objet
```php
$recipe = new Recipe();
```

On va faire des setters pour remplir notre objet

Tous les setters sont des setters Fluent c'est-à-dire que lorsqu'ils ont fini de faire le setter ils retournent l'instance de l'objet, ça permet d'enchainer les différentes méthodes les unes sur les autres. L'avantage, on n’est pas obligé de redéfinir les choses.
```php

        $recipe->setTitle('Barbe à papa')
            ->setSlug('barbe-papa')
            ->setContent('Mettez du sucre')
            ->setDuration(2)
            ->setCreatedAt(new \DateTimeImmutable())
            ->setUpdatedAt(new \DateTimeImmutable());

```


`\DateTimeImmutable` est une classe fournie par php pour gérer les dates et les heures. Immutable : veut dire que la date est immuable. Une fois créée, elle ne peut pas être modifiée. Cela garantit que la date de création reste constante.

**Source :** 
- https://www.php.net/manual/fr/class.datetimeimmutable.php

Là on a créé notre objet, mais si on fait un `flush()` derrière cet objet-là ne sera pas automatiquement sauvegardé dans notre base de données, parce l'Entity Manager n'est pas au courant que cet objet existe, il n'y a pas de lien qui a été fait. Donc avant de faire un flush il faudra lui dire au niveau de l'Entity Manager je veux que tu persistes cet objet-là `persist($recipe)`.

```php
        $recipe = new Recipe();
        $recipe->setTitle('Barbe à papa')
            ->setSlug('barbe-papa')
            ->setContent('Mettez du sucre')
            ->setDuration(2)
            ->setCreatedAt(new \DateTimeImmutable())
            ->setUpdatedAt(new \DateTimeImmutable());
        $em->persist($recipe);
        $em->flush();
```

`persist($recipe)` : je souhaite que tu sauvegardes la présence de cet objet.

Si on ne fait qu'un persist ça ne fera rien. Ce n'est que lorsqu'on fait un flush que Doctrine fera la comparaison et générera les requêtes SQL et les exécuter pour que les modifications qu'on a effectuées jusqu'au flush soient persisté.

### Supprimer un objet (supprimer une recette)

```php
        $em->remove($recipes[2]);
        $em->flush();
```

### En résumé :
Dès que l'on voudra faire des modifications au niveau de la base de données, ce sera l'Entity Manager que l'on devra utiliser.

> [!TIP]
> L'Entity Manager peut permettre de récupérer un repository

Je lui donne le nom de la classe qui correspond à mon entité, il a va automatiquement retrouver le repository associé
```php
dd($em->getRepository(Recipe::class));
```

Entity\Recipe.php
```php
#[ORM\Entity(repositoryClass: RecipeRepository::class)]
class Recipe
{
```
Au niveau de notre Recipe on lui a dit c'est une entité Doctrine `Entity()` et voilà la classe qui sert de repository `repositoryClass: RecipeRepository::class`

ça nous permettra de pas trop injecter de choses au niveau de notre contrôleur
Avant :
```php
    #[Route('/recettes', name: 'recipe.index')]
    public function index(Request $request, RecipeRepository $repository, EntityManagerInterface $em): Response
    {
        dd($em->getRepository(Recipe::class));
        $recipes = $repository->findWithDurationLowerThan(20);
```
Après :
```php
    #[Route('/recettes', name: 'recipe.index')]
    public function index(Request $request, EntityManagerInterface $em): Response
    {
        dd($em->getRepository(Recipe::class));
        $recipes = $em->getRepository(Recipe::class)->findWithDurationLowerThan(20);
```

On souhaite récupérer le total des durées de nos recettes pour afficher sur la page d'accueil plus de 10h de recette de cuisine 

RecipeRepository.php
```php
    public function findTotalDuration()
    {
        return $this->createQueryBuilder('r')
        // Je veux sélectionner un champ particulier
        // On veut la somme des durations et on va l'appeler total
            ->select('SUM(r.duration) as total')
            ->getQuery()
            ->getResult();
    }
```
Vu qu'on utilise un select on ne va pas récupérer des champs particuliers, ce n'est pas quelque chose qu'on peut hydrater dans une recette, dans ce cas-là on va faire un `getQuery()` et quand on fait un `getResult()` on voit qu'on a différent systèmes de résultats.

- `getArrayResult()` : on peut récupérer ça sous la forme d'un tableau
- `getFirstResult()` : Nous renverra qu'une seule ligne
- `getScalarResult()` : Nous renverra une valeur scalaire
- `getSingleResult()` : Nous renverra qu'un seul résultat (il ne fonctionne que si on a qu'une seule ligne)
- `getSingleScalarResult()` : Donnera directement le premier champ

Ces fonctions-là sont intéressantes dès lors qu'on ne veut pas simplement récupérer une collection d'entités.

> [!NOTE]
> Une **valeur scalaire** est une valeur simple. Elle représente une seule donnée.

Pour retourner le premier champ en un entier
```php
    public function findTotalDuration(): int
    {
        return $this->createQueryBuilder('r')
        // Je veux sélectionner un champ particulier
        // On veut la somme des durations et on va l'appeler total
            ->select('SUM(r.duration) as total')
            ->getQuery()
            ->getSingleScalarResult();
    }
```

### Récapitulatif

Lorsqu'on a besoin de stocker des informations, on va créer une nouvelle entité

#### READ
Une fois qu'on a ses entités on va pouvoir les récupérer grâce au repository qui contient des méthodes de bases comme `find()` ... mais on peut aussi créer nos propres méthodes en fonction de nos besoins. Méthode dans lequel on pourra utiliser le QueryBuilder pour construire une requête SQL permettant de récupérer les données que l'on souhaite.

#### MODIFY
Si on souhaite faire des modifications, il suffit de modifier nos objets et d'utiliser l'Entity Manager et la méthode `flush()` pour persister toutes les modifications qui ont été faite.

#### CREATE
De la même manière si on souhaite crée un nouvel objet, on commence par crée une nouvelle entité en l'initialisant et ensuite on utilise la méthode `persist()` pour dire à l'Entity Manager de suivre cet objet-là, une fois qu'on l'a persisté on peut utiliser la méthode `flush()` pour mémoriser les informations en base de données.

#### DELETE
On peut faire un `remove()` et ensuite on `flush()` pour enregistrer la suppression.

## Les formulaires

### Création du formulaire avec le FormBuilder
On créer un formulaire qui va nous permettre de gérer les recettes que l'on nommera `RecipeType`. On mettra un suffixe `Type` pour le nom de nos formulaires. Et on le rattachera à l'entité `Recipe`.
```shell
php bin/console make:form

 The name of the form class (e.g. GentlePopsicleType):
 > RecipeType

 The name of Entity or fully qualified model class name that the new form will be bound to (empty for none):
 > Recipe
Recipe

 created: src/Form/RecipeType.php

 
  Success! 
 

 Next: Add fields to your form and start using it.
 Find the documentation at https://symfony.com/doc/current/forms.html
```

Il va générer automatiquement un fichier `RecipeType.php` qu'il va placer dans le dossier `src\Form`

Automatiquement le fichier a été prérempli en fonction des champs qui sont dans mon entité `Recipe`

```php
class RecipeType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('title')
            ->add('slug')
            ->add('content')
            ->add('createdAt', null, [
                'widget' => 'single_text',
            ])
            ->add('updatedAt', null, [
                'widget' => 'single_text',
            ])
            ->add('duration')
        ;
    }
```

Il a également défini une `data_class`, qui est une option qui permet d'expliquer qu'à l'intérieur de ce formulaire on va traiter un objet de type recette
```php
    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => Recipe::class,
        ]);
    }
}
```

### Création de la route dans RecipeController

On va créer une route qui sera
`localhost:8000/recettes/{id}/edit`

`RecipeController.php`
```php
    #[Route('/recettes/{id}/edit', name: 'recipe.edit', methods: ['GET', 'POST'])]
    public function edit(int $id)
    {
        dd($recipe);
    }

```
Dans cette action on va récupérer la recette qui correspond à un id

> [!TIP]
> Lorsqu'on a un {id} on peut directement mettre je veux une recette `Recipe $recipe`
> (Tu vas recevoir un paramètre qui sera une recette)
> Il va se charger de faire la requete `$recipe = $repository->find($id)` et ça économisera une ligne de code

`RecipeController.php`
```php
    #[Route('/recettes/{id}/edit', name: 'recipe.edit', methods: ['GET', 'POST'])]
    public function edit(Recipe $recipe)
    {
        dd($recipe);
    }
```

Envoyer ça à ma vue `recipe/edit.html.twig`

`RecipeController.php`
```php
    #[Route('/recettes/{id}/edit', name: 'recipe.edit', methods: ['GET', 'POST'])]
    public function edit(Recipe $recipe)
    {
        return $this->render('recipe/edit.html.twig');
    }
```

Au niveau de notre contrôleur on va pouvoir créer notre formulaire
On va utiliser une méthode de notre `AbstactController` qui est `createForm()`

La méthode `createForm()` prend en premier paramètre le formulaire que l'on va vouloir utiliser `RecipeType::class` et en second paramètre on va lui donner les données, on va lui donner l'entité `$recipe` qui est déjà rempli avec les données de la base de données.

Maintenant on va l'envoyer à la vue `recipe/edit.html.twig` en plus des données `$recipe` on va lui envoyer `$form`

```php
    #[Route('/recettes/{id}/edit', name: 'recipe.edit', methods: ['GET', 'POST'])]
    public function edit(Recipe $recipe)
    {
        // Je veux utiliser le recipeType que j'ai crée grâce à la commande make:form
        $form = $this->createForm(RecipeType::class, $recipe);
        return $this->render('recipe/edit.html.twig', [
            'recipe' => $recipe,
            'form' => $form
        ]);
    }
```

### Création de la vue edit.html.twig

`edit.html.twig`
```php
{% extends 'base.html.twig' %}

{% block title "Recette : " ~ recipe.title %}


{% block body %}

<h1>{{ recipe.title }}</h1>

{{ form(form) }}

{% endblock %}
```

**Il faut afficher le formulaire.**

Pour cela on va utiliser les **helpers** qui sont des fonctions ou des méthodes qui simplifient la création et l'affichage de formulaire dans nos applications.

Dans Symfony il y a un helper qui est une fonction `form()` et on passera en paramètre le formulaire `{{ form(form) }}` (`['form' => $form]`)

Notre formulaire sera généré dynamiquement.

Le formulaire sera en `method=post` et par défaut il n'y aura pas d'action. Et il y aura un champ caché qui permet de valider que le formulaire provient bien de cette page là, une sécurité mise automatiquement.

### Utilisation d'un thème pour nos formulaires

Aller dans le dossier configuration de Symfony et éditer le fichier de configuration de Twig

Chemin : `config` > `packages` > `twig.yaml`

On va ajouter une nouvelle clé `form_themes: ['bootstrap_5_layout.html.twig']`

twig.yaml
```yaml
twig:
    file_name_pattern: '*.twig'
    form_themes: ['bootstrap_5_layout.html.twig']

when@test:
    twig:
        strict_variables: true

```

`bootstrap_5_layout.html.twig` est un fichier Twig qui va définir les blocks qui permettent de piloter l'affichage des différents champs.

On pourra également utiliser son propre thème.

On peut également piloter l'organisation des champs comme ci-dessous
Lorsque l'on met `{{ form_end(form) }}` il va afficher tous les autres champs

`edit.html.twig`
```php
{% extends 'base.html.twig' %}

{% block title "Recette : " ~ recipe.title %}


{% block body %}

<h1>{{ recipe.title }}</h1>

{{ form_start(form) }}
    <div class="d-flex">
        {{ form_row(form.title) }}
        {{ form_row(form.duration) }}
    </div>
{{ form_rest(form) }}
{{ form_end(form) }}

{% endblock %}
```

Si on souhaite afficher les autres champs qui manquent, il y a une méthode `{{ form_rest(form) }}` qui va afficher tous les champs qui n'ont pas été préalablement affiché dans notre formulaire.

On peut ajouter le bouton directement dans la vue Twig ou dans `RecipeType.php` qui se trouve dans `src\Form` en ajoutant `add('save', SubmitType::class)` en premier paramètre c'est le nom et en second le type de champ a utiliser. Si on souhaite changer le libellé en ajoutera `['label' => 'Envoyer']

On pourra faire la même chose sur les autres champs en précisant le type de champ et son label

```php
public function buildForm(FormBuilderInterface $builder, array $options): void
{
    $builder
        ->add('title', TextType::class, [
            'label' => 'Titre'
        ])
        ->add('slug')
        ->add('content')
        ->add('createdAt', null, [
            'widget' => 'single_text'
        ])
        ->add('updatedAt', null, [
            'widget' => 'single_text'
        ])
        ->add('duration')
        ->add('save', SubmitType::class, [
            'label' => 'Envoyer'
        ]);
}
```

**Documentation formulaire** :
- Explication de toutes les options et comment les utiliser : https://symfony.com/doc/current/forms.html
- Form Types Reference : https://symfony.com/doc/current/reference/forms/types.html

#### UPDATE : Gestion du traitement (mise à jour de l'entité)

Mettre à jour mon entité :

Au niveau du contrôleur `RecipeController.php` on demande à notre formulaire de gérer la requête pour cela on va utiliser la méthode `handleRequest()` et on doit lui passer une requête.
Au niveau du contrôleur on va lui indiquer qu'on a besoin de l'objet `Request` de `HttpFoundation` et je vais le passer à mon formulaire `handleRequest($request)`.

Fonctionnement est le suivant :
1. Le code va vérifier si la requête est en **POST**
2. Si le formulaire a été soumis (s'il y a eu soumission des données), il va modifier l'entité `$recipe` en remplissant ses champs avec les données provenant du formulaire

```php
    #[Route('/recettes/{id}/edit', name: 'recipe.edit', methods: ['GET', 'POST'])]
    public function edit(Recipe $recipe, Request $request)
    {
        $form = $this->createForm(RecipeType::class, $recipe);
        $form->handleRequest($request);

        return $this->render('recipe/edit.html.twig', [
            'recipe' => $recipe,
            'form' => $form
        ]);
    }
```

**En résumé** :

Le formulaire vérifie chaque champ pour voir s'il contient une valeur. Si une valeur est présente, il appelle la méthode **set** correspondante sur notre entité `$recipe`

La méthode `handleRequest` agit comme si nous avions appelé les méthodes setTitle(), setContent() ... pour tous les champs listés dans le fichier `Form\RecipeType.php`. Elle gère la soumission des données du formulaire et les associe à l'entité `$recipe`.

##### Vérification de la soumission du formulaire

- Pour vérifier si le formulaire a été envoyé, on utilise la méthode `isSubmitted()`
- Pour vérifier si le formulaire est valide (si les données soumises sont conformes aux règles de validation) on utilise la méthode `isValid()`

##### Sauvegarde des modifications
Si on souhaite sauvegarder les modifications apportées à l'entité, on utilise la méthode `flush()` sur l'`EntityManager.` 

Pour cela on passe en paramètre de la méthode `edit` l'`EntityManager Interface $em`. 

Ensuite on pourra rediriger l'utilisateur vers une nouvelle route.

```php
    #[Route('/recettes/{id}/edit', name: 'recipe.edit', methods: ['GET', 'POST'])]
    public function edit(Recipe $recipe, Request $request, EntityManagerInterface $em)
    {
        $form = $this->createForm(RecipeType::class, $recipe);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $recipe->setUpdatedAt(new \DateTimeImmutable());
            $em->flush();
            $this->addFlash(
               'success',
               'La recette a bien été modifiée !'
            );
            return $this->redirectToRoute('recipe.index');
        }

        return $this->render('recipe/edit.html.twig', [
            'recipe' => $recipe,
            'form' => $form
        ]);
    }

```

### MESSAGE : Affichage d'un message flash

Un **message flash** est un moyen de stocker temporairement un message dans la session de l'utilisateur. 
Ce message ne sera utilisé que pour la prochaine requête et sera automatiquement effacé après cela. 
Cela permet de transmettre des informations importantes ou des notifications à l'utilisateur sans qu'elles persistent dans la session.

On utilisera la méthode `addFlash()` qui est disponible dans le `AbstractController`.
On précisera en premier paramètre le type, et en second le message

```php
$this->$this->addFlash(
    'success',
    'La recette a bien été modifiée !'
);
```

Ensuite on va éditer notre vue `templates\base.html.twig` et ajouter `{{ dump(app.flashes) }}`
```php
        <main>
            <div class="container my-4">
                {{ dump(app.flashes) }}
                {% block body %}{% endblock %}
            </div>
        </main>

```

#### Création d'un template pour les messages flash

On créer dans `\templates` un repertoire `partials` qui contiendra notre vue pour les alertes.

partials : des bouts de vue Twig

On inclut notre fichier Twig `flash.html.twig`

```php
        <main>
            <div class="container my-4">
                {% include "partials/flash.html.twig" %}
                {% block body %}{% endblock %}
            </div>
        </main>
```

Dans `flash.html.twig`
```php
{% for type, messages in app.flashes %}
    <div class="alert alert-{{ type }}">
        {{ message | join('. ') }}
    </div>
{% endfor %}
```

Join : On a un tableau de valeurs et l'on peut choisir par quoi on les fusionne

Documentations :
- **Iterating over Keys and Values** : https://twig.symfony.com/doc/3.x/tags/for.html#iterating-over-keys-and-values
- **join** : https://twig.symfony.com/doc/3.x/filters/join.html


Affiche les messages et on va les joindre par un point séparé d'un espace
```php
{{ message | join('. ') }}
```

### CREATE : Création d'un bouton ajout de recette

On ouvre le fichier `\recipe\index.html.twig` et on crée un tableau avec un bouton qui utilise la route `recipe.edit`. 

On crée un bouton Créer une recette et on va créer une route `recipe.create`.

```php
<p>
    <a href="{{ path('recipe.create') }}" class="btn btn-primary btn-sm">Créer une recette</a>
</p>

<table class="table">
    <thead>
        <tr>
            <th>Titre</th>
            <th>Actions</th>
        </tr>
    </thead>
    <tbody>
        {% for recipe in recipes %}
            <tr>
                <td>
                    <a href="{{ path('recipe.show', {id: recipe.id, slug: recipe.slug}) }}">{{ recipe.title }}</a>
                </td>
                <td>
                    <a href="{{ path('recipe.edit', {id: recipe.id}) }}" class="btn btn-primary btn-sm">Editer</a>
                </td>
            </tr>
        {% endfor %}
    </tbody>
</table>
```

Recipe.Controller.php
```php
    #[Route('/recettes/create', name: 'recipe.create')]
    public function create(Request $request, EntityManagerInterface $em)
    {
        $recipe = new Recipe();
        $form = $this->createForm(RecipeType::class, $recipe);
        $form->handleRequest($request);
        if ($form->isSubmitted() && $form->isValid()) {
            $recipe->setCreatedAt(new \DateTimeImmutable());
            $recipe->setUpdatedAt(new \DateTimeImmutable());
            $em->persist($recipe);
            $em->flush();
            $this->addFlash('success', 'La recette a bien été créée');
            return $this->redirectToRoute('recipe.index');
        }
        return $this->render('recipe/create.html.twig', [
            'formTitle' => 'Créer une nouvelle recette',
            'form' => $form
        ]);
    }
```

### DELETE : Suppression d'une recette

```php
    #[Route('/recettes/{id}', name: 'recipe.delete', methods: ['DELETE'])]
    public function delete(Recipe $recipe, EntityManagerInterface $em)
    {
        $em->remove($recipe);
        $em->flush();
        $this->addFlash('success', 'La recette a bien été supprimée');
        return $this->redirectToRoute('recipe.index');
    }
```

`recipe\index.html.twig`

```php
<p>
    <a href="{{ path('recipe.create') }}" class="btn btn-primary btn-sm">Créer une recette</a>
</p>

<table class="table">
    <thead>
        <tr>
            <th>Titre</th>
            <th>Actions</th>
        </tr>
    </thead>
    <tbody>
        {% for recipe in recipes %}
            <tr>
                <td>
                    <a href="{{ path('recipe.show', {id: recipe.id, slug: recipe.slug}) }}">{{ recipe.title }}</a>
                </td>
                <td>
                    <a href="{{ path('recipe.edit', {id: recipe.id}) }}" class="btn btn-primary btn-sm">Editer</a>
                    <form action="{{ path('recipe.delete', {id: recipe.id}) }}" method="post">
                        <input type="hidden" name="_method" value="DELETE">
                        <button type="submit" class="btn btn-danger btn-sm">Supprimer</button>
                    </form>
                </td>
            </tr>
        {% endfor %}
    </tbody>
</table>
```

Dans le fichier `config\packages\framework.yaml`

On ajoute la clé `http_method_override: true` pour activer la substitution de méthode HTTP. Cette fonctionnalité permet de modifier la méthode HTTP utilisée grâce à un champ caché de type `"hidden"` dans les formulaires.

```php
# see https://symfony.com/doc/current/reference/configuration/framework.html
framework:
    secret: '%env(APP_SECRET)%'
    #csrf_protection: true

    # Note that the session will be started ONLY if you read or write from it.
    session: true

    #esi: true
    #fragments: true
    http_method_override: true

when@test:
    framework:
        test: true
        session:
            storage_factory_id: session.storage.factory.mock_file


```

**Documentation* :
- http_method_override : https://symfony.com/doc/current/reference/configuration/framework.html#http-method-override