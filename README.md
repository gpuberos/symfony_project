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
            return $this->redirectToRoute('contact');
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

#### 3eme étape : ajout d'un select service

Documentation :
- Form Type : https://symfony.com/doc/current/reference/forms/types.html
- ChoiceType : https://symfony.com/doc/current/reference/forms/types/choice.html

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

    #[Assert\NotBlank]
    public string $service = '';
}
```

On crée le select `service`

`ContactType.php`
```php
namespace App\Form;

use App\DTO\ContactDTO;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\ChoiceType;
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
            ->add('service', ChoiceType::class, [
                'choices'  => [
                    'Comptabilité' => 'compta@example.com',
                    'Support' => 'support@example.com',
                    'Marketing' => 'marketing@example.com',
                ],
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

On lui précise vers ou envoyer, pour ça on récupère le service sélectionné `->to($data->service)`

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
                ->to($data->service)
                ->from($data->email)
                ->subject('Demande de contact')
                ->htmlTemplate('emails/contact.html.twig')
                ->context(['data' => $data]);
            $mailer->send($mail);
            $this->addFlash(
                'success',
                'Votre email a bien été envoyé'
            );
            return $this->redirectToRoute('contact');
        }
        return $this->render('contact/contact.html.twig', [
            'form' => $form,
        ]);
        
    }
}
```

#### 4 étape : gestion d'erreur en cas d'échec de l'envoi

On utilisera un try catch

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

        $form = $this->createForm(ContactType::class, $data);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            try {
                $mail = (new TemplatedEmail())
                    // Pour tester remplacer $data->service par une email non valide 'emailnonvalide'
                    ->to($data->service)
                    ->from($data->email)
                    ->subject('Demande de contact')
                    ->htmlTemplate('emails/contact.html.twig')
                    ->context(['data' => $data]);

                $mailer->send($mail);
                $this->addFlash(
                    'success',
                    'Votre email a bien été envoyé'
                );

                return $this->redirectToRoute('contact');
            
            } catch (\Exception $e) {
                $this->addFlash(
                    'danger',
                    'Impossible d\'envoyer votre email'
                );
            }
        }

        return $this->render('contact/contact.html.twig', [
            'form' => $form,
        ]);
    }
}
```

#### En résumé :

1. On crée un objet qui représente mes données, ici ce n'est pas une entité parce qu'on ne communique pas avec la base de données.
2. On crée un formulaire grâce à la commande `php bin/console make:form ContactType` puis on le personnalise.
3. On crée un controleur grâce à la commande `php bin/console make:controller ContactController` et on y mets les méthodes classiques `createForm()`, le `handleRequest`, le `isSubmitted()` et `isValid()` et on fait notre traitement. On injectera le service `MailerInterface` pour avoir la capacité d'envoyer un email.

## Catégorie

On réorganise notre application.

On créer un dossier `Admin` dans le dossier `Controller`. On y met `RecipeController.php`

On modifie le namespace de `RecipeController.php` comme ceci `namespace App\Controller\Admin` et on ajoute une route à la class `RecipeController`

`src\Controller\Admin\RecipeController.php`
```php
namespace App\Controller\Admin;

#[Route('admin/recettes', name: 'admin.recipe')]
class RecipeController extends AbstractController
{
```

```shell
$ php bin/console debug:router
 --------------------------- ---------- -------- ------ --------------------------------------
  Name                        Method     Scheme   Host   Path
 --------------------------- ---------- -------- ------ --------------------------------------
  _preview_error              ANY        ANY      ANY    /_error/{code}.{_format}
  _wdt                        ANY        ANY      ANY    /_wdt/{token}
  _profiler_home              ANY        ANY      ANY    /_profiler/
  _profiler_search            ANY        ANY      ANY    /_profiler/search
  _profiler_search_bar        ANY        ANY      ANY    /_profiler/search_bar
  _profiler_phpinfo           ANY        ANY      ANY    /_profiler/phpinfo
  _profiler_xdebug            ANY        ANY      ANY    /_profiler/xdebug
  _profiler_font              ANY        ANY      ANY    /_profiler/font/{fontName}.woff2
  _profiler_search_results    ANY        ANY      ANY    /_profiler/{token}/search/results
  _profiler_open_file         ANY        ANY      ANY    /_profiler/open
  _profiler                   ANY        ANY      ANY    /_profiler/{token}
  _profiler_router            ANY        ANY      ANY    /_profiler/{token}/router
  _profiler_exception         ANY        ANY      ANY    /_profiler/{token}/exception
  _profiler_exception_css     ANY        ANY      ANY    /_profiler/{token}/exception.css
  admin.reciperecipe.index    ANY        ANY      ANY    /admin/recettes/recettes
  admin.reciperecipe.show     ANY        ANY      ANY    /admin/recettes/recettes/{slug}-{id}
  admin.reciperecipe.edit     GET|POST   ANY      ANY    /admin/recettes/recettes/{id}/edit
  admin.reciperecipe.create   ANY        ANY      ANY    /admin/recettes/recettes/create
  admin.reciperecipe.delete   DELETE     ANY      ANY    /admin/recettes/recettes/{id}
  contact                     ANY        ANY      ANY    /contact
  home                        ANY        ANY      ANY    /
 --------------------------- ---------- -------- ------ --------------------------------------
```

On peut voir que la route a bien été crée
```shell
  admin.reciperecipe.index    ANY        ANY      ANY    /admin/recettes/recettes
```

On ajoute des requirements pour l'id pour indiquer qu'on attend que des nombres. On pourra utiliser l'enumérateur `Requirement` qui contiendra des constantes par rapport aux pré-requis des URL.

```php
namespace Symfony\Component\Routing\Requirement;

/*
 * A collection of universal regular-expression constants to use as route parameter requirements.
 */
enum Requirement
{
    public const ASCII_SLUG = '[A-Za-z0-9]+(?:-[A-Za-z0-9]+)*'; // symfony/string AsciiSlugger default implementation
    public const CATCH_ALL = '.+';
    public const DATE_YMD = '[0-9]{4}-(?:0[1-9]|1[012])-(?:0[1-9]|[12][0-9]|(?<!02-)3[01])'; // YYYY-MM-DD
    public const DIGITS = '[0-9]+';
    public const POSITIVE_INT = '[1-9][0-9]*';
    public const UID_BASE32 = '[0-9A-HJKMNP-TV-Z]{26}';
    public const UID_BASE58 = '[1-9A-HJ-NP-Za-km-z]{22}';
    public const UID_RFC4122 = '[0-9a-f]{8}(?:-[0-9a-f]{4}){3}-[0-9a-f]{12}';
    public const ULID = '[0-7][0-9A-HJKMNP-TV-Z]{25}';
    public const UUID = '[0-9a-f]{8}-[0-9a-f]{4}-[13-8][0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}';
    public const UUID_V1 = '[0-9a-f]{8}-[0-9a-f]{4}-1[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}';
    public const UUID_V3 = '[0-9a-f]{8}-[0-9a-f]{4}-3[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}';
    public const UUID_V4 = '[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}';
    public const UUID_V5 = '[0-9a-f]{8}-[0-9a-f]{4}-5[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}';
    public const UUID_V6 = '[0-9a-f]{8}-[0-9a-f]{4}-6[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}';
    public const UUID_V7 = '[0-9a-f]{8}-[0-9a-f]{4}-7[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}';
    public const UUID_V8 = '[0-9a-f]{8}-[0-9a-f]{4}-8[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}';
}
```

On choisira `requirements: ['id' => Requirement::DIGITS]`

On supprimera la route et la méthode pour afficher une recette car on en aura pas besoin dans l'admin.

Modifier :
- les Routes
- les chemins des gabarits Twig que l'on mettre dans un dossier `admin`
- les `redirectToRoute`

```php
namespace App\Controller\Admin;

use App\Entity\Recipe;
use App\Form\RecipeType;
use App\Repository\RecipeRepository;
use Doctrine\ORM\EntityManagerInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Attribute\Route;
use Symfony\Component\Routing\Requirement\Requirement;

#[Route('/admin/recettes', name: 'admin.recipe.')]
class RecipeController extends AbstractController
{
    public function __construct(private RecipeRepository $repository)
    {
    }

    #[Route('/', name: 'index')]
    public function index(): Response
    {
        $recipes = $this->repository->findWithDurationLowerThan(20);

        return $this->render('recipe/index.html.twig', [
            'recipes' => $recipes
        ]);
    }

    #[Route('/create', name: 'create')]
    public function create(Request $request, EntityManagerInterface $em)
    {
        $recipe = new Recipe();
        $form = $this->createForm(RecipeType::class, $recipe);
        $form->handleRequest($request);
        if ($form->isSubmitted() && $form->isValid()) {
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

    #[Route('/{id}', name: 'edit', methods: ['GET', 'POST'], requirements: ['id' => Requirement::DIGITS])]
    public function edit(Recipe $recipe, Request $request, EntityManagerInterface $em)
    {
        $form = $this->createForm(RecipeType::class, $recipe);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $em->flush();
            $this->addFlash(
                'success',
                'La recette a bien été modifiée'
            );
            return $this->redirectToRoute('recipe.index');
        }

        return $this->render('recipe/edit.html.twig', [
            'recipe' => $recipe,
            'formTitle' => 'Editer : ' . $recipe->getTitle(),
            'form' => $form
        ]);
    }

    #[Route('/{id}', name: 'delete', methods: ['DELETE'], requirements: ['id' => Requirement::DIGITS])]
    public function delete(Recipe $recipe, EntityManagerInterface $em)
    {
        $em->remove($recipe);
        $em->flush();
        $this->addFlash('success', 'La recette a bien été supprimée');
        return $this->redirectToRoute('recipe.index');
    }
}


```

Et on revifiera avec le `debug:router`
```shell
$ php bin/console debug:router
 -------------------------- ---------- -------- ------ -----------------------------------
  Name                       Method     Scheme   Host   Path
 -------------------------- ---------- -------- ------ -----------------------------------
  _preview_error             ANY        ANY      ANY    /_error/{code}.{_format}
  _wdt                       ANY        ANY      ANY    /_wdt/{token}
  _profiler_home             ANY        ANY      ANY    /_profiler/
  _profiler_search           ANY        ANY      ANY    /_profiler/search
  _profiler_search_bar       ANY        ANY      ANY    /_profiler/search_bar
  _profiler_phpinfo          ANY        ANY      ANY    /_profiler/phpinfo
  _profiler_xdebug           ANY        ANY      ANY    /_profiler/xdebug
  _profiler_font             ANY        ANY      ANY    /_profiler/font/{fontName}.woff2
  _profiler_search_results   ANY        ANY      ANY    /_profiler/{token}/search/results
  _profiler_open_file        ANY        ANY      ANY    /_profiler/open
  _profiler                  ANY        ANY      ANY    /_profiler/{token}
  _profiler_router           ANY        ANY      ANY    /_profiler/{token}/router
  _profiler_exception        ANY        ANY      ANY    /_profiler/{token}/exception
  _profiler_exception_css    ANY        ANY      ANY    /_profiler/{token}/exception.css
  admin.recipeindex          ANY        ANY      ANY    /admin/recettes/
  admin.recipecreate         ANY        ANY      ANY    /admin/recettes/create
  admin.recipeedit           GET|POST   ANY      ANY    /admin/recettes/{id}
  admin.recipedelete         DELETE     ANY      ANY    /admin/recettes/{id}
  contact                    ANY        ANY      ANY    /contact
  home                       ANY        ANY      ANY    /
 -------------------------- ---------- -------- ------ -----------------------------------
```

Création Admin\CategoryController.php
```php
namespace App\Controller\Admin;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Attribute\Route;
use Symfony\Component\Routing\Requirement\Requirement;

#[Route('/admin/category', name: 'admin.category.')]
class CategoryController extends AbstractController
{
    #[Route('/', name: 'index')]
    public function index()
    {
        
    }

    #[Route('/create', name: 'create')]
    public function create()
    {
        
    }

    #[Route('/{id}', name: 'edit', methods: ['GET', 'POST'], requirements: ['id' => Requirement::DIGITS])]
    public function edit()
    {
        
    }

    #[Route('/{id}', name: 'delete', methods: ['DELETE'], requirements: ['id' => Requirement::DIGITS])]
    public function remove()
    {

    }
}

```


Création de l'entité :

```shell
$ bin/console make:entity Category
 created: src/Entity/Category.php
 created: src/Repository/CategoryRepository.php
 
 Entity generated! Now let's add some fields!
 You can always add more fields later manually or by re-running this command.

 New property name (press <return> to stop adding fields):
 > name

 Field type (enter ? to see all types) [string]:
 >


 Field length [255]:
 >

 Can this field be null in the database (nullable) (yes/no) [no]:
 >

 updated: src/Entity/Category.php

 Add another property? Enter the property name (or press <return> to stop adding fields):
 > slug

 Field type (enter ? to see all types) [string]:
 >


 Field length [255]:
 >

 Can this field be null in the database (nullable) (yes/no) [no]:
 >

 updated: src/Entity/Category.php

 Add another property? Enter the property name (or press <return> to stop adding fields):
 > createdAt

 Field type (enter ? to see all types) [datetime_immutable]:
 >


 Can this field be null in the database (nullable) (yes/no) [no]:
 >

 updated: src/Entity/Category.php

 Add another property? Enter the property name (or press <return> to stop adding fields):
 > updatedAt

 Field type (enter ? to see all types) [datetime_immutable]:
 >


 Can this field be null in the database (nullable) (yes/no) [no]:
 >

 updated: src/Entity/Category.php

 Add another property? Enter the property name (or press <return> to stop adding fields):
 >


 
  Success! 
 

 Next: When you're ready, create a migration with php bin/console make:migration
```

```shell
php bin/console make:migration
```

```shell
php bin/console doctrine:migrations:migrate
```

On génère le formulaire :

```shell
$ php bin/console make:form CategoryType

 The name of Entity or fully qualified model class name that the new form will be bound to (empty for none):
 > Category
Category

 created: src/Form/CategoryType.php

 
  Success! 
 

 Next: Add fields to your form and start using it.
 Find the documentation at https://symfony.com/doc/current/forms.html
```

### Service `FormListenerFactory`

Fonction créer à la volée on n'a pas accès aux variables qui proviennent de l'extérieur il faut utiliser `use ($field)` pour cela

`FormListenerFactory.php`
```php
namespace App\Form;

use Symfony\Component\Form\Event\PostSubmitEvent;
use Symfony\Component\Form\Event\PreSubmitEvent;
use Symfony\Component\String\Slugger\AsciiSlugger;
use Symfony\Component\String\Slugger\SluggerInterface;

class FormListenerFactory
{
    public function __construct(private SluggerInterface $slugger)
    {

    }

    public function autoSlug(string $field): callable
    {
        return function (PreSubmitEvent $event) use ($field) {
            $data = $event->getData();
            if (empty($data['slug'])) {
                $data['slug'] = strtolower($this->slugger->slug($data[$field]));
            }

            $event->setData($data);
        };
    }

    public function timestamps(): callable
    {

        return function (PostSubmitEvent $event) {

            $data = $event->getData();

            $data->setUpdatedAt(new \DateTimeImmutable());

            if (!$data->getId()) {
                $data->setCreatedAt(new \DateTimeImmutable());
            }
        };
    }
}
```

## ORM : relation ManyToOne

### Création de la relation ManyToOne

C'est sur les recettes qu'on souhaite rajouter les catégories

```shell
$ php bin/console make:entity Recipe
 Your entity already exists! So let's add some new fields!
```

Nom de la propriété : `category`
```shell
 New property name (press <return> to stop adding fields):
 > category
 ```

Au niveau du type on lui indique que ça va être une `relation`
```shell
 Field type (enter ? to see all types) [string]:
 > relation
relation
```

Il nous demande alors à quel entité va être lié notre entité `Recipe`. On lui indique que c'est lié aux catégories `Category`
```shell
 What class should this entity be related to?:
 > Category
Category
```

Il nous propose les différents type de relation
```shell
What type of relationship is this?
 ------------ -----------------------------------------------------------------------
  Type         Description
 ------------ -----------------------------------------------------------------------
  ManyToOne    Each Recipe relates to (has) one Category.
               Each Category can relate to (can have) many Recipe objects.

  OneToMany    Each Recipe can relate to (can have) many Category objects.
               Each Category relates to (has) one Recipe.

  ManyToMany   Each Recipe can relate to (can have) many Category objects.
               Each Category can also relate to (can also have) many Recipe objects.

  OneToOne     Each Recipe relates to (has) exactly one Category.
               Each Category also relates to (has) exactly one Recipe.
 ------------ -----------------------------------------------------------------------
 ```

ManyToOne : chaque recette a une catégorie par contre chaque catégorie peut avoir plusieurs recettes.

```shell
 Relation type? [ManyToOne, OneToMany, ManyToMany, OneToOne]:
 > ManyToOne
ManyToOne
```

Est ce qu'on autorise les recettes à avoir des catégories null (on dirait non si on avait pas déjà rempli les catégories). Donc on peut avoir des recettes sans catégorie.
```shell
 Is the Recipe.category property allowed to be null (nullable)? (yes/no) [yes]:
 > yes
 ```

Est ce que l'on veut ajouter une nouvelle propriété sur catégorie ? C'est à dire rajouter la relation inverse. Depuis une catégorie on aura la possibilité de récupérer les recettes. 
```shell
 Do you want to add a new property to Category so that you can access/update Recipe objects from it - e.g. $category->getRecipes()? (yes/no) [
yes]:
 > yes
 A new property will also be added to the Category class so that you can access the related Recipe objects from it.
```

Il nous demande de choisir le nom de ce champ dans l'entité catégorie
```shell
 New field name inside Category [recipes]:
 >

 updated: src/Entity/Recipe.php
 updated: src/Entity/Category.php

 Add another property? Enter the property name (or press <return> to stop adding fields):
 >


 
  Success! 
 

 Next: When you're ready, create a migration with php bin/console make:migration
```

### Modification apporté à l'entité Recipe

Il a ajouter à l'entité `Entity\Recipe.php`

```php
    #[ORM\ManyToOne(inversedBy: 'recipes')]
    private ?Category $category = null;
```

Il a d'abord ajouté une nouvelle propriété privée `Category` qui peut être null et il nous a rajouté au dessus un attribut pour expliquer à l'ORL que c'est une relation `ManyToOne` qui est inversée par `recipes`.

Et il nous a crée le getter et le setter
```php
    public function getCategory(): ?Category
    {
        return $this->category;
    }

    public function setCategory(?Category $category): static
    {
        $this->category = $category;

        return $this;
    }
```

### Modification apporté à l'entité Category

Au niveau de catégorie `Category.php` là ou il a fait la relation inverse.

```php
    #[ORM\OneToMany(targetEntity: Recipe::class, mappedBy: 'category')]
    private Collection $recipes;
```
Une propriété privée, le type est de type `Collection`, c'est un objet qui provient de doctrine (une collection est une sorte de tableau amélioré sur lesquels on aura des méthodes intéressantes, mais ça peut être utilisé comme un tableau).

Au dessus il y a notre attribut

Dès la construction de l'objet il va initialiser cette propriété `recipes` comme une collection vide `ArrayCollection()`. Il fait ça pour qu'on ait toujours quelque chose de type collection à l'intérieur de la variable `$recipes`
```php
    public function __construct()
    {
        $this->recipes = new ArrayCollection();
    }
```

Il a crée plusieurs méthodes :
- `getRecipes()` : permet de récupérer l'ensemble des recettes.
- `addRecipe()` : permet d'ajouter une recette. Cette méthode va prendre notre Collection et utiliser la méthode `add()` pour ajouter notre recette, mais en même temps elle va en profiter pour modifier la recette et l'attacher à la catégorie associée.
- `removeRecipe()` : permet de supprimer une recette, ça prend la recette et ça lui dit au fait la catégorie est null. 

ça s'assure que l'objet recette que l'on attache à cette catégorie est bien initialisé convenablement.


```php
    /**
     * @return Collection<int, Recipe>
     */
    public function getRecipes(): Collection
    {
        return $this->recipes;
    }

    public function addRecipe(Recipe $recipe): static
    {
        if (!$this->recipes->contains($recipe)) {
            $this->recipes->add($recipe);
            $recipe->setCategory($this);
        }

        return $this;
    }

    public function removeRecipe(Recipe $recipe): static
    {
        if ($this->recipes->removeElement($recipe)) {
            // set the owning side to null (unless already changed)
            if ($recipe->getCategory() === $this) {
                $recipe->setCategory(null);
            }
        }

        return $this;
    }
```

#### Test
On teste `http://localhost:8000/admin/recettes/`

Si on souhaite attacher `'pates-bolognaises'` à la catégorie `'plat-principal'` on le fait avec les setters. C'est à dire qu'on va prendre `$pates` et qu'on lui assigne la catégorie en faisant `setCategory($platPrincipal)`.
Et une fois qu'on est satisfait on utilise l'`EntityManagerInterface` pour sauvegarder.


`RecipeController.php`
```php
    #[Route('/', name: 'index')]
    public function index(CategoryRepository $categoryRepository, EntityManagerInterface $em): Response
    {
        $platPrincipal = $categoryRepository->findOneBy(['slug' => 'plat-principal']);
        $pates = $this->repository->findOneBy(['slug' => 'pates-bolognaise']);
        $pates->setCategory($platPrincipal);
        $em->flush();
        dd($platPrincipal);
        
        $recipes = $this->repository->findWithDurationLowerThan(20);

        return $this->render('admin/recipe/index.html.twig', [
            'recipes' => $recipes
        ]);
    }

```

Si l'on souhaite récupérer la catégorie dans la fonction index on mettra :
```php
dd($recipes[0]->getCategory());
```

```
RecipeController.php on line 29:
Proxies\__CG__\App\Entity\Category {#938 ▼
  -id: 2
  -name: ? string
  -slug: ? string
  -createdAt: ? ?DateTimeImmutable
  -updatedAt: ? ?DateTimeImmutable
  -recipes: ? Doctrine\Common\Collections\Collection
  -lazyObjectState: 
Symfony\Component\VarExporter\Internal
\
LazyObjectState {#939 ▶}
}
```

On remarque un détail intéressant, il ne nous donne pas un objet de type Category et il ne nous donnera pas forcément le même objet si on fait ça (peux tu me donner le nom `getName()`. 

```php
$recipes[1]->getCategory()->getName();
dd($recipes[1]->getCategory());
```

```
RecipeController.php on line 30:
Proxies\__CG__\App\Entity\Category {#938 ▼
  -id: 2
  -name: "Plat principal"
  -slug: "plat-principal"
  -createdAt: DateTimeImmutable @1710701512 {#951 ▶}
  -updatedAt: DateTimeImmutable @1710701512 {#862 ▶}
  -recipes: 
Doctrine\ORM
\
PersistentCollection {#764 ▶}
  -lazyObjectState: 
Symfony\Component\VarExporter\Internal
\
LazyObjectState {#939 ▶}
}
```

Il nous donne un objet différent. C'est quelque chose que Symfony gère pour éviter de faire trop de requête SQL, par défaut lorsqu'on récupère la catégorie il ne la remplit pas, mais par contre dès qu'on va essayer d'accéder à une propriété il se dira que là il a besoin d'une propriété donc je suis obligé de faire une requête supplémentaire.

> [!IMPORTANT] 
> C'est ce qu'on appelle le problème n+1 c'est à dire que pour afficher n enregistrement on peut potentiellement avoir n + une requête. 

**Pour éviter ce problème on a plusieurs solutions :**

On pourra directement récupérer toutes les données dont on a besoin au niveau du query Builder.

Il va falloir faire la liaison pour récupérer les catégories. Pour faire une liaison en SQL on utiliserait un LEFT JOIN sauf qu'on a pas à mettre toutes les conditions automatiquement Symfony sait les liaisons qu'il existe. Donc si on fait `->leftJoin('r.category', 'c')`. Ensuite dans le SELECT `->select('r', 'c')` on lui dit qu'on souhaite récupérer les informations concernant les recettes et les catégories.

Si on réactualise la page on voit qu'on tombe à une seule requête au lieu de 4.

`Repository\RecipePrepository.php`
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
                    ->setMaxResults(20)
                    ->setParameter('duration', $duration)
                    ->getQuery()
                    ->getResult();
    }
```

C'est aussi ce LEFT JOIN qui peut permettre de filter les choses. 
Exemple : Je ne veux que les recettes qui sont des desserts et j'ai le Slug dessert. Vu que maintenant on a fait la liaison, on peut faire une condition supplémentaire, je veux que le Slug de ma catégorie `'c.slug'` soit égal à dessert `->andWhere('c.slug = \'dessert\'')`. Dans ce cas là il me montrerait que les recettes qui concerne les desserts.

> [!NOTE]
> En SQL c'est des simples quotes qu'il faudra échapper ex: `andWhere('c.slug = \'dessert\'')`

`Repository\RecipePrepository.php`
```php
    public function findWithDurationLowerThan(int $duration): array
    {
        return $this->createQueryBuilder('r')
                    ->select('r', 'c')
                    ->where('r.duration <= :duration')
                    ->orderBy('r.duration', 'ASC')
                    ->leftJoin('r.category', 'c')
                    ->andWhere('c.slug = \'dessert\'')
                    ->setMaxResults(20)
                    ->setParameter('duration', $duration)
                    ->getQuery()
                    ->getResult();
    }
```

Si on a simplement l'id on pourrait dire ici je veux que `c.id` soit égal à l'id de dessert.

```php
->andWhere('c.id = 1')
```

L'inconvénient c'est qu'on fait une relation qui ne sert pas vraiment, on fait un LEFT JOIN alors qu'on utilise que l'id. On n'a malheureusement pas accès aux champs `r.category_id`, il nous dira que ce champ n'existe pas.

`Repository\RecipePrepository.php`
```php
    public function findWithDurationLowerThan(int $duration): array
    {
        return $this->createQueryBuilder('r')
                    ->where('r.duration <= :duration')
                    ->orderBy('r.duration', 'ASC')
                    ->andWhere('r.category_id = 1')
                    ->setMaxResults(20)
                    ->setParameter('duration', $duration)
                    ->getQuery()
                    ->getResult();
    }
```
On aura une erreur. Pourtant on saite que category_id existe mais l'ORM ne parle qu'en terme de champ qui existe dans notre entité. Donc ça pourra sembler valable en SQL mais pas au niveau de Doctrine.

Donc si on souhaite faire une sélection via l'id on mettre direment
```php
->andWhere('r.category = 1')
```

### Modifier la recette associée à un plat

Documentation :
- Choice Fields : https://symfony.com/doc/current/reference/forms/types.html#choice-fields
- EntityType Field : https://symfony.com/doc/current/reference/forms/types/entity.html

Le champ EntityType va nous permettre de spécifier une entité qui va être associé.

Ajouter un champ category de type `EntityType` et on précisera différents arguments, la class (entité associé) c'est `Category` et le champ qui sera utilisé pour les options sera `name`
```php
->add('category', EntityType::class, [
    'class' => Category::class,
    'choice_label' => 'name'
])
```

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
            ->add('category', EntityType::class, [
                'class' => Category::class,
                'choice_label' => 'name'
            ])
            ->add('content', TextareaType::class, [
                'empty_data' => ''
            ])
            ->add('duration')
            ->add('save', SubmitType::class, [
                'label' => 'Envoyer'
            ])
            ->addEventListener(FormEvents::PRE_SUBMIT, $this->formListenerFactory->autoSlug('title'))
            ->addEventListener(FormEvents::POST_SUBMIT, $this->formListenerFactory->timestamps());
    }
```

On précisera que le choix pourra être multiple `'multiple' => true`

`Form\CategoryType.php`
```php
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('name', TextType::class, [
                'empty_data' => ''
            ])
            ->add('slug', TextType::class, [
                'required' => false,
                'empty_data' => ''
            ])
            ->add('recipes', EntityType::class, [
                'class' => Recipe::class,
                'choice_label' => 'title'
                'multiple' => true
            ])
            ->add('save', SubmitType::class, [
                'label' => 'Enregistrer'
            ])
            ->addEventListener(FormEvents::PRE_SUBMIT, $this->formListenerFactory->autoSlug('name'))
            ->addEventListener(FormEvents::POST_SUBMIT, $this->formListenerFactory->timestamps());
    }
```

ça ne fonctionnera pas car il a modifié les propriétés à la volée ce qui fait que si on regarde la recette Recette de démo il n'a pas modifié la catégorie. 
```
CategoryController.php on line 49:
App\Entity\Category {#655 ▼
  -id: 1
  -name: "Dessert"
  -slug: "dessert"
  -createdAt: DateTimeImmutable @1710701330 {#648 ▶}
  -updatedAt: DateTimeImmutable @1710709832 {#1071 ▶}
  -recipes: 
Doctrine\ORM
\
PersistentCollection {#679 ▼
    #collection: 
Doctrine\Common\Collections
\
ArrayCollection {#663 ▼
      -elements: array:2 [▼
        0 => 
App\Entity
\
Recipe {#947 ▼
          -id: 3
          -title: "Barbe à papa"
          -slug: "barbe-papa"
          -content: """
            
Il est indispensable pour faire de la barbe à papa de disposer d'une machine spéciale, qui peut se louer a la journée chez les spécialistes de locations de maté
 ▶

            

            Mettez du sucre raffiné.
            """
          -createdAt: DateTimeImmutable @1710421666 {#941 ▶}
          -updatedAt: DateTimeImmutable @1710421666 {#938 ▶}
          -duration: 5
          -category: 
App\Entity
\
Category {#655}
        }
        1 => 
App\Entity
\
Recipe {#1050 ▼
          -id: 6
          -title: "Recette de démo"
          -slug: "recette-de-demo"
          -content: "auto slugger test"
          -createdAt: DateTimeImmutable @1710530789 {#1058 ▶}
          -updatedAt: DateTimeImmutable @1710530789 {#1062 ▶}
          -duration: 15
          -category: null
        }
      ]
    }
    #initialized: true
    -snapshot: array:1 [ …1]
    -owner: 
App\Entity
\
Category {#655}
    -association: 
Doctrine\ORM\Mapping
\
OneToManyAssociationMapping {#599 …}
    -backRefFieldName: "category"
    -isDirty: true
    -em: 
ContainerEvR7nLh
\
EntityManagerGhostEbeb667 {#230 …12}
    -typeClass: 
Doctrine\ORM\Mapping
\
ClassMetadata {#657 …}
  }
}
```

> [!IMPORTANT]
> Il faudra mettre un autre attribut lorsqu'on fait ce type de champ. (Par défaut il modifie la collection directement).
> Dans notre objet Entity\Category.php ce sont les méthodes add et remove qui sont appelées parce que ce sont ces méthodes là qui viennent modifier l'objet lorsqu'il est attaché ou détaché et qui vient mettre le setCategory convenablement. Par défaut le formulaire utilise plutôt les setters, il est donc important de faire ce **by_reference** à **false** lorsqu'on a des relations de type **ManyToMany** ou **OneToMany** à modifier.

Documentation :
- by_reference : https://symfony.com/doc/current/reference/forms/types/choice.html#by-reference

Par défaut le `by_reference` est à `true` et s'attend à avoir une méthode `setAuthor()`. Si on met le `by_reference` à `false` dans ce cas là il utilisera plutôt les méthodes `add()` et `remove()`. Dans notre cas c'est la méthode `add()`

`Form\CategoryType.php`
```php
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('name', TextType::class, [
                'empty_data' => ''
            ])
            ->add('slug', TextType::class, [
                'required' => false,
                'empty_data' => ''
            ])
            ->add('recipes', EntityType::class, [
                'class' => Recipe::class,
                'choice_label' => 'title',
                'multiple' => true,
                'by_reference' => false
            ])
            ->add('save', SubmitType::class, [
                'label' => 'Enregistrer'
            ])
            ->addEventListener(FormEvents::PRE_SUBMIT, $this->formListenerFactory->autoSlug('name'))
            ->addEventListener(FormEvents::POST_SUBMIT, $this->formListenerFactory->timestamps());
    }
```

`Entity\Category.php`
```php
    public function addRecipe(Recipe $recipe): static
    {
        if (!$this->recipes->contains($recipe)) {
            $this->recipes->add($recipe);
            $recipe->setCategory($this);
        }

        return $this;
    }

    public function removeRecipe(Recipe $recipe): static
    {
        if ($this->recipes->removeElement($recipe)) {
            // set the owning side to null (unless already changed)
            if ($recipe->getCategory() === $this) {
                $recipe->setCategory(null);
            }
        }

        return $this;
    }
```

On aura la possibilité de changer le format de ce champ `expanded` pour faire en sorte que ce soit des checkbox

Documentation :
- expanded : https://symfony.com/doc/current/reference/forms/types/choice.html#expanded

On retira le champ recipes car il ne sera pas adapté à notre projet de milliers de recettes.

`Form\CategoryType.php`
```php
            ->add('recipes', EntityType::class, [
                'class' => Recipe::class,
                'choice_label' => 'title',
                'multiple' => true,
                'expanded' => true,
                'by_reference' => false
            ])
```

### Persistance en cascade

On va prendre notre première recette et on va créer à la volée une nouvelle catégorie.

`RecipeController.php`
```php
    #[Route('/', name: 'index')]
    public function index(CategoryRepository $categoryRepository, EntityManagerInterface $em): Response
    {

        $recipes = $this->repository->findWithDurationLowerThan(20);
        $category = (new Category())
            ->setCreatedAt(new \DateTimeImmutable())
            ->setUpdatedAt(new \DateTimeImmutable())
            ->setName('demo')
            ->setName('demo');
        $em->persist($category);
        $recipes[0]->setCategory($category);
        $em->flush();
        return $this->render('admin/recipe/index.html.twig', [
            'recipes' => $recipes
        ]);
    }
```
Sans le persist($category) on aurait une erreur. Il nous dira qu'il a trouvé une nouvelle entité à travers la relation mais qu'elle n'a pas été persistée. Pour éviter cette erreur on pourrait mettre `$em->persist($category);` directement dans le contrôleur.

L'autre solution c'est de rajouter une propriété au niveau des relations. Donc dans notre modèle Recipe on lui dira je veux qu'en cascade tu fasses de la persistance `cascade: ['persist']`. Si il reçoit un nouvel objet et que c'est objet n'est pas persisté il le persistera de manière automatique.

`Entity\Recipe.php`
```php
    #[ORM\ManyToOne(inversedBy: 'recipes', cascade: ['persist'])]
    private ?Category $category = null;
```

Si maintenant on supprime une catégorie on tombera sur une erreur. Il nous dira qu'il ne pas la supprimer car il y a une contrainte de clé étrangère.

Donc au niveau de notre class Category lui dire que lorsqu'on a une suppression de catégorie on veut qu'en cascade il aille supprimer les recettes associées. Dans ce cas là quand on supprimera une catégorie il fera aussi un remove au niveau de l'Entity Manager ce qui fait que lorsqu'on fera un flush il supprimera à la fois les recettes et les catégories associées.

`Entity\Category.php`
```php
    #[ORM\OneToMany(targetEntity: Recipe::class, mappedBy: 'category', cascade: ['remove'])]
    private Collection $recipes;
```

### Aller plus loin (Relations)

**Orphan Removal** : permet de supprimer automatiquement un élément lorsqu'il devient orphelin.

**Documentations :**
- Transitive persistence / Cascade Operations : https://www.doctrine-project.org/projects/doctrine-orm/en/3.1/reference/working-with-associations.html#transitive-persistence-cascade-operations
- Orphan Removal : https://www.doctrine-project.org/projects/doctrine-orm/en/3.1/reference/working-with-associations.html#orphan-removal

### En résumé :

Si on veut avoir une relation entre des données de modifier l'entité et d'utiliser le type relation ensuite Symfony nous guide pour définir cette relation. Il va automatiquement créer les méthodes qui vont nous permettre d'associer des données ensemble et ensuite on pourra manipuler des objets pour sauvegarder et modifier les relations qu'il y a entre les objets. Ensuite lors de la récupération des informations soit on fait une récupération classique et dans ce cas là il génère automatiquement des requêtes SQL quand il en a besoin mais si on a besoin de faire des requêtes plus compliquées, il faudra faire les choses en SQL, on utilisera les systèmes de LEFT JOIN et on pourra automatiquement récupérer les résultats qui nous intéressent (il est important de comprendre les relations qui sont faites).

On a vu également comment modifier les formulaires pour faire en sorte que ces relations soient sauvegardées automatiquement et on a vu le système de cascade et de orphan removal qui nous permet de gérer la suppression ou la création en cascade.

## Envoi de fichiers

### Création de la propriété Thumbnail dans l'entité Recipe

Création d'une propriété thumbnail dans l'entité Recipe
```shell
$ php bin/console make:entity Recipe
 Your entity already exists! So let's add some new fields!

 New property name (press <return> to stop adding fields):
 > thumbnail

 Field type (enter ? to see all types) [string]:
 >


 Field length [255]:
 >

 Can this field be null in the database (nullable) (yes/no) [no]:
 > yes

 updated: src/Entity/Recipe.php

 Add another property? Enter the property name (or press <return> to stop adding fields):
 >


 
  Success! 
 

 Next: When you're ready, create a migration with php bin/console make:migration
```

### Ajout du champ file upload dans notre formulaire :

On ajoute le champ file upload `thumbnailFile`

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
            ->add('thumbnailFile', FileType::class, [
                'mapped' => false,
                'constraints' => [
                    new Image()
                ]
            ])
            ->add('category', EntityType::class, [
                'class' => Category::class,
                'choice_label' => 'name'
            ])
            ->add('content', TextareaType::class, [
                'empty_data' => ''
            ])
            ->add('duration')
            ->add('save', SubmitType::class, [
                'label' => 'Envoyer'
            ])
            ->addEventListener(FormEvents::PRE_SUBMIT, $this->formListenerFactory->autoSlug('title'))
            ->addEventListener(FormEvents::POST_SUBMIT, $this->formListenerFactory->timestamps());
    }
```

On ajoute 
```php
            ->add('thumbnailFile', FileType::class, [
                'mapped' => false,
                'constraints' => [
                    new Image()
                ]
            ])
```
On a une erreur qui indique qu'il ne trouve pas de propriété `thumbnailFile` dans l'entité `Recipe`. On va lui indiquer que c'est un champ qui n'existe pas vraiment en mettant la propriété `mapped`  sur `false`, ça veut dire que ce champ là ne sera pas mappé dans les données qui sont interne à notre formulaire, donc dans ce cas là il ne va pas utiliser les setters pour essayer de sauvegarder cette valeur là et il n'utilisera pas les getters pour la récupérer, c'est pour indiquer que c'est un champ qui vie de sa propre manière.

Documentation : 
- File Constraints : https://symfony.com/doc/current/reference/constraints.html#file-constraints
- Image : https://symfony.com/doc/current/reference/constraints/Image.html


### Récupération des informations du fichier au niveau du contrôleur :

On va récupérer les informations au niveau de notre contrôleur :
Pour tester on va mettre un `dd($form->get('thumbnailFile')->getData());`

```php
    #[Route('/{id}', name: 'edit', methods: ['GET', 'POST'], requirements: ['id' => Requirement::DIGITS])]
    public function edit(Recipe $recipe, Request $request, EntityManagerInterface $em)
    {
        $form = $this->createForm(RecipeType::class, $recipe);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            dd($form->get('thumbnailFile')->getData());
            $em->flush();
            $this->addFlash(
                'success',
                'La recette a bien été modifiée'
            );
            return $this->redirectToRoute('admin.recipe.index');
        }

        return $this->render('admin/recipe/edit.html.twig', [
            'recipe' => $recipe,
            'formTitle' => 'Editer : ' . $recipe->getTitle(),
            'form' => $form
        ]);
    }
```
On voit que dans `getData()` on obtient une instance de `UploadedFile`. Cette classe là représente un fichier qui a été uploadé et va contenir une méthode `move()` qui va permettre de déplacer le fichier.

```
RecipeController.php on line 57:
Symfony\Component\HttpFoundation\File\UploadedFile {#19 ▼
  -test: false
  -originalName: "barbe-a-papa.jpg"
  -mimeType: "image/jpeg"
  -error: 0
  path: "C:\wamp64\tmp"
  filename: "php358.tmp"
  basename: "php358.tmp"
  pathname: "C:\wamp64\tmp\php358.tmp"
  extension: "tmp"
  realPath: "
C:\wamp64
\
tmp\php358.tmp"
  aTime: 2024-03-18 08:26:52
  mTime: 2024-03-18 08:26:52
  cTime: 2024-03-18 08:26:52
  inode: 20547673300282431
  size: 15365
  perms: 0100666
  owner: 0
  group: 0
  type: "file"
  writable: true
  readable: true
  executable: false
  file: true
  dir: false
  link: false
  linkTarget: "C:\wamp64\tmp\php358.tmp"
}
```

```php
    #[Route('/{id}', name: 'edit', methods: ['GET', 'POST'], requirements: ['id' => Requirement::DIGITS])]
    public function edit(Recipe $recipe, Request $request, EntityManagerInterface $em)
    {
        $form = $this->createForm(RecipeType::class, $recipe);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            /** @var UploadedFile $file */
            $file = $form->get('thumbnailFile')->getData();
            dd($file->getClientOriginalName(), $file->getClientOriginalExtension());
            $em->flush();
            $this->addFlash(
                'success',
                'La recette a bien été modifiée'
            );
            return $this->redirectToRoute('admin.recipe.index');
        }

        return $this->render('admin/recipe/edit.html.twig', [
            'recipe' => $recipe,
            'formTitle' => 'Editer : ' . $recipe->getTitle(),
            'form' => $form
        ]);
    }
```

- `getClientOriginalName()` : si on veut récupérer le nom original du fichier envoyer.
- `getClientOriginalExtension()` : si on veut récupérer l'extension du fichier.

Pour voir les valeurs associées
```php
dd($file->getClientOriginalName(), $file->getClientOriginalExtension());
```

```
1 in RecipeController.php on line 60:
"barbe-a-papa.jpg"

2 in RecipeController.php on line 60:
"jpg"
```
### Déplacement du fichier dans un répertoire

On va déplacer le fichier dans un dossier image, on aura un sous dossier recette et on utilisera l'ID de la recette pour nommer le fichier.

Sur les variables de type UploadedFile on a une méthode `move()` et qui va permettre de déplacer un élément, ça prendra en premier paramètre un dossier et en second paramètre on peut mettre un nom si on choisit de renommer le fichier, si on ne met rien ça utilisera le nom de fichier original.

`RecipeController.php`
```php
    #[Route('/{id}', name: 'edit', methods: ['GET', 'POST'], requirements: ['id' => Requirement::DIGITS])]
    public function edit(Recipe $recipe, Request $request, EntityManagerInterface $em)
    {
        $form = $this->createForm(RecipeType::class, $recipe);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            /** @var UploadedFile $file */
            $file = $form->get('thumbnailFile')->getData();
            $filename = $recipe->getId() . '.' . $file->getClientOriginalExtension();
            $file->move($this->getParameter(), $filename);
            $recipe->setThumbnail($filename);
            dd($file->getClientOriginalName(), $file->getClientOriginalExtension());
            // $em->flush();
            $this->addFlash(
                'success',
                'La recette a bien été modifiée'
            );
            return $this->redirectToRoute('admin.recipe.index');
        }

        return $this->render('admin/recipe/edit.html.twig', [
            'recipe' => $recipe,
            'formTitle' => 'Editer : ' . $recipe->getTitle(),
            'form' => $form
        ]);
    }
```

Pour obtenir le chemin du dossier public on va pouvoir accéder à des paramètres qui concernent le framework et ces paramètres sont accessibles depuis le controleur grâce à la méthode getParameter().

> [!TIPS]
> Si on ne connait pas le nom d'un paramètre et qu'on a besoin de débugguer on utilisera dans le terminal la commande :
> ```shell
> php bin/console debug:container --parameters
> ```
> ça nous donnera les alias et les valeurs qui sont associées

```shell
$ php bin/console debug:container --parameters

Symfony Container Parameters
============================

 ------------------------------------------------------------- --------------------------------------------------------------------------     
  Parameter                                                     Value
 ------------------------------------------------------------- --------------------------------------------------------------------------     
  asset.request_context.base_path                               null
  asset.request_context.secure                                  null
  cache.prefix.seed                                             _C:\wamp64\www\tuto_symfony.App_KernelDevDebugContainer
  console.command.ids                                           []
  data_collector.templates                                      {"data_collector.request":["request","@WebProfiler\/Collecto...
  debug.container.dump                                          C:\wamp64\www\tuto_symfony\var\cache\dev/App_KernelDevDebugContainer.xml      
  debug.error_handler.throw_at                                  -1
  debug.file_link_format                                        %env(default::SYMFONY_IDE)%
  doctrine.class                                                Doctrine\Bundle\DoctrineBundle\Registry
  doctrine.connections                                          {"default":"doctrine.dbal.default_connection"}
  doctrine.data_collector.class                                 Doctrine\Bundle\DoctrineBundle\DataCollector\DoctrineDataCollector
  doctrine.dbal.configuration.class                             Doctrine\DBAL\Configuration
  doctrine.dbal.connection.event_manager.class                  Symfony\Bridge\Doctrine\ContainerAwareEventManager
  doctrine.dbal.connection_factory.class                        Doctrine\Bundle\DoctrineBundle\ConnectionFactory
  doctrine.dbal.connection_factory.types                        []
  doctrine.dbal.events.mysql_session_init.class                 Doctrine\DBAL\Event\Listeners\MysqlSessionInit
  doctrine.dbal.events.oracle_session_init.class                Doctrine\DBAL\Event\Listeners\OracleSessionInit
  doctrine.default_connection                                   default
  doctrine.default_entity_manager                               default
  doctrine.entity_managers                                      {"default":"doctrine.orm.default_entity_manager"}
  doctrine.migrations.preferred_connection                      null
  doctrine.migrations.preferred_em                              null
  doctrine.orm.auto_generate_proxy_classes                      true
  doctrine.orm.cache.apc.class                                  Doctrine\Common\Cache\ApcCache
  doctrine.orm.cache.array.class                                Doctrine\Common\Cache\ArrayCache
  doctrine.orm.cache.memcache.class                             Doctrine\Common\Cache\MemcacheCache
  doctrine.orm.cache.memcache_host                              localhost
  doctrine.orm.cache.memcache_instance.class                    Memcache
  doctrine.orm.cache.memcache_port                              11211
  doctrine.orm.cache.memcached.class                            Doctrine\Common\Cache\MemcachedCache
  doctrine.orm.cache.memcached_host                             localhost
  doctrine.orm.cache.memcached_instance.class                   Memcached
  doctrine.orm.cache.memcached_port                             11211
  doctrine.orm.cache.redis.class                                Doctrine\Common\Cache\RedisCache
  doctrine.orm.cache.redis_host                                 localhost
  doctrine.orm.cache.redis_instance.class                       Redis
  doctrine.orm.cache.redis_port                                 6379
  doctrine.orm.cache.wincache.class                             Doctrine\Common\Cache\WinCacheCache
  doctrine.orm.cache.xcache.class                               Doctrine\Common\Cache\XcacheCache
  doctrine.orm.cache.zenddata.class                             Doctrine\Common\Cache\ZendDataCache
  doctrine.orm.configuration.class                              Doctrine\ORM\Configuration
  doctrine.orm.enable_lazy_ghost_objects                        true
  doctrine.orm.entity_listener_resolver.class                   Doctrine\Bundle\DoctrineBundle\Mapping\ContainerEntityListenerResolver        
  doctrine.orm.entity_manager.class                             Doctrine\ORM\EntityManager
  doctrine.orm.listeners.attach_entity_listeners.class          Doctrine\ORM\Tools\AttachEntityListenersListener
  doctrine.orm.listeners.resolve_target_entity.class            Doctrine\ORM\Tools\ResolveTargetEntityListener
  doctrine.orm.manager_configurator.class                       Doctrine\Bundle\DoctrineBundle\ManagerConfigurator
  doctrine.orm.metadata.annotation.class                        Doctrine\ORM\Mapping\Driver\AnnotationDriver
  doctrine.orm.metadata.attribute.class                         Doctrine\ORM\Mapping\Driver\AttributeDriver
  doctrine.orm.metadata.driver_chain.class                      Doctrine\Persistence\Mapping\Driver\MappingDriverChain
  doctrine.orm.metadata.php.class                               Doctrine\ORM\Mapping\Driver\PHPDriver
  doctrine.orm.metadata.staticphp.class                         Doctrine\ORM\Mapping\Driver\StaticPHPDriver
  doctrine.orm.metadata.xml.class                               Doctrine\ORM\Mapping\Driver\SimplifiedXmlDriver
  doctrine.orm.metadata.yml.class                               Doctrine\ORM\Mapping\Driver\SimplifiedYamlDriver
  doctrine.orm.naming_strategy.default.class                    Doctrine\ORM\Mapping\DefaultNamingStrategy
  doctrine.orm.naming_strategy.underscore.class                 Doctrine\ORM\Mapping\UnderscoreNamingStrategy
  doctrine.orm.proxy_cache_warmer.class                         Symfony\Bridge\Doctrine\CacheWarmer\ProxyCacheWarmer
  doctrine.orm.proxy_dir                                        C:\wamp64\www\tuto_symfony\var\cache\dev/doctrine/orm/Proxies
  doctrine.orm.proxy_namespace                                  Proxies
  doctrine.orm.quote_strategy.ansi.class                        Doctrine\ORM\Mapping\AnsiQuoteStrategy
  doctrine.orm.quote_strategy.default.class                     Doctrine\ORM\Mapping\DefaultQuoteStrategy
  doctrine.orm.second_level_cache.cache_configuration.class     Doctrine\ORM\Cache\CacheConfiguration
  doctrine.orm.second_level_cache.default_cache_factory.class   Doctrine\ORM\Cache\DefaultCacheFactory
  doctrine.orm.second_level_cache.default_region.class          Doctrine\ORM\Cache\Region\DefaultRegion
  doctrine.orm.second_level_cache.filelock_region.class         Doctrine\ORM\Cache\Region\FileLockRegion
  doctrine.orm.second_level_cache.logger_chain.class            Doctrine\ORM\Cache\Logging\CacheLoggerChain
  doctrine.orm.second_level_cache.logger_statistics.class       Doctrine\ORM\Cache\Logging\StatisticsCacheLogger
  doctrine.orm.second_level_cache.regions_configuration.class   Doctrine\ORM\Cache\RegionsConfiguration
  doctrine.orm.security.user.provider.class                     Symfony\Bridge\Doctrine\Security\User\EntityUserProvider
  doctrine.orm.validator.unique.class                           Symfony\Bridge\Doctrine\Validator\Constraints\UniqueEntityValidator
  doctrine.orm.validator_initializer.class                      Symfony\Bridge\Doctrine\Validator\DoctrineInitializer
  env(VAR_DUMPER_SERVER)                                        127.0.0.1:9912
  event_dispatcher.event_aliases                                {"Symfony\\Component\\Console\\Event\\ConsoleCommandEvent":"...
  form.type_extension.csrf.enabled                              true
  form.type_extension.csrf.field_name                           _token
  form.type_guesser.doctrine.class                              Symfony\Bridge\Doctrine\Form\DoctrineOrmTypeGuesser
  fragment.path                                                 /_fragment
  fragment.renderer.hinclude.global_template                    null
  kernel.build_dir                                              C:\wamp64\www\tuto_symfony\var\cache\dev
  kernel.bundles                                                {"FrameworkBundle":"Symfony\\Bundle\\FrameworkBundle\\Framew...
  kernel.bundles_metadata                                       {"FrameworkBundle":{"path":"C:\\wamp64\\www\\tuto_symfony\\v...
  kernel.cache_dir                                              C:\wamp64\www\tuto_symfony\var\cache\dev
  kernel.charset                                                UTF-8
  kernel.container_class                                        App_KernelDevDebugContainer
  kernel.debug                                                  true
  kernel.default_locale                                         en
  kernel.enabled_locales                                        []
  kernel.environment                                            dev
  kernel.error_controller                                       error_controller
  kernel.http_method_override                                   true
  kernel.logs_dir                                               C:\wamp64\www\tuto_symfony\var\log
  kernel.project_dir                                            C:\wamp64\www\tuto_symfony
  kernel.runtime_environment                                    %env(default:kernel.environment:APP_RUNTIME_ENV)%
  kernel.runtime_mode                                           %env(query_string:default:container.runtime_mode:APP_RUNTIME_MODE)%
  kernel.runtime_mode.cli                                       %env(not:default:kernel.runtime_mode.web:)%
  kernel.runtime_mode.web                                       %env(bool:default::key:web:default:kernel.runtime_mode:)%
  kernel.runtime_mode.worker                                    %env(bool:default::key:worker:default:kernel.runtime_mode:)%
  kernel.secret                                                 %env(APP_SECRET)%
  kernel.trust_x_sendfile_type_header                           false
  kernel.trusted_hosts                                          []
  monolog.handlers_to_channels                                  {"monolog.handler.console":{"type":"exclusive","elements":["...
  monolog.swift_mailer.handlers                                 []
  monolog.use_microseconds                                      true
  profiler.storage.dsn                                          file:C:\wamp64\www\tuto_symfony\var\cache\dev/profiler
  profiler_listener.only_exceptions                             false
  profiler_listener.only_main_requests                          false
  request_listener.http_port                                    80
  request_listener.https_port                                   443
  router.cache_dir                                              C:\wamp64\www\tuto_symfony\var\cache\dev
  router.request_context.base_url
  router.request_context.host                                   localhost
  router.request_context.scheme                                 http
  router.resource                                               kernel::loadRoutes
  security.access.denied_url                                    null
  security.authentication.hide_user_not_found                   true
  security.authentication.manager.erase_credentials             true
  security.authentication.session_strategy.strategy             migrate
  security.firewalls                                            ["dev","main"]
  security.logout_uris                                          []
  security.role_hierarchy.roles                                 []
  serializer.mapping.cache.file                                 C:\wamp64\www\tuto_symfony\var\cache\dev/serialization.php
  session.metadata.storage_key                                  _sf2_meta
  session.metadata.update_threshold                             0
  session.save_path                                             null
  session.storage.options                                       {"cache_limiter":"0","cookie_secure":"auto","cookie_httponly...
  translator.default_path                                       C:\wamp64\www\tuto_symfony/translations
  translator.logging                                            false
  twig.default_path                                             C:\wamp64\www\tuto_symfony/templates
  twig.form.resources                                           ["form_div_layout.html.twig","bootstrap_5_layout.html.twig"]
  validator.mapping.cache.file                                  C:\wamp64\www\tuto_symfony\var\cache\dev/validation.php
  validator.translation_domain                                  validators
  web_profiler.debug_toolbar.intercept_redirects                false
  web_profiler.debug_toolbar.mode                               2
 ------------------------------------------------------------- --------------------------------------------------------------------------     


 // To search for a specific parameter, re-run this command with a search term. (e.g. debug:container
 // --parameter=kernel.debug)
```

> [!WARNING]
> grep ne fonctionnera pas sous Windows, il faudra utiliser l'alternative `Select-String` dans Powershell

Ne fonctionne pas sous windows :
```shell
php bin/console debug:container --parameters | grep dir
```

La commande `Select-String` qui est l'équivalent de `grep` dans Unix
Pour contourner ce problème ouvrez un powershell est tapez la commande suivante :
```shell
php bin/console debug:container --parameters | Select-String "dir"
```

Celui qui nous interessera sera le `kernel.project_dir`.

Pour tester `dd($this->getParameter('kernel.project_dir'));`
```
RecipeController.php on line 63:
"C:\wamp64\www\tuto_symfony"
```

On aura pas besoin de créer le dossier car la fonction `move()` si elle ne trouve pas le dossier va automatiquement le créer.
```php
$file->move($this->getParameter('kernel.project_dir') . '/public/recettes/images', $filename);
```

`RecipeController.php`
```php
    #[Route('/{id}', name: 'edit', methods: ['GET', 'POST'], requirements: ['id' => Requirement::DIGITS])]
    public function edit(Recipe $recipe, Request $request, EntityManagerInterface $em)
    {
        $form = $this->createForm(RecipeType::class, $recipe);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            /** @var UploadedFile $file */
            $file = $form->get('thumbnailFile')->getData();
            $filename = $recipe->getId() . '.' . $file->getClientOriginalExtension();
            $file->move($this->getParameter('kernel.project_dir') . '/public/recettes/images', $filename);
            $recipe->setThumbnail($filename);
            $em->flush();
            $this->addFlash(
                'success',
                'La recette a bien été modifiée'
            );
            return $this->redirectToRoute('admin.recipe.index');
        }

        return $this->render('admin/recipe/edit.html.twig', [
            'recipe' => $recipe,
            'formTitle' => 'Editer : ' . $recipe->getTitle(),
            'form' => $form
        ]);
    }
```

Maintenant il va falloir gérer :
- quand on envoit une nouvelle image il supprime l'ancienne
- quand je supprime une recette, il supprime l'image associé.