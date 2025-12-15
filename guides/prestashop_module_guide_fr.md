# Guide de Développement de Modules PrestaShop (1.7, 8, & 9)

**Rôle :** Expert Développeur PrestaShop & Architecte
**Contexte :** Basé strictement sur la documentation officielle pour PrestaShop 1.7, 8 et 9.

---

## 🧱 1. Structure du Code d'un Module PrestaShop

Une structure propre et standardisée est critique pour la maintenabilité et la compatibilité entre les versions.

### 1.1 Structure de Répertoire Standard

Pour toutes les versions supportées (1.7.x, 8.x, 9.x), la racine de votre module doit contenir le fichier PHP principal (`modulename.php`) et `composer.json` est fortement recommandé pour l'autoloading des commandes et services.

#### Architecture Recommandée (Compatible Moderne)

```
modulename/
├── composer.json               # Dépendances & Règles d'Autoloading
├── modulename.php              # Classe principale (étend Module)
├── config.xml                  # (Généré auto par export XML, ne pas éditer manuellement)
├── logo.png                    # Icône 32x32
├── src/                        # MODERNE : Classes PHP (PSR-4)
│   ├── Controller/             # Contrôleurs Symfony (Back Office)
│   ├── Entity/                 # Entités Doctrine (ou ObjectModels Legacy)
│   ├── Repository/             # Repositories Doctrine
│   ├── Form/                   # Formulaires Symfony
│   └── Grid/                   # Définitions de Grid (Listes BO Modernes)
├── controllers/                # LEGACY : Contrôleurs Front & Admin
│   ├── admin/
│   └── front/
├── views/                      # Templates & Assets
│   ├── css/
│   ├── js/
│   ├── img/
│   └── templates/
│       ├── admin/
│       ├── front/
│       └── hook/
└── upgrade/                    # Scripts de mise à jour (upgrade-X.Y.Z.php)
```

### 1.2 Différences de Version

*   **PrestaShop 1.7** : Architecture hybride. Supporte à la fois les Contrôleurs Legacy (`controllers/admin`) et les Contrôleurs Symfony (`src/Controller`). La logique est souvent mélangée dans `modulename.php`.
*   **PrestaShop 8 & 9** : Accent plus fort sur l'architecture "Moderne". Commandes, Requêtes (Queries) et Gestionnaires (Handlers) (CQRS) sont des éléments de premier plan.
    *   **Recommandation** : Placez toute la logique métier dans `src/` en utilisant des namespaces (ex : `PrestaShop\Module\NomDuModule\...`).
    *   **Legacy** : Gardez `controllers/` uniquement pour les contrôleurs Front Office (jusqu'à ce qu'une pile Front entièrement moderne soit standard) ou les onglets Admin legacy.

---

## 🧠 2. Pattern CQRS dans les Modules PrestaShop

**CQRS** (Command Query Responsibility Segregation) sépare les opérations d'**Écriture** (Commandes) des opérations de **Lecture** (Requêtes/Queries).

### 2.1 Quand utiliser CQRS ?
*   **Utiliser** : Pour une logique métier complexe, la création de nouveaux endpoints API (PS 9), ou des opérations de masse. Cela découple votre Logique du Contrôleur.
*   **Ne pas utiliser** : Pour un simple "CRUD" sur une table de base de données unique où l'ObjectModel standard suffit et où vous n'avez pas besoin de sécurité transactionnelle ou de validation complexe.

### 2.2 Composants & Règles (Strictement selon la Docs)

#### Commandes (Écriture)
*   **Définition** : Décrit une intention *unique* de changer l'état (ex : `AddAttributeGroupCommand`).
*   **Règles** :
    *   Doit être **Immuable**.
    *   Contient uniquement des types primitifs (int, string, bool, array).
    *   **AUCUNE** logique métier à l'intérieur.

#### Requêtes / Queries (Lecture)
*   **Définition** : Décrit une demande de données (ex : `GetAttributeGroupForEditing`).
*   **Règles** :
    *   Doit être **Immuable**.
    *   Contient uniquement des primitives (critères de filtrage, IDs).

#### Gestionnaires / Handlers (La Logique)
*   **Gestionnaire de Commande (Command Handler)** :
    *   Exécute la Commande.
    *   Ne devrait **PAS** retourner de données (void), sauf pour des IDs lors de la création.
    *   Devrait lancer des Exceptions Typées en cas d'échec.
*   **Gestionnaire de Requête (Query Handler)** :
    *   Retourne un DTO (Data Transfer Object) spécifique ou un tableau.
    *   Ne devrait **PAS** retourner d'objets "Internes" (comme des Entités Doctrine) directement au contrôleur ; mappez-les d'abord vers un DTO.

### 2.3 Exemple de Structure d'Implémentation

```
src/
└── Domain/
    └── Order/
        ├── Command/
        │   ├── CancelOrderCommand.php
        │   └── CancelOrderCommandHandler.php
        ├── Query/
        │   ├── GetOrderDetails.php
        │   └── GetOrderDetailsHandler.php
        └── Exception/
            └── OrderNotFoundException.php
```

> **Note** : Dans le Core PrestaShop, les classes `Command` sont souvent dans `Core/Domain` et les handlers dans `Adapter/` (si utilisation de code Legacy). À l'intérieur d'un **Module**, vous pouvez les garder ensemble dans `src/Domain` ou `src/Application`.

---

## 🧩 3. DDD (Domain-Driven Design) dans les Modules PrestaShop

Le DDD aide à organiser le code autour du "Langage Métier" plutôt que des artefacts techniques.

### 3.1 Couches dans le contexte d'un Module

1.  **Couche Domaine (Domain Layer)** (`src/Domain/`)
    *   **Quoi** : Le "Cœur" de votre logiciel.
    *   **Contient** :
        *   **Value Objects** : Paramètres typés (ex : `EmailAddress`, `Price`).
        *   **Interfaces** : Contrats pour les Repositories (ex : `OrderRepositoryInterface`).
        *   **Exceptions** : Erreurs métier (ex : `InvalidCartException`).
    *   *Contrainte* : Pas de dépendances aux classes du Core PrestaShop (Cookie, Context) si possible.

2.  **Couche Application (Application Layer)** (`src/Application/`)
    *   **Quoi** : Orchestre les tâches.
    *   **Contient** : Commandes/Requêtes CQRS, Souscripteurs d'Événements (Event Subscribers).

3.  **Couche Infrastructure (Infrastructure Layer)** (`src/Infrastructure/`)
    *   **Quoi** : Implémentation technique.
    *   **Contient** :
        *   **Repositories Doctrine** (Implémentant les Interfaces du Domaine).
        *   **Adapteurs Legacy** : Wrappers autour de `ObjectModel` ou `Db::getInstance()`.

### 3.2 Compatibilité
*   **PS 1.7** : Vous pouvez utiliser les namespaces DDD manuellement dans `src/`. Nécessite une autodiscipline plus stricte car le Core ne l'impose pas autant.
*   **PS 8/9** : L'architecture du Core elle-même a migré vers le DDD (`Core/Domain`). Votre module devrait refléter cette structure pour une intégration plus facile avec les services du Core.

---

## ✅ 4. Validité du Module & Conformité

Pour passer la validation (PrestaShop Addons ou QA strict), votre module **doit** adhérer à ces règles (issues de `techvalidation-checklist.md`).
=> Uploadez votre zip sur [validator.prestashop.com](https://validator.prestashop.com) (mentionné dans la doc).

### 4.1 Exigences Obligatoires
1.  **`index.php`** : Chaque dossier (y compris les sous-dossiers comme `img`, `css`) DOIT contenir un fichier `index.php` pour empêcher le listing du répertoire.
2.  **Licence** : Doit être compatible (Apache, MIT, AFL).
3.  **Code en Anglais** : Les variables, commentaires et noms de fonctions DOIVENT être en Anglais.
4.  **Pas de Modification Directe du Core** : Ne modifiez jamais les fichiers du core. Utilisez des **Hooks** ou des **Overrides** (Overrides déconseillés dans PS moderne).

### 4.2 Checklist de Sécurité
*   **Injection SQL** :
    *   Legacy : Utilisez `pSQL($var)` pour les chaînes, `(int)$var` pour les entiers.
    *   Moderne : Utilisez Doctrine (les requêtes préparées sont automatiques).
*   **XSS** :
    *   Templates : Échappez la sortie (`{$var|escape:'html':'UTF-8'}`).
*   **Exec/Eval** : Strictement Interdit.
*   **Vérification des Tokens** : Toutes les soumissions AJAX/Formulaire doivent vérifier un token de sécurité.

---

## 🔍 5. Validation du Code, Revue & Contrôle Qualité

Utilisez ces outils pour vous assurer que votre module respecte les standards définis ci-dessus.

### 5.1 Outils Automatisés

1.  **PHP-CS-Fixer** : Appliquer les standards de codage PrestaShop.
    *   *Usage* : `vendor/bin/php-cs-fixer fix src/`
    *   *Config* : Utilisez la [Config PrestaShop CS Fixer](https://github.com/PrestaShop/php-dev-tools) officielle.

2.  **PHPStan** : Analyse statique pour trouver les erreurs de type.
    *   *Usage* : `vendor/bin/phpstan analyse src/ --level=5`
    *   *Module* : Utilisez `prestashop/php-dev-tools` qui fournit une extension PHPStan pour PrestaShop.

3.  **PrestaShop Validator** :
    *   Uploadez votre zip sur [validator.prestashop.com](https://validator.prestashop.com) (mentionné dans la doc).

### 5.2 Checklist de Revue Manuelle

- [ ] **Structure** : La logique est-elle séparée du Contrôleur ?
- [ ] **CQRS** : Les Commandes sont-elles immuables ? Les Handlers retournent-ils `void` (pour les commandes) ?
- [ ] **Sécurité** : Tous les `Tools::getValue()` sont-ils castés ou sanitisés ?
- [ ] **Performance** :
    - [ ] Pas de requêtes dans des boucles (problème N+1).
    - [ ] Assets JS/CSS ajoutés uniquement sur les pages pertinentes (pas de hook global).
- [ ] **Propreté** : Pas de code commenté, pas de `var_dump()` oublié.
