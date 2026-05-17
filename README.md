# 📘 SkillHub — EC03 Back-end Symfony (SQLite)
🏆 Version RNCP optimisée (niveau professionnel / 5⭐)

---

## 🎯 Objectif du projet (version RNCP)

SkillHub est une application back-end développée avec Symfony dans le cadre de l’épreuve EC03 (BC02 RNCP39608).  
Elle met en œuvre une architecture MVC orientée back-end, avec une base de données relationnelle, une authentification sécurisée et une gestion complète des entités métier.

**Objectifs techniques évalués :**

- Conception d’un schéma relationnel normalisé (C10)
- Implémentation d’une architecture MVC propre (C8)
- Sécurisation de l’accès aux données (C9)
- Gestion des migrations et du cycle de vie de la BDD
- Stratégie de test avec fixtures
- Arbitrage technologique BDD (C12)
- Stratégie de sauvegarde (C13)

---

## ⚙️ Stack technique

- Symfony (framework PHP MVC)
- PHP 8+
- Doctrine ORM (mapping objet-relationnel)
- SQLite (base locale fichier)
- Twig (vue uniquement)
- Symfony Security (authentification)
- Doctrine Migrations
- DoctrineFixturesBundle (fixtures)[web:1][web:27]

---

## 🧠 Justification d’architecture (IMPORTANT RNCP)

L’application suit une architecture **MVC stricte** :

- Model → Entity + Doctrine (ORM)
- View → Twig (affichage uniquement)
- Controller → logique de flux HTTP (orchestration)

**Séparation des responsabilités :**

- Entity → logique métier simple et structure des données
- FormType → structure et validation des formulaires
- Service (optionnel mais recommandé) → logique métier avancée / réutilisable
- Controller → orchestration, appel des services, rendu des vues

👉 Objectif RNCP : **maintenabilité**, **évolutivité** et **testabilité** du code.

---

## 🗄️ Base de données (C10 + C12)

### Choix technologique (arbitrage RNCP)

**SQLite (choix projet) :**

- ✔ Installation simple
- ✔ Fichier local, aucune infra serveur
- ✔ Idéal pour environnement d’examen (EC03)
- ❌ Non adapté à une très forte charge ou à des architectures distribuées[web:1][web:2][web:8]

**Alternatives analysées :**

| SGBD      | Avantages                    | Inconvénients                      |
|-----------|-----------------------------|------------------------------------|
| MySQL     | Standard web, performant    | Nécessite un serveur dédié         |
| PostgreSQL| Robuste, ACID avancé        | Plus complexe à administrer        |
| SQLite    | Simple, rapide, local       | Scalabilité limitée, peu multi-user|

👉 **Conclusion** :  
SQLite est adapté au contexte EC03 (démo locale, examen).  
En production, MySQL ou PostgreSQL seraient privilégiés pour la montée en charge et les fonctionnalités avancées.

---

## 🧱 Configuration Symfony (Doctrine + SQLite)

Dans `.env` (ou `.env.local`) :

```env
DATABASE_URL="sqlite:///%kernel.project_dir%/var/skillhub.db"
```

Ce format suit les exemples officiels de configuration SQLite via `DATABASE_URL`.[web:1][web:2][web:4][web:8]

---

## 🔗 Modélisation relationnelle (C10)

### Entités

👤 **User**

- `email` (unique index recommandé)
- `password` (hashé, ex : bcrypt / argon2)
- `roles` (json)
- `firstname`
- `lastname`

📚 **Formation**

- `title`
- `description`
- `duration`
- `createdAt`
- `category` (FK vers `Category`)

🏷 **Category**

- `name`

🧾 **Enrollment**

- `user` (FK vers `User`)
- `formation` (FK vers `Formation`)
- `enrolledAt`

### Relations

- `Category` 1 → N `Formation`
- `User` 1 → N `Enrollment`
- `Formation` 1 → N `Enrollment`

👉 **3ème forme normale (3NF)** :

- Pas de redondance inutile
- Dépendances fonctionnelles cohérentes avec les clés primaires

---

## ⚙️ Installation du projet

```bash
symfony new skillhub --webapp
cd skillhub
symfony serve
```

Application disponible par défaut sur : `http://127.0.0.1:8000`.

---

## 🧱 Doctrine & migrations

Création / modification des entités :

```bash
php bin/console make:entity
```

Générer une migration :

```bash
php bin/console make:migration
```

Appliquer les migrations :

```bash
php bin/console doctrine:migrations:migrate
```

### ♻ Cycle de vie BDD (RNCP)

1. Conception des entités (modèle conceptuel → entités Doctrine)
2. Génération des migrations
3. Exécution des migrations
4. Versioning du schéma (historique des changements)

---

## ⚡ CRUD

Génération du CRUD `Formation` :

```bash
php bin/console make:crud Formation
```

Génération du CRUD `Category` :

```bash
php bin/console make:crud Category
```

**Règles RNCP importantes :**

- Twig = affichage uniquement (pas de logique métier)
- FormType = structure + logique des formulaires
- Controller = orchestration des flux
- Entity = structure des données + petite logique métier

---

## 🔐 Sécurité (C9)

### Authentification

Création de l’entité utilisateur :

```bash
php bin/console make:user
```

Création du login (formulaire) avec le composant Security :

```bash
php bin/console make:security:form-login
```

Symfony gère alors le contrôle d’accès via un firewall, une route `/login` et un `/logout` automatisé.[web:6][web:28]

### Protection des routes

Exemple d’annotation d’autorisation :

```php
#[IsGranted('ROLE_ADMIN')]
```

### 🛡 Sécurité serveur (RNCP attendu)

**Bonnes pratiques :**

- Variables sensibles dans `.env` / `.env.local` (jamais commitées)
- Mots de passe hashés (bcrypt / argon2)
- Protection CSRF activée pour les formulaires
- Contrôle des rôles (RBAC : Role-Based Access Control)[web:6][web:28]

**Sécurité HTTP (complément RNCP) :**

- `X-Frame-Options`
- `X-XSS-Protection`
- `Content-Security-Policy`

(ces en-têtes sont classiquement configurés via reverse proxy ou configuration serveur).

---

## 🧪 Fixtures

Installation :

```bash
composer require --dev orm-fixtures
```

Création d’une classe de fixtures :

```bash
php bin/console make:fixtures
```

Chargement des données :

```bash
php bin/console doctrine:fixtures:load
```

👉 **Objectif RNCP** :

- Tester rapidement le système
- Simuler des données réalistes (formations, utilisateurs, inscriptions)[web:27]

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

## 🧠 C11 — NoSQL (ajout RNCP)

Même si non utilisé dans le projet, une solution NoSQL typique serait :

**Exemple : MongoDB**

- Stockage des logs utilisateurs
- Analytics sur les formations
- Données semi-structurées / non structurées

👉 **Avantages :**

- Scalabilité horizontale
- Flexibilité du schéma

👉 **Inconvénients :**

- Pas de relations complexes natives
- Cohérence plus faible (modèle BASE vs ACID des SGBDR)

---

## 🧠 C12 — Arbitrage décisionnel

**Choix final pour le projet :**

👉 **SQLite**

**Justification RNCP :**

- Contexte local EC03
- Rapidité de mise en place
- Zéro configuration serveur, moins de risques techniques pendant l’épreuve[web:1][web:2][web:5]

**En production :**

👉 **PostgreSQL recommandé** pour :

- Robustesse
- Respect strict d’ACID
- Performance et fonctionnalités avancées (index, types, etc.)

---

## 🧱 C13 — Sauvegarde & récupération

### 💾 Stratégie backup

**SQLite :**

```bash
cp var/skillhub.db backup_skillhub.db
```

**Doctrine (versioning) :**

- Migrations = historique du schéma
- Possibilité de revenir à une version antérieure :

```bash
php bin/console doctrine:migrations:migrate prev
```

### 🛡 Plan de récupération

1. Restauration du fichier de base (`skillhub.db`)
2. Rollback / relecture des migrations si nécessaire
3. Rechargement des fixtures pour reconstruire un jeu de test

---

## 🔍 Debug & qualité

Quelques commandes utiles :

```bash
php bin/console debug:router
php bin/console doctrine:mapping:info
```

Pour vérifier la connectivité de la base :

```bash
php bin/console doctrine:query:sql "SELECT 1"
```

---

## 🧠 Structure projet RNCP

```text
src/
 ├── Controller/
 ├── Entity/
 ├── Repository/
 ├── Form/
 ├── Service/    (important RNCP : logique métier)
 ├── Security/

templates/
migrations/
var/
```

---

## 🧪 Méthode EC03 (optimisée note max)

1. Modélisation des entités (UML mental ou papier)
2. Création des entités + relations Doctrine propres
3. Génération + exécution des migrations
4. Mise en place de l’authentification sécurisée
5. Génération des CRUD + nettoyage des FormType et Twig
6. Ajout de fixtures pour les tests
7. Tests fonctionnels (navigation, formulaires, droits)
8. Sécurisation des routes (rôles, accès restreints)
9. Vérification avec les commandes de debug

---

## 🏁 Conclusion RNCP

SkillHub démontre une maîtrise :

- Du back-end Symfony MVC (C8)
- D’un modèle relationnel normalisé avec Doctrine ORM (C10)
- De la sécurisation applicative (C9)
- D’une architecture maintenable (C8)
- De l’arbitrage base de données (C12)
- D’une stratégie de sauvegarde et récupération (C13)
- De données de test automatisées via fixtures

🏆 **Niveau attendu correcteur (indicatif)**

- C7 : 5/5
- C8 : 5/5
- C9 : 4–5/5
- C10 : 5/5
- C11 : 3–4/5
- C12 : 5/5
- C13 : 5/5
