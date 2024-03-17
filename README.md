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

Tous les setters sont des setters Fluent c'est-à-dire que lorsqu'ils ont fini de faire le setter ils retournent l'instance de l'objet, ça permet d'enchainer les différentes méthodes les unes sur les autres. L'avantage, on n'est pas obligé de redéfinir les choses.
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


### AutoSlug

Lorsqu'un formulaire est soumis, il se déroule généralement en 3 étapes :

1. **PRE_SUBMIT** : Cet événement se produit avant que les données du formulaire ne soient effectivement soumises. À ce stade, vous pouvez effectuer des validations supplémentaires ou des modifications sur les données du formulaire avant qu'elles ne soient traitées.
2. **SUBMIT** : Lorsque le formulaire est soumis, les informations saisies par l'utilisateur sont envoyées au serveur. Dans cette phase, les données sont validées, nettoyées et associées à votre modèle (ou entité) côté serveur. C'est là que les informations sont enregistrées dans la base de données ou utilisées pour d'autres traitements.
3. **POST_SUBMIT** : Après que les données ont été traitées et enregistrées, cet événement intervient. Vous pouvez effectuer des actions supplémentaires ici, telles que l'envoi d'e-mails de confirmation, la mise à jour d'autres entités liées, etc.

Documentation : 
- Form Event : https://symfony.com/doc/current/form/events.html

`Form\RecipeType.php`
```php
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('title')
            ->add('slug')
            ->add('content')
            ->add('duration')
            ->add('save', SubmitType::class, [
                'label' => 'Envoyer'
            ])
            ->addEventListener(FormEvents::PRE_SUBMIT, $this->autoSlug(...));
    }

    public function autoSlug()
    {
        
    }
```

`autoSlug(...)` : on crée un callable `...` à partir de la méthode `autoSlug()`.

**callable** : le type callable est un moyen de dire à votre code : "Hé, quand ceci se produit, exécute cette fonction". Il permet à une fonction d'appeler une autre fonction (ou méthode) au bon moment.

`Form\RecipeType.php`
```php
    public function autoSlug(PreSubmitEvent $event): void
    {
        // Récupère les données du formulaire à partir de l'événement
        $data = $event->getData();

        // Vérifie si le champ "slug" est vide
        if (empty($data['slug'])) {
            // Crée une instance de la classe AsciiSlugger
            $slugger = new AsciiSlugger();
             // Génère un slug à partir du titre et convertit en minuscules
            $data['slug'] = strtolower($slugger->slug($data['title']));
        }
         // Réinjecte les données mises à jour dans l'événement
        $event->setData($data);
    }
```

`autoSlug()` recevra en paramètre un `PreSubmitEvent`

`getData()` : Permet de récupérer les données qui ont été posté
`setData()` : Permet de modifier les données

Documentation :
- **Slugger** : https://symfony.com/doc/current/components/string.html#slugger


### attachTimestamps

`Form\RecipeType.php`
```php
    public function attachTimestamps(PostSubmitEvent $event): void
    {
        // Récupère les données associées à l'événement
        $data = $event->getData();

        // Vérifie si $data est une instance de la classe Recipe
        if (!($data instanceof Recipe)) {
            return; // Quitte la fonction si ce n'est pas une Recipe
        }

        // Met à jour la propriété updatedAt avec la date et l'heure actuelles
        $data->setUpdatedAt(new DateTimeImmutable());

        // Si l'objet n'a pas d'ID, initialise la propriété createdAt avec la date actuelle
        if (!$data->getId()) {
            $data->setCreatedAt(new DateTimeImmutable());
        }
    }

```

La fonction `attachTimestamps` met à jour les horodatages (`updatedAt` et `createdAt`) d'un objet `Recipe` lorsqu'un événement `PostSubmitEvent` se produit. 
Si l'objet `$data` n'est pas une instance de `Recipe`, la fonction se termine.

## Validation de formulaire

### Les contraintes au niveau du formulaire

`Form\RecipeType.php`
```php
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('title')
            ->add('slug', TextType::class, [
                'required' => false,
                'constraints' => [
                    new Length(min: 10),
                    new Regex('/^[a-z0-9]+(?:-[a-z0-9]+)*$/', message: "Ceci n'est pas un slug valide")
                ]
            ])
            ->add('content')
            ->add('duration')
            ->add('save', SubmitType::class, [
                'label' => 'Envoyer'
            ])
            ->addEventListener(FormEvents::PRE_SUBMIT, $this->autoSlug(...))
            ->addEventListener(FormEvents::POST_SUBMIT, $this->attachTimestamps(...));
    }
```

Documentation :
- **Validation Constraints** : https://symfony.com/doc/current/reference/constraints.html
- Contrainte `Length()` : https://symfony.com/doc/current/reference/constraints/Length.html
- Regex : https://symfony.com/doc/current/reference/constraints/Regex.html

Outils :
- Regex for url slug : https://ihateregex.io/expr/url-slug/

Autres contraintes intéressantes :
- `All()` permet d'appliquer une contrainte à tous les éléments d'un tableau : https://symfony.com/doc/current/reference/constraints/All.html
- `AtLeastOneOf()`
- `Sequentially()` : ça permet de couper une validation si une des règles de validation précédente n'est pas remplie. Et ainsi afficher qu'un seul message (pour éviter de spammer l'utilisateur d'information).

`Form\RecipeType.php`
```php
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('title')
            ->add('slug', TextType::class, [
                'required' => false,
                'constraints' => new Sequentially([
                    new Length(min: 10),
                    new Regex('/^[a-z0-9]+(?:-[a-z0-9]+)*$/', message: "Ceci n'est pas un slug valide")
                ])
            ])
            ->add('content')
            ->add('duration')
            ->add('save', SubmitType::class, [
                'label' => 'Envoyer'
            ])
            ->addEventListener(FormEvents::PRE_SUBMIT, $this->autoSlug(...))
            ->addEventListener(FormEvents::POST_SUBMIT, $this->attachTimestamps(...));
    }
```

### Les contraintes au niveau de l'entité

`Entity\Recipe.php`
```php
#[ORM\Entity(repositoryClass: RecipeRepository::class)]
class Recipe
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[ORM\Column(length: 255)]
    #[Assert\Length(min: 5)]
    private ?string $title = null;

    #[ORM\Column(length: 255)]
    #[Assert\Length(min: 5)]
    #[Assert\Regex('/^[a-z0-9]+(?:-[a-z0-9]+)*$/', message: 'Invalid slug')]
    private ?string $slug = null;

    #[ORM\Column(type: Types::TEXT)]
    #[Assert\Length(min: 5)]
    private ?string $content = null;

    #[ORM\Column]
    private ?\DateTimeImmutable $createdAt = null;

    #[ORM\Column]
    private ?\DateTimeImmutable $updatedAt = null;

    #[ORM\Column(nullable: true)]
    #[Assert\NotBlank()]
    #[Assert\Positive()]
    private ?int $duration = null;
```



`Entity\Recipe.php`
```php
  #[ORM\Column(nullable: true)]
    #[Assert\NotBlank()]
    #[Assert\Positive()]
    private ?int $duration = null;

```

`NotBlank()` : on n'accepte pas de valeur `null` (il ajoute un attribut required sur le champ)
`Positive()` : la valeur positive ne va agir que si la valeur est un entier

Documentation :
- Number Positive : https://symfony.com/doc/current/reference/constraints/Positive.html

> [!IMPORTANT]
> Il est conseillé de mettre les **contraintes directement dans les entités**. 
> On mettra les contraintes dans le form, uniquement si on a des champs qui ne correspondent à rien au niveau de l'entité.


**Contrainte qui se met directement au niveau de la class** :

- `UniqueEntity()` : https://symfony.com/doc/current/reference/constraints/UniqueEntity.html

On peut lui passer en paramètre un tableau, mais dans ce cas c'est la combinaison des champs qui sera unique.

```php
#[ORM\Entity(repositoryClass: RecipeRepository::class)]
#[UniqueEntity('title')]
#[UniqueEntity('slug')]
class Recipe
```

Contrainte : Recette de cuisinne de moins de 24h
```php
    #[ORM\Column(nullable: true)]
    #[Assert\Positive()]
    #[Assert\LessThan(value: 1440)]
    private ?int $duration = null;
```
`LessThan()` : https://symfony.com/doc/current/reference/constraints/LessThan.html


### Contraintes personnalisées

#### Création d'une liste de mots bannies

On ouvre un terminal

```shell
$ php bin/console make:validator

 The name of the validator class (e.g. EnabledValidator):
 > BanWordValidator

 created: src/Validator/BanWordValidator.php
 created: src/Validator/BanWord.php

 
  Success! 
 

 Next: Open your new constraint & validators and add your logic.
 Find the documentation at http://symfony.com/doc/current/validation/custom_constraint.html
```

Le dossier Validator est crée et contient 2 fichiers :

`src/Validator/BanWordValidator.php`
```php
namespace App\Validator;

use Symfony\Component\Validator\Constraint;
use Symfony\Component\Validator\ConstraintValidator;

class BanWordValidator extends ConstraintValidator
{
    public function validate($value, Constraint $constraint)
    {
        /* @var BanWord $constraint */

        if (null === $value || '' === $value) {
            return;
        }

        $value = strtolower($value);
        foreach ($constraint->banWords as $banWord) {
            if (str_contains($value, $banWord)) {
                $this->context->buildViolation($constraint->message)
                ->setParameter('{{ banWord }}', $banWord)
                ->addViolation();
            }
        }     
    }
}
```

`src/Validator/BanWord.php`
```php
namespace App\Validator;

use Symfony\Component\Validator\Constraint;

#[\Attribute(\Attribute::TARGET_PROPERTY | \Attribute::TARGET_METHOD | \Attribute::IS_REPEATABLE)]
class BanWord extends Constraint
{
    public function __construct(
        public string $message = 'This contains a banned word "{{ banWord }}".',
        public array $banWords = ['spam', 'viagra'],
        ?array $groups = null,
        mixed $payload = null
        ) 
    {
        parent::__construct(null, $groups, $payload);
    }
}
```

####  Création de groups :

Permets d'activer ou de désactiver des règles de validation en fonction de situation.

Par exemple, lorsqu'on crée une entité, on a moins de contraintes de validation que lorsqu'on l'édite. Dans ce cas là ça peut être intéressant d'associer des groupes particuliers à nos différentes règles pour pouvoir par la suite facilement depuis un formulaire activer ou désactiver certaines règles.

`\Entity\Recipe.php`
```php
    #[ORM\Column(length: 255)]
    #[Assert\Length(min: 5, groups: ['Extra'])]
    #[BanWord(groups:['Extra'])]
    private ?string $title = null;
```

`\form\RecipeType.php`
```php
    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => Recipe::class,
            'validation_groups' => ['Default', 'Extra']
        ]);
    }
```

### La valeur vide pour les champs (null)

> [!IMPORTANT]
> Pensez à ajouter la contrainte `NotBlank()` au niveau des Assert pour s'assurer qu'un champ ne soit pas laissé vide. Par défaut les règles de validation vont laisser passer un champ vide.

Si on met un champ title vide on va avoir une erreur par défaut le formulaire va considérer que cette valeur est nulle et il va essayer d'appeler setTitle() en lui donnant une valeur nulle

Pourtant dans le code ci-dessous on lui a dit c'est forcement une chaine de caractère.
`Entity\Recipe.php`
```php
public function setTitle(string $title): static
```

Une solution c'est dans le champ on peut spécifier la valeur à associer si le champ est vide en utilisant la clé `empty_data` et on lui donnera une chaine de caractère vide `''`.

`Form\RecipeType.php`
```php
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('title', TextType::class, [
                'empty_data' => ''
            ])
            ->add('slug', TextType::class, [
                'required' => false,
            ])
            ->add( 'content', TextareaType::class, [
                'empty_data' => ''
            ])
            ->add('duration')
            ->add('save', SubmitType::class, [
                'label' => 'Envoyer'
            ])
            ->addEventListener(FormEvents::PRE_SUBMIT, $this->autoSlug(...))
            ->addEventListener(FormEvents::POST_SUBMIT, $this->attachTimestamps(...));
    }
```

On peut également dire dans l'entité Recipe que le titre n'est jamais nul, et qu'il est toujours une chaine de caractère par défaut, en remplaçant `?string $title = null` par `string $title = ''` pour faire en sorte d'avoir des types de variables qui sont cohérents par rapport à ce que l'on définit au niveau des setters. 
On ne travaillera plus avec des null et ça permettra d'éviter des erreurs par la suite.

```php
    #[ORM\Column(length: 255)]
    #[Assert\Length(min: 5)]
    #[BanWord]
    private string $title = '';

    #[ORM\Column(length: 255)]
    #[Assert\Length(min: 5)]
    #[Assert\Regex('/^[a-z0-9]+(?:-[a-z0-9]+)*$/', message: 'Invalid slug')]
    private string $slug = '';

    #[ORM\Column(type: Types::TEXT)]
    #[Assert\Length(min: 5)]
    private string $content = '';

    // ... code

    public function getTitle(): string
    {
        return $this->title;
    }

    public function getSlug(): string
    {
        return $this->slug;
    }

    public function getContent(): string
    {
        return $this->content;
    }
```

## Les services

Comment ça fonctionne ?

Au coeur du framework on a un système de container, c'est une sorte de gros objet qui va contenir l'ensemble des classes dont on a besoin et qui va aussi contenir les méthodes qui vont permettre de les construire.

Un container c'est une classe qui va contenir des clés et pour chaque clé on va avoir une fonction qui va permettre de construire le service dont on a besoin d'instancier si vous voulez le classe dont en a besoin.

Cette commande va nous montrer l'ensemble des classes qui sont disponible dans le système de container de Symfony :
```shell
php bin/console debug:autowiring

```

Si on cherche une classe en particulier, on tape la même commande en rajoutant un nom
```shell
php bin/console debug:autowiring form

```

Exemple : on a envie de valider les données mais en dehors d'un système de formulaire , on peut chercher tout ce qui contient valide
```shell
$ php bin/console debug:autowiring valid

Autowirable Types
=================

 The following classes & interfaces can be used as type-hints when autowiring:
 (only showing classes/interfaces matching valid)

 Validates PHP values against constraints.
 Symfony\Component\Validator\Validator\ValidatorInterface - alias:validator

 2 more concrete services would be displayed when adding the "--all" option.
```
On a un alias validator `alias:validator` et on nous donne l'interface qui correspond `Symfony\Component\Validator\Validator\ValidatorInterface` ce qui fait que dans une action de mon contrôleur si j'injecte cette interface, il va être capable de résoudre le service qui implémente cette interface de manière automatique


On créer une fonction demo, on crée la route qui correspond `#[Route('/demo')]` au niveau de ce demo j'attend en premier paramètre quelque chose qui va implémenter la `ValidatorInterface`. C'est cette interface là que j'avais dans la liste des services et on sauvegarde ça dans une variable `$validator`

`src\Controller\RecipeController.php`
```php
    use Symfony\Component\Validator\Validator\ValidatorInterface;

    #[Route('/demo')]
    public function demo(ValidatorInterface $validator)
    {
        dd($validator);
    }

```

Si l'on va sur `localhost:8000/demo`, il va me donner une instance de mon validateur.
```
RecipeController.php on line 21:
Symfony\Component\Validator\Validator\TraceableValidator {#4276 ▼
  -validator: Symfony\Component\Validator\Validator\RecursiveValidator {#1483 ▶}
  -collectedData: []
}
```
Le système de container sait que pour cette interface là il va falloir instancier la classe `TraceableValidator`
Après on peut l'utiliser
On instancie une nouvelle Recipe `$recipe = new Recipe()`.
On va faire un Validator et on va lui demander est-ce que tu peux me valider ma recette `validate($recipe)`.

La méthode `validate()` va nous renvoyer une liste d'erreur. On va sauvegarder ça dans une variable `$errors`. Et on va afficher les erreurs sous forme de chaîne de caractères `dd((string)$errors)`

`src\Controller\RecipeController.php`
```php
    use Symfony\Component\Validator\Validator\ValidatorInterface;

    #[Route('/demo')]
    public function demo(ValidatorInterface $validator)
    {
        $recipe = new Recipe();
        $errors = $validator->validate($recipe);
        dd((string)$errors);
    }
```

En résumé : On a injecté un service à la volée `ValidatorInterface` qui me permet de faire un traitement particulier.

### Résolution de classe `namespace App`

Symfony va être capable de faire cette résolution pour les classes qui proviennent de notre `namespace App`

A la racine du dossier `src` on crée une nouvelle classe `Demo.php`
`src\Demo.php`
```php
<?php

namespace App;

class Demo
{
    
}

```

Je veux avoir une instance de `Demo`, il a automatiquement fourni une instance de la classe `Demo`

`src\Controller\RecipeController.php`
```php
use App\Demo;

#[Route('/demo')]
public function demo(Demo $demo)
{
    dd($demo);
}
```

Pour faire celà automatiquement

Dans le dossier `config` on a un fichier qui s'appelle `services.yaml`. Ce fichier va permettre d'enregistrer des services. 

> [!IMPORTANT]
> Un **service** c'est une class que l'on va pouvoir ensuite brancher de manière automatique dans nos contrôleurs.

`config\services.yaml`
```php
# This file is the entry point to configure your own services.
# Files in the packages/ subdirectory configure your dependencies.

# Put parameters here that don't need to change on each machine where the app is deployed
# https://symfony.com/doc/current/best_practices.html#use-parameters-for-application-configuration
parameters:

services:
    # default configuration for services in *this* file
    _defaults:
        autowire: true      # Automatically injects dependencies in your services.
        autoconfigure: true # Automatically registers your services as commands, event subscribers, etc.

    # makes classes in src/ available to be used as services
    # this creates a service per class whose id is the fully-qualified class name
    App\:
        resource: '../src/'
        exclude:
            - '../src/DependencyInjection/'
            - '../src/Entity/'
            - '../src/Kernel.php'

    # add more service definitions when explicit configuration is needed
    # please note that last definitions always *replace* previous ones

```

`autowire: true` : ça veut dire qu'il va être capable de manière automatique d'injecter les dépendances.
`autoconfigure: true` : ça permet pour certaines calsses de rajouter un comportement particulier. ex: lorsqu'on crée des classes pour gérer les événements, automatiquement il va les enregister.

Tout ce que est dans le namespace `App\` et qui n'est pas dans `../src/DependencyInjection/`, `../src/Entity/` et `../src/Kernel.php` va constituer un service valide.

C'est pour ça que `Demo` qui est dans le namespace `App` est automatiquement enregistrée et peut être utilisé comme un service au niveau de nos contrôleurs.

#### Injection de dépendances

J'ai besoin d'une instance de Validator.

L'autowiring va reconnaitre `ValidatorInterface` dans mon service `container` et donc quand je vais construire l'objet démo, il va pouvoir directement nous donner la bonne valeur.

`src\Demo.php`
```php
namespace App;

use Symfony\Component\Validator\Validator\ValidatorInterface;

class Demo
{
    public function __construct(private ValidatorInterface $validator)
    {
        
    }
}

```

> [!WARNING]
> Si on lui donne un paramètre (ex: clé API `string $key`), il va être bloqué lorsqu'il va essayer d'instancier cet objet car il connait le `ValidatorInterface` grâce à l'autowiring par contre la clé il ne saura pas le faire. On devra définir un nouveau service dans `services.yaml` pour gérer ce cas de figure.

`src\Demo.php`
```php
    public function __construct(private ValidatorInterface $validator, private string $key)
    {
        
    }
```

Pour palier à cela, on pourra enregistrer de nouveaux services directement dans le fichier `services.yaml`
Comme ça lorsque le container va essayé de résoudre notre classe Demo, il va utiliser la définition dans `services.yaml` et être capable de comprendre que l'argument qui s'appelle clé doit avoir cette chaîne de caractères.

`config\services.yaml`
```php
    # add more service definitions when explicit configuration is needed
    # please note that last definitions always *replace* previous ones
    App\Demo:
        class: App\Demo
        arguments:
            $key: 'hello'

```

Là il sera capable d'instancier `Demo` avec le `ValidatorInterface` et la clé `$key` qui a été automatiquement remplie.


**ManagerRegistry**

On a le même mode de fonctionnement lorsqu'on injecte un Repository.

`src\Repository\RecipeRepository.php`
```php
    public function __construct(ManagerRegistry $registry)
    {
        parent::__construct($registry, Recipe::class);
    }
```
On a un constructeur `__construct` qui a besoin du `ManagerRegistry`

On regarde à quoi il correspond en faisant un debug:autowiring
```shell
$ php bin/console debug:autowiring ManagerRegistry

Autowirable Types
=================

 The following classes & interfaces can be used as type-hints when autowiring:
 (only showing classes/interfaces matching ManagerRegistry)

 Doctrine\Common\Persistence\ManagerRegistry - alias:doctrine

 Contract covering object managers for a Doctrine persistence layer ManagerRegistry class to implement.
 Doctrine\Persistence\ManagerRegistry - alias:doctrine

 1 more concrete service would be displayed when adding the "--all" option.
```

`ManagerRegistry` est un service reconnu par Symfony, et Symfony va automatiquement l'injecter. C'est ce qui permet de se connecter à la base de données et de récupérer des informations.

##### Autowiring (cablage automatique)

L'`autowiring` fonctionne à 2 endroits :


1. Il fonctionne sur un service qui est injecté sur le container.

Comme ci-dessous dans une construction `__construct` ou on a injecté des éléments `ValidatorInterface` provenant de notre service `container`

`src\Demo.php`
```php
public function __construct(private ValidatorInterface $validator)
```

2. Il fonctionne aussi automatiquement dans les contrôleurs
Dans un contrôleur lorsqu'on crée une action (une méthode) on a la possibilité d'injecter autant de services que l'on souhaite. On peut le faire au niveau de la méthode.

`src\Controller\RecipeController.php`
```php
    #[Route('/demo')]
    public function demo(Demo $app)
    {

    }
```

> [!IMPORTANT] 
> Si on a plusieurs méthodes qui ont besoin de la même chose, on définira un constructeur `__construct` et je le déclare comme propriété privée `private` à l'intérieur de mon contrôleur. Ce qui permet d'accéder au Repository quand on le souhaite, et on y accèdera comme une instance.

```php
    public function __construct(private RecipeRepository $repository)
    {
        
    }
```

On remplacera :
- `public function index(RecipeRepository $repository): Response` par `public function index(): Response`
- `$recipes = $repository->findWithDurationLowerThan(20)` par `$recipes = $this->repository->findWithDurationLowerThan(20)` car on y accède comme une instance.


```php
    #[Route('/recettes', name: 'recipe.index')]
    public function index(): Response
    {
        $recipes = $this->repository->findWithDurationLowerThan(20);

        return $this->render('recipe/index.html.twig', [
            'recipes' => $recipes
        ]);
    }
```

Une manière d'obtenir un service c'est de faire par exemple ce qui est fait dans les méthodes `createForm` en accédant directement au `container`.
Si je souhaite l'instance du `Validator`, on peut faire un `$this->container` et lui demander est-ce que tu peux me donner `get('validator')` le service correspondant à la validation (on peut se référer au debug:autowiring), c'était l'alias `validator`.

> [!NOTE]
> Ce `container` n'est accessible que dans les contrôleurs qui `extends AbstractController`

```php
    #[Route('/recettes', name: 'recipe.index')]
    public function index(): Response
    {
        dd($this->container->get('validator'));
        $recipes = $this->repository->findWithDurationLowerThan(20);

        return $this->render('recipe/index.html.twig', [
            'recipes' => $recipes
        ]);
    }
```

```shell
$ php bin/console debug:autowiring valid

Autowirable Types
=================

 The following classes & interfaces can be used as type-hints when autowiring:
 (only showing classes/interfaces matching valid)

 Validates PHP values against constraints.
 Symfony\Component\Validator\Validator\ValidatorInterface - alias:validator

 2 more concrete services would be displayed when adding the "--all" option.
```

`dd($this->container->get('validator'))` : donne une instance du validateur mais provoque une erreur.

**Erreur**
```
Service "validator" not found: even though it exists in the app's container, the container inside "App\Controller\RecipeController" is a smaller service locator that only knows about the "form.factory", "http_kernel", "parameter_bag", "request_stack", "router", "security.authorization_checker", "security.csrf.token_manager", "security.token_storage", "serializer", "twig" and "web_link.http_header_serializer" services. Try using dependency injection instead.
```

Cette erreur va nous permettre d'expliquer la notion de **public**. Lorsqu'on enregistre un service on a la possibilité de préciser s'il est `public` ou non. S'il est `public` il va être accessible dans le `container`.

Exemple : le form.factory est un service qui est public, donc lorsqu'on l'utilise on peut l'obtenir
```php
dd($this->container->get('form.factory'));
```

Par contre le `validator` n'est pas `public`, il ne peut pas être accessible directement depuis le `container`. C'est ce qu'indique l'erreur, on ne peut y accéder que lorsqu'on fait une construction `__construct` il peut être automatiquement injecté mais il ne peut pas être récupé de cette manière.

> [!NOTE]
> De manière générale il est déconseillé d'utiliser le `container`, il est utilisé en interne sur Symfony pour quelques éléments pratiques comme par exemple le `form.factory` et `render`. On préféra utiliser l'injection car ça permet de mieux comprendre les dépendances qui sont liées à une action (méthode), de quoi l'action à besoin pour fonctionner, ça sera intéressant dans le cadre des tests.

Pour le formulaire d'édition de recette on va injecter le `FormFactoryInterface`

On pourrait remplacer :
```php
    public function edit(
        Recipe $recipe, 
        Request $request, 
        EntityManagerInterface $em)
    {
        $form = $this->createForm(RecipeType::class, $recipe);
```
Par :
```php
    public function edit(
        Recipe $recipe, 
        Request $request, 
        EntityManagerInterface $em, 
        FormFactoryInterface $formFactory)
    {

        $form = $formFactory->create(RecipeType::class, $recipe);
```

#### En résumé

Symfony est un gros container qui va contenir plein de services préconfigurés qui vont nous permettre de travailler plus rapidement.

Lorsqu'on a installé la partie squelette de Symfony `composer create-project symfony/skeleton:"7.0.*" my_project_directory` et qu'on avait pas fait le `composer require webapp`. C'est ce que l'on a, on a le container de Symfony qui est casiment vide avec juste les fonctionnalités de base. Lorsqu'on a installé webapp il a rajouté toutes les dépendances qui sont venu remplir notre container avec des éléments utiles. C'est ce qui permet d'être un framework complet dans le cadre de la création d'application web.

#### A retenir
Lorsqu'on a besoin de fonctionnalité particulières on peut se reposer sur des classes qui proviennent de Symfony. Ces classes sont enregistrées sous forme de service dans le service **container**, mais on a pas besoin d'intéragir directement avec lui on peut utiliser le système d'injection qui se fait à la fois au niveau des actions (méthodes) et aussi au niveau des constructeurs.
Si plus tard on souhaite créer des classes on peut les créer dans notre namespace **App**, on peut les placer ou on le souhaite et on peut utiliser dans leur constructeur des éléments qui proviennent du container de service et automatiquement lorsque l'on va les utiliser dans nos actions ou dans d'autres classes ça va être hydrater correctement grâce à ce système d'**autowiring**

> [!NOTE]
> La plupart du temps ce sont des **singletons** qui sont renvoyés c'est à dire que l'EntityManager que j'utilise ça va être le même qui est utilisé en constructions dans d'autres classes.

**hydrater** : signifie associer un objet (ou une classe) avec des données provenant d'une source externe, comme une base de données. L'hydratation consiste à remplir les propriétés d'un objet avec les valeurs correspondantes provenant de ces données.

Exemple : Lorsqu'on récupères des informations d'un utilisateur depuis une base de données, on hydrate l'objet représentant cet utilisateur avec les valeurs appropriées (nom, prénom, adresse, etc.)

**singleton** : c'est un patron de conception (design pattern) qui garantit qu'une classe possède une seule instance tout au long de l'exécution de l'application. Il permet d'accéder à cette instance depuis n'importe quel endroit du code.

Exemple : Si on a une classe de gestion de configuration qui stocke des paramètres globaux pour une application, on peut la concevoir comme un singleton. Ainsi, il n'existera qu'une seule instance de cette classe, et on pourra y accéder facilement depuis n'importe quel autre composant de l'application

## Formulaire de contact

Créer une page contact accessible via /contacct qui demande à l'utilisateur son nom, son email et un message

On utilisera un objet `ContactFormDTO` pour représenter les données de ce formulaire (pas une entité car on ne sauvegarde pas en base de données).

DTO (Data Transfert Object) : c'est un objet qui permet de représenter des données qui sont transférées.

On utilisera mailpit ou maildev pour tester la réception d'email.



### Modification de `messenger.yaml`

Messenger est un système qui permet de gérer des files d'attente et par défaut les mails s'envoient par défaut sur messenger. Il va falloir donc modifier le fichier `config\packages\messenger.yaml` pour changer son comportement.

Modifier le fichier `config\packages\messenger.yaml`
```yaml
            # sync: 'sync://'

        routing:
            Symfony\Component\Mailer\Messenger\SendEmailMessage: async
            Symfony\Component\Notifier\Message\ChatMessage: async
            Symfony\Component\Notifier\Message\SmsMessage: async
```
En 

```yaml
            sync: 'sync://'

        routing:
            Symfony\Component\Mailer\Messenger\SendEmailMessage: sync
            Symfony\Component\Notifier\Message\ChatMessage: sync
            Symfony\Component\Notifier\Message\SmsMessage: sync
```

Pour le passer de synchrone pour qu'elle ne passe pas par Messenger.

### Installation de serveur mail

- Maildev (nécessite NodeJs) https://maildev.github.io/maildev/
- Mailpit (est un executable) https://github.com/axllent/mailpit/releases/

Sous Windows : mettre le fichier mailpit.exe dans le dossier `\bin` de Symfony
```shell
C:\wamp64\www\symfony>cd bin

C:\wamp64\www\symfony\bin>mailpit
time="2024/03/17 10:03:49" level=info msg="[smtpd] starting on [::]:1025 (no encryption)"
time="2024/03/17 10:03:49" level=info msg="[http] starting on [::]:8025"
time="2024/03/17 10:03:49" level=info msg="[http] accessible via http://localhost:8025/"
```

Sous Linux : mettre le fichier mailpit dans le dossier `\bin` de Symfony
Executer la commande dans le terminal
```shell
chmod +x bin/mailpit
./bin/mailpit
```

Ouvrir dans le navigateur `http://localhost:8025/` pour accéder à l'interface de Mailpit.

### Configuration `.env` pour l'email

On configure notre environnement pour qu'il envoit les emails sur Mailpit.

```shell
###> symfony/mailer ###
MAILER_DSN=smtp://localhost:1025
###< symfony/mailer ###

```

### Création du formulaire

Dans `\src` créer un répertoire pour avoir un namespace dédié `DTO`, puis créer la classe `ContactDTO`.

`src\DTO\ContactDTO.php`
```php
<?php

namespace App\DTO;

class ContactDTO
{
    public string $name = '';
    public string $email = '';
    public string $message = '';
}
```

On crée le formulaire ContactType

```shell
$ php bin/console make:form ContactType

 The name of Entity or fully qualified model class name that the new form will be bound to (empty for none):
 > \App\DTO\ContactDTO
\App\DTO\ContactDTO

 created: src/Form/ContactType.php

 
  Success! 
 

 Next: Add fields to your form and start using it.
 Find the documentation at https://symfony.com/doc/current/forms.html
```

`src\Form\ContactType.php`
```php
namespace App\Form;

use App\DTO\ContactDTO;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class ContactType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('name')
            ->add('email')
            ->add('message')
        ;
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => ContactDTO::class,
        ]);
    }
}

```

On définit le type des champs de notre formulaire
```php
namespace App\Form;

use App\DTO\ContactDTO;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\EmailType;
use Symfony\Component\Form\Extension\Core\Type\TextareaType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class ContactType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('name', TextType::class, [
                'empty_data' => ''
            ])
            ->add('email', EmailType::class, [
            'empty_data' => ''
            ])
            ->add('message', TextareaType::class, [
                'empty_data' => ''
            ])
        ;
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => ContactDTO::class,
        ]);
    }
}
```

On crée le contrôleur `ContactController`
```shell
$ php bin/console make:controller ContactController
 created: src/Controller/ContactController.php
 created: templates/contact/index.html.twig

 
  Success! 
 

 Next: Open your new controller class and add some pages!
```

`src\Controller\ContactController.php`
```php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class ContactController extends AbstractController
{
    #[Route('/contact', name: 'app_contact')]
    public function index(): Response
    {
        return $this->render('contact/index.html.twig', [
            'controller_name' => 'ContactController',
        ]);
    }
}

```

#### 1er étape

On créer la méthode `contact`, on génère le formulaire et son gabarit, on définit les types de champs et on ajoute une condition pour testé s'il a été envoyé et s'il est valide.

`ContactController.php`
```php
namespace App\Controller;

use App\DTO\ContactDTO;
use App\Form\ContactType;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;

class ContactController extends AbstractController
{
    #[Route('/contact', name: 'contact')]
    public function contact(Request $request): Response
    {
        $data = new ContactDTO();

        // TODO : A Supprimer (nécessaire que pour les tests de validation)
        $data->name = 'John Doe';
        $data->email = 'john';
        $data->message = 'Message de John Doe';
        
        $form = $this->createForm(ContactType::class, $data);
        $form->handleRequest($request);
        if ($form->isSubmitted() && $form->isValid()) {
            // Envoyer Email
        }
        return $this->render('contact/contact.html.twig', [
            'form' => $form,
        ]);
        
    }
}
```

On définit les types de champs.
`ContactType.php`
```php
namespace App\Form;

use App\DTO\ContactDTO;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\EmailType;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;
use Symfony\Component\Form\Extension\Core\Type\TextareaType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class ContactType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('name', TextType::class, [
                'empty_data' => ''
            ])
            ->add('email', EmailType::class, [
                'empty_data' => ''
            ])
            ->add('message', TextareaType::class, [
                'empty_data' => ''
            ])
            ->add('save', SubmitType::class, [
                'label' => 'Envoyer'
            ])
        ;
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => ContactDTO::class,
        ]);
    }
}
```

On crée un template de formulaire plus personnalisée
`contact.html.twig`
```php
{% extends 'base.html.twig' %}

{% block title %}Nous contacter{% endblock %}

{% block body %}
    <h1>Nous contacter</h1>
    {{ form_start(form) }}
    <div class="row">
        <div class="col-sm"> {{ form_row(form.name) }}</div>
        <div class="col-sm">{{ form_row(form.email) }}</div>
    </div>
    {{ form_end(form) }}
{% endblock %}
```

On met des contraintes au niveau des valeurs soumises.

Documentation :
- Contraints Email : https://symfony.com/doc/current/reference/constraints/Email.html

`ContactDTO.php`
```php

namespace App\DTO;

use Symfony\Component\Validator\Constraints as Assert;

class ContactDTO
{
    #[Assert\NotBlank]
    #[Assert\Length(min: 3, max: 200)]
    public string $name = '';

    #[Assert\NotBlank]
    #[Assert\Email]
    public string $email = '';

    #[Assert\NotBlank]
    #[Assert\Length(min: 10, max: 200)]
    public string $message = '';
}

```

#### 2eme étape : envoi d'email

Documentation :
- Sending Emails with Mailer : https://symfony.com/doc/current/mailer.html
- Creating & Sending Messages : https://symfony.com/doc/current/mailer.html#creating-sending-messages
- Mailer - Twig: HTML & CSS https://symfony.com/doc/current/mailer.html#twig-html-css
- Inky Email Templating : https://symfony.com/doc/current/mailer.html#inky-email-templating-language
- Inky : https://get.foundation/emails/docs/inky.html

En plus de l'object `Request` on a besoin de la `MailerInterface`
Mailer sert à envoyer un email mais pour représenter l'email on va devoir créer une instance de `Email`

`ContactController.php`
```php
namespace App\Controller;

use App\DTO\ContactDTO;
use App\Form\ContactType;
use Symfony\Bridge\Twig\Mime\TemplatedEmail;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Mailer\MailerInterface;
use Symfony\Component\Routing\Attribute\Route;

class ContactController extends AbstractController
{
    #[Route('/contact', name: 'contact')]
    public function contact(Request $request, MailerInterface $mailer): Response
    {
        $data = new ContactDTO();

        // TODO : Supprimer ça
        $data->name = 'John Doe';
        $data->email = 'john@doe.fr';
        $data->message = 'Message de John Doe';

        $form = $this->createForm(ContactType::class, $data);
        $form->handleRequest($request);
        if ($form->isSubmitted() && $form->isValid()) {
            $mail = (new TemplatedEmail())
                ->to('contact@example.com')
                ->from($data->email)
                ->subject('Demande de contact')
                ->htmlTemplate('emails/contact.html.twig')
                ->context(['data' => $data]);
            $mailer->send($mail);
            $this->addFlash(
                'success',
                'Votre email a bien été envoyé'
            );
            $this->redirectToRoute('contact');
        }
        return $this->render('contact/contact.html.twig', [
            'form' => $form,
        ]);
        
    }
}
```

On crée un gabarit pour l'email

`templates\emails\contact.html.twig`
```php
<p>
    Une nouvelle demande de contact a été reçue
</p>

<ul>
    <li>Nom : {{ data.name }}</li>
    <li>Email : {{ data.email }}</li>
</ul>

<p>
    <strong>Message</strong>:<br>
    {{ data.message | nl2br }}
</p>
```