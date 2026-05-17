# 📘 SkillHub — EC03 Back-end Symfony (SQLite)

## 🎯 Objectif du projet

SkillHub est une application back-end Symfony développée dans le cadre de l’épreuve EC03.  
Elle permet de gérer :

- des utilisateurs
- des formations
- des catégories
- des inscriptions
- une authentification sécurisée
- une base SQLite locale
- des données de test (fixtures)

---

## ⚙️ Stack technique

- Symfony (framework PHP)
- PHP
- Doctrine ORM
- SQLite (base de données fichier)
- Twig (templates)
- Symfony Security
- DoctrineFixturesBundle (fixtures)[web:1][web:23]

---

## 🚀 Installation du projet

### 1. Créer le projet

```bash
symfony new skillhub --webapp
```

### 2. Se déplacer dans le dossier

```bash
cd skillhub
```

### 3. Lancer le serveur

```bash
symfony serve
```

### 4. Accès à l’application

Par défaut : [http://127.0.0.1:8000](http://127.0.0.1:8000)

---

## 🗄️ Base de données SQLite

Symfony utilise la variable d’environnement `DATABASE_URL` pour configurer la connexion Doctrine.[web:1][web:2][web:8]

### 1. Configuration `.env`

Dans le fichier `.env` :

```env
DATABASE_URL="sqlite:///%kernel.project_dir%/var/skillhub.db"
```

Exemples similaires dans la doc Symfony : `sqlite:///%kernel.project_dir%/var/app.db`.[web:1][web:2][web:8]

### 2. Créer la base

```bash
php bin/console doctrine:database:create
```

### 3. Fichier généré

```text
var/skillhub.db
```

---

## 🧩 Entités du projet

### 👤 User

- `email` (string)
- `password` (string)
- `roles` (json)
- `firstname` (string)
- `lastname` (string)

### 📚 Formation

- `title` (string)
- `description` (text)
- `duration` (integer)
- `createdAt` (datetime_immutable)
- `category` (relation vers `Category`)

### 🏷 Category

- `name` (string)

### 🧾 Enrollment

- `user` (ManyToOne vers `User`)
- `formation` (ManyToOne vers `Formation`)
- `enrolledAt` (datetime_immutable)

---

## 🔗 Relations Doctrine

- `Category` → `Formation` : OneToMany / ManyToOne
- `User` → `Enrollment` : OneToMany / ManyToOne
- `Formation` → `Enrollment` : OneToMany / ManyToOne

---

## 🧱 Création des entités

Commande principale :

```bash
php bin/console make:entity
```

Tu définis ensuite les champs et les relations depuis l’assistant en ligne de commande.

---

## 🧱 Migrations

### Créer une migration

```bash
php bin/console make:migration
```

### Appliquer les migrations

```bash
php bin/console doctrine:migrations:migrate
```

### Reset base SQLite (si besoin)

```bash
rm var/skillhub.db
php bin/console doctrine:database:create
php bin/console doctrine:migrations:migrate
```

---

## ⚡ CRUD automatique

Générer le CRUD de `Formation` :

```bash
php bin/console make:crud Formation
```

Générer le CRUD de `Category` :

```bash
php bin/console make:crud Category
```

Les CRUD générés comprennent les contrôleurs, les formulaires et les templates Twig.

---

## 🔐 Authentification (moderne)

Symfony propose la génération d’un utilisateur et d’un login form via le composant Security.[web:1]

### 1. Créer l’utilisateur

```bash
php bin/console make:user
```

### 2. Créer le formulaire de login

```bash
php bin/console make:security:form-login
```

Cette commande génère la configuration de sécurité, le contrôleur de login et le formulaire de connexion.[web:1]

### Routes générées

- `/login`
- `/logout`

### Protection des routes

Exemple d’attribut sur un contrôleur ou une méthode :

```php
#[IsGranted('ROLE_ADMIN')]
```

### 👥 Rôles utilisateurs

- `ROLE_USER`
- `ROLE_ADMIN`

---

## 🧪 Fixtures (données de test)

DoctrineFixturesBundle permet de charger des données de test programmatiquement dans Doctrine ORM.[web:23]

### 1. Installer les fixtures

```bash
composer require --dev orm-fixtures
```

### 2. Créer une classe de fixtures

```bash
php bin/console make:fixtures
```

### 3. Charger les données

```bash
php bin/console doctrine:fixtures:load
```

---

## 🧠 CRUD & FORM — IMPORTANT (EXAM EC03)

### ❌ Erreurs fréquentes dans un CRUD généré

- `createdAt` affiché dans le formulaire.
- `ID` affiché dans un dropdown.
- Logique métier dans Twig.
- Formulaires non nettoyés.

### 🧾 1. Cacher `createdAt` correctement

Mauvaise pratique : supprimer uniquement le champ dans Twig (le formulaire contient toujours le champ).  
Bonne pratique : ne pas inclure `createdAt` dans le `FormType`.

```php
// src/Form/FormationType.php
public function buildForm(FormBuilderInterface $builder, array $options): void
{
    $builder
        ->add('title')
        ->add('description')
        ->add('duration')
        ->add('category');
        // createdAt NON présent
}
```

Bonus propre côté entity :

```php
public function __construct()
{
    $this->createdAt = new \DateTimeImmutable();
}
```

### 🏷️ 2. Afficher les noms dans les dropdowns

Si le dropdown affiche des IDs, il faut fournir un label lisible. Symfony permet d’utiliser `choice_label` dans `EntityType`, et en l’absence de réglage explicite il repose sur `__toString()` pour convertir l’objet en texte.[web:19][web:22]

Solution recommandée :

```php
// src/Entity/Category.php
public function __toString(): string
{
    return $this->name;
}
```

Alternative dans le formulaire :

```php
->add('category', EntityType::class, [
    'class' => Category::class,
    'choice_label' => 'name'
])
```

### 🎨 3. Modification Twig CRUD

Twig doit gérer uniquement l’affichage, tandis que la logique d’accès aux données est portée par Doctrine et la logique métier par les services/entités.[web:18][web:21]

Exemple propre dans `index.html.twig` :

```twig
<td>{{ formation.title }}</td>
<td>{{ formation.duration }}</td>
<td>{{ formation.category.name }}</td>
```

On évite d’afficher des IDs ou des champs inutiles, et on supprime les colonnes comme `createdAt` si elles ne sont pas nécessaires à l’affichage.

### 🧠 RÈGLES IMPORTANTES

- FormType = logique des formulaires.
- Entity = logique métier.
- Twig = affichage uniquement.
- Doctrine = gestion des relations et de la persistance.[web:1][web:4]

### ⚠️ ERREURS FATALES EXAM

- Oublier `make:migration`.
- Ne pas lancer `doctrine:migrations:migrate`.
- Relations cassées.
- CRUD non testé (erreurs 500 en navigation).
- Sécurité incomplète (routes non protégées).
- Logique métier directement dans Twig ou dans les contrôleurs.

---

## 🔍 Debug & vérification

### Routes disponibles

```bash
php bin/console debug:router
```

### Mapping Doctrine

```bash
php bin/console doctrine:mapping:info
```

### Test requête SQL simple

```bash
php bin/console doctrine:query:sql "SELECT 1"
```

---

## 🧠 Structure du projet

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

---

## ⚠️ Bonnes pratiques EC03

- Toujours utiliser les migrations pour modifier le schéma.
- Relations Doctrine propres et normalisées.
- Logique métier dans les services ou entités, pas seulement dans les contrôleurs.
- Validation correcte des formulaires.
- Sécurité active (rôles, accès protégés).
- Fixtures obligatoires pour les tests et la démonstration.

---

## 🚨 Erreurs fréquentes

- Oublier `php bin/console make:migration`.
- Ne pas exécuter `php bin/console doctrine:migrations:migrate`.
- Ne pas créer/charger de fixtures.
- Relations mal définies (clé étrangère manquante, cascade incorrecte).
- Trop de logique métier dans les contrôleurs ou Twig.
- Base SQLite non synchronisée avec les entités/migrations.

---

## 🧪 Méthode recommandée (EXAM 4H)

1. Créer les entités + relations.
2. Générer les migrations et les exécuter.
3. Mettre en place l’authentification.
4. Générer les CRUD et nettoyer les formulaires/Twig.
5. Ajouter les fixtures.
6. Tester dans le navigateur (tous les cas principaux).

---

## 🎯 Conclusion

Ce projet démontre :

- une architecture Symfony propre avec séparation des responsabilités
- une base SQLite fonctionnelle configurée via `DATABASE_URL`[web:1][web:2][web:8]
- une sécurité complète avec formulaire de login
- des relations Doctrine maîtrisées
- des fixtures prêtes pour les tests
- une application prête pour une petite mise en production ou une démonstration EC03
