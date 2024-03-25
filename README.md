# Découverte de Symfony

## Installation de Symfony

```shell
composer create-project symfony/skeleton:"7.0.*" my_project_directory
cd my_project_directory
composer require webapp
```
## Lancement du serveur PHP en local depuis terminal

```shell
# --------------------------------------------------
# Pour démarrer le serveur dans Visual Studio Code
# --------------------------------------------------
php -S localhost:8000 -t public

# --------------------------------------------------
# Pour démarrer le serveur Symfony
# --------------------------------------------------
symfony server:start

# --------------------------------------------------
# Pour arrêter le serveur Symfony
# --------------------------------------------------
symfony server:stop
```
## Lancement d'adminer

http://localhost:8000/adminer.php

## Installation de serveur mail

- Maildev (nécessite NodeJs) https://maildev.github.io/maildev/
- Mailpit (est un executable) https://github.com/axllent/mailpit/releases/

Sous Windows : mettre le fichier mailpit.exe dans le dossier `\bin` de Symfony
Executer la commande dans le terminal
```shell
bin\mailpit.exe
```

Sous Linux : mettre le fichier mailpit dans le dossier `\bin` de Symfony
Executer la commande dans le terminal
```shell
chmod +x bin/mailpit
./bin/mailpit
```

Ouvrir dans le navigateur `http://localhost:8025/` pour accéder à l'interface de Mailpit.

## ORM Doctrine

**Méthode Find :**

- `find($id)` : permettra de retrouver un enregistrement en particulier
- `findBy()` : permettra de spécifier un critère sous forme de tableau et de récupérer des enregistrements en fonction d'une condition sur un champ
- `findAll()` : permettra de récupérer tous les enregistrements 
- `findOneBy()` : permettra de récupérer un enregistrement en fonction d'un critère en particulier

## File Upload

**VichUploaderBundle**

Pour gérer plus facilement le système d'envoi de fichier il est possible d'utiliser le bundle VichUploaderBundle qui permet d'attacher automatiquement un système d'upload de fichier aux évènements de notre entité.

Source : 
- https://github.com/dustin10/VichUploaderBundle

Documentation :
- Getting started with VichUploaderBundle : https://github.com/dustin10/VichUploaderBundle/blob/master/docs/index.md

## Commandes Symfony

```shell
# Liste toutes les commandes disponibles dans Symfony
php bin/console


# -------------------------------------------
# Création d'un contrôleur
# -------------------------------------------

# Crée un nouveau contrôleur appelé NameController
php bin/console make:controller NameController

# Affiche toutes les routes définies dans l'application
php bin/console debug:router


# -------------------------------------------
# Création d'entité
# -------------------------------------------

# Crée une nouvelle entité
php bin/console make:entity

# Crée une nouvelle entité avec un nom spécifique
php bin/console make:entity EntityName

# Génère une nouvelle migration basée sur les différences entre vos entités et la base de données
php bin/console make:migration

# Exécute les migrations et met à jour la base de données
php bin/console doctrine:migrations:migrate


# -------------------------------------------
# Création du formulaire avec le FormBuilder
# -------------------------------------------

# Crée un nouveau formulaire
php bin/console make:form

# Crée un nouveau formulaire avec un type spécifique
php bin/console make:form FormType

# Affiche tous les services liés aux formulaires qui peuvent être injectés
php bin/console debug:autowiring form


# -------------------------------------------
# Création d'un validateur
# -------------------------------------------

# Crée un nouveau validateur
php bin/console make:validator

# Affiche tous les services liés aux validateurs qui peuvent être injectés
php bin/console debug:autowiring valid


# -------------------------------------------
# Création d'un composant de sécurité
# -------------------------------------------

# Crée une nouvelle entité User pour la gestion des utilisateurs
php bin/console make:user

# Crée un nouveau système d'authentification
php bin/console make:auth

# Génère une nouvelle migration après la création du système d'authentification
php bin/console make:migration

# Exécute les migrations après la création du système d'authentification
php bin/console doctrine:migrations:migrate

# Crée un formulaire d'inscription
php bin/console make:registration-form

# Affiche les services liés au mot de passe qui peuvent être injectés
php bin/console debug:autowiring Password


# -------------------------------------------
# Gestion des assets de l'application
# -------------------------------------------

# Exécute la commande pour versionner et rendre publics les assets de l'application
php bin/console asset-map:compile


# -------------------------------------------
# Debugging
# -------------------------------------------

# Affiche tous les services qui peuvent être automatiquement injectés.
# Utile pour comprendre quels services sont disponibles pour l'injection de dépendances.
php bin/console debug:autowiring

# Affiche tous les paramètres du conteneur de services.
# Utile pour voir les paramètres actuellement utilisés par l'application.
php bin/console debug:container --parameters


# -------------------------------------------
# Réinitialiser la base de données
# -------------------------------------------

# Supprime la base de données. L'option --force est nécessaire pour confirmer cette action.
php bin/console doctrine:database:drop --force

# Crée une nouvelle base de données.
php bin/console doctrine:database:create

# Exécute les migrations, ce qui met à jour la base de données avec les dernières modifications du schéma.
php bin/console doctrine:migrations:migrate


# -----------------------------------------------------------
# Installation runtime après récupération sur dépôt Github
# -----------------------------------------------------------

# Cette commande installe le composant "symfony/runtime" via Composer.
# Ce composant fournit les fonctionnalités nécessaires pour exécuter les applications Symfony.
composer require symfony/runtime

```

### Autres commandes Symfony à retenir :

- `php bin/console list` : Affiche la liste de toutes les commandes disponibles.
- `php bin/console debug:config` : Affiche la configuration actuelle de l'application, elle affichera une liste de bundles et les paramètres associés à ces bundles.
- `php bin/console cache:clear` : Efface le cache, supprime tous les fichiers en cache et améliore les performances de l'application.
- `php bin/console assets:install` : Installe les ressources web du bundle sous un répertoire public.
- `php bin/console doctrine:fixtures:load` : Charge les données de test dans la base de données.
- `php bin/console security:check` : Vérifie la sécurité de l'application.

Pour obtenir de l'aide sur une commande spécifique, vous pouvez utiliser l'option --help avec la commande. 
Par exemple, pour obtenir de l'aide sur la commande assets:install, vous pouvez exécuter php bin/console assets:install --help.

## Documentations

- Doc Symfony : https://symfony.com/doc/current/index.html
- Référence Symfony : https://symfony.com/doc/current/reference/index.html
- Doc Twig : https://twig.symfony.com/doc/
- Doctrine ORM : https://www.doctrine-project.org/projects/doctrine-orm/en/3.1/index.html