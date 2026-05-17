# SkillHub — EC03 Back-end Symfony (SQLite)

## Présentation

SkillHub est un projet back-end Symfony réalisé pour l'épreuve EC03. Il permet de gérer des utilisateurs, des formations, des catégories, des inscriptions, l'authentification et une base de données SQLite locale.

## Fonctionnalités

- Gestion des utilisateurs
- Gestion des formations
- Gestion des catégories
- Gestion des inscriptions
- Authentification sécurisée
- Base SQLite locale
- Données de test avec fixtures

## Stack technique

- Symfony
- PHP
- Doctrine ORM
- SQLite
- Twig
- Symfony Security
- Doctrine Fixtures Bundle

## Installation

### 1. Créer le projet

```bash
symfony new skillhub --webapp
```

### 2. Entrer dans le dossier

```bash
cd skillhub
```

### 3. Lancer le serveur

```bash
symfony serve
```

### 4. Ouvrir l'application

Par défaut, l'application est accessible sur `http://127.0.0.1:8000`.

## Configuration SQLite

Symfony configure la connexion via la variable d'environnement `DATABASE_URL`, et un format SQLite valide est par exemple `sqlite:///%kernel.project_dir%/var/skillhub.db`.[1][2]

Dans le fichier `.env` :

```env
DATABASE_URL="sqlite:///%kernel.project_dir%/var/skillhub.db"
```

Créer ensuite la base :

```bash
php bin/console doctrine:database:create
```

Le fichier généré sera :

```text
var/skillhub.db
```

## Entités du projet

### User

- `email` : string
- `password` : string
- `roles` : json
- `firstname` : string
- `lastname` : string

### Formation

- `title` : string
- `description` : text
- `duration` : integer
- `createdAt` : datetime immutable
- `category` : relation vers `Category`

### Category

- `name` : string

### Enrollment

- `user` : relation ManyToOne vers `User`
- `formation` : relation ManyToOne vers `Formation`
- `enrolledAt` : datetime immutable

## Relations Doctrine

- `Category` → `Formation` : OneToMany / ManyToOne
- `User` → `Enrollment` : OneToMany / ManyToOne
- `Formation` → `Enrollment` : OneToMany / ManyToOne

## Création des entités

Pour générer une entité :

```bash
php bin/console make:entity
```

## Migrations

Créer une migration :

```bash
php bin/console make:migration
```

Appliquer les migrations :

```bash
php bin/console doctrine:migrations:migrate
```

Réinitialiser la base SQLite si nécessaire :

```bash
rm var/skillhub.db
php bin/console doctrine:database:create
php bin/console doctrine:migrations:migrate
```

## CRUD automatique

Générer le CRUD de `Formation` :

```bash
php bin/console make:crud Formation
```

Générer le CRUD de `Category` :

```bash
php bin/console make:crud Category
```

## Authentification

Symfony fournit la commande `php bin/console make:security:form-login` pour générer un formulaire de connexion moderne ainsi que la configuration de sécurité associée.[3]

Créer l'utilisateur :

```bash
php bin/console make:user
```

Créer l'authentification :

```bash
php bin/console make:security:form-login
```

Routes habituelles :

- `/login`
- `/logout`

Protéger une route ou un contrôleur :

```php
#[IsGranted('ROLE_ADMIN')]
```

## Rôles

- `ROLE_USER`
- `ROLE_ADMIN`

## Fixtures

DoctrineFixturesBundle sert à charger des données de test dans Symfony.[4]

Installer les fixtures :

```bash
composer require --dev orm-fixtures
```

Créer une fixture :

```bash
php bin/console make:fixtures
```

Charger les données :

```bash
php bin/console doctrine:fixtures:load
```

## Debug utile

Lister les routes :

```bash
php bin/console debug:router
```

Vérifier le mapping Doctrine :

```bash
php bin/console doctrine:mapping:info
```

Tester une requête SQL simple :

```bash
php bin/console doctrine:query:sql "SELECT 1"
```

## Arborescence conseillée

```text
src/
├── Controller/
├── Entity/
├── Repository/
├── Form/
├── Security/
├── Service/

templates/
migrations/
var/
```

## Bonnes pratiques EC03

- Utiliser les migrations à chaque modification de structure
- Définir des relations Doctrine propres et cohérentes
- Mettre la logique métier dans des services quand c'est pertinent
- Valider les formulaires
- Activer la sécurité par rôles
- Prévoir des fixtures pour les démonstrations et les tests

## Erreurs fréquentes

- Oublier `make:migration`
- Ne pas lancer `doctrine:migrations:migrate`
- Oublier les fixtures
- Mal définir les relations
- Mettre trop de logique métier dans les contrôleurs
- Laisser la base SQLite désynchronisée

## Méthode recommandée pendant l'examen

1. Créer les entités et les relations
2. Générer les migrations
3. Mettre en place l'authentification
4. Générer les CRUD
5. Ajouter les fixtures
6. Vérifier le comportement dans le navigateur

## Objectif démontré

Ce projet montre une architecture Symfony claire, une base SQLite locale, une authentification fonctionnelle, des relations Doctrine maîtrisées et une base exploitable pour une application légère.
