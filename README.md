# Nsiareact
# NSIA GRC — Gestion des Réclamations Clients

Application web de gestion des réclamations clients pour **NSIA Vie Assurances Côte d'Ivoire**, développée dans le cadre d'un projet de fin d'études (Licence Professionnelle Réseaux et Génie Logiciel — PIGIER Côte d'Ivoire).

## Fonctionnalités

| Rôle | Fonctionnalités |
|---|---|
| **Client** | Inscription / connexion, soumission d'une réclamation avec pièces jointes (PDF, JPG, PNG — 5 Mo max), suivi des statuts, historique des notifications, export PDF d'une réclamation, gestion du profil |
| **Agent Spécifique** | Traite les réclamations de **son service uniquement**, changement de statut, recherche et filtres, statistiques de son service |
| **Agent Général** | Voit les réclamations de **tous les services**, changement de statut, filtre par service, statistiques globales |
| **Administrateur** | Gestion des agents, des services et des types de réclamation, liste des clients avec leurs réclamations, statistiques globales |

**Règles de gestion principales**

- Statuts : Nouvelle, En cours, En attente, Résolue, Rejetée, Clôturée
- Affectation automatique de la réclamation au service selon son type
- Délai de traitement (SLA) : **10 jours ouvrés**
- Au-delà du délai, un courrier de dépassement est envoyé au client (notification interne + email) par une commande planifiée
- Toutes les actions sont tracées dans une table d'historique (audit trail)

## Stack technique

- **Backend** : Laravel 11/12 (PHP 8.2+), Laravel Sanctum
- **Frontend** : Vue 3 + Inertia.js + Tailwind CSS v4 + Chart.js (via Vite)
- **Base de données** : PostgreSQL
- **PDF** : barryvdh/laravel-dompdf

---

## Installation sur un nouveau PC (Windows)

### 1. Logiciels à installer

| Logiciel | Version | Lien |
|---|---|---|
| Git | dernière | https://git-scm.com |
| XAMPP (fournit PHP) ou PHP seul | **PHP 8.2 ou supérieur** | https://www.apachefriends.org |
| Composer | dernière | https://getcomposer.org |
| Node.js | version LTS | https://nodejs.org |
| PostgreSQL (+ pgAdmin) | 14 ou supérieur | https://www.postgresql.org/download |
| Visual Studio Code | dernière | https://code.visualstudio.com |

### 2. Extensions PHP à activer

Ouvrez `php.ini` (avec XAMPP : `C:\xampp\php\php.ini`) et vérifiez que ces lignes **ne commencent pas par un point-virgule** (sinon retirez-le) :

```ini
extension=pdo_pgsql
extension=pgsql
extension=mbstring
extension=openssl
extension=fileinfo
extension=gd
extension=zip
extension=curl
```

Vérification rapide dans le terminal :

```bash
php -m
```

La liste doit contenir `pdo_pgsql`, `pgsql`, `gd`, `zip` et `fileinfo`. Redémarrez le terminal après toute modification de `php.ini`.

### 3. Extensions VS Code recommandées

- **PHP Intelephense** (bmewburn.vscode-intelephense-client)
- **Vue - Official** (Vue.volar)
- **Tailwind CSS IntelliSense** (bradlc.vscode-tailwindcss)
- **Laravel Blade Snippets** (onecentlin.laravel-blade)
- **DotENV** (mikestead.dotenv)

Astuce : si VS Code affiche des erreurs du type `Undefined method 'load'` sur des méthodes Eloquent, ce sont des faux positifs de l'analyseur. Faites `Ctrl+Shift+P` → *Intelephense: Index Workspace*.

### 4. Récupérer le projet

```bash
git clone <URL_DU_DEPOT_GITHUB>
cd nsia-grc
```

### 5. Installer les dépendances

```bash
composer install
npm install
```

### 6. Configurer l'environnement

Copiez le fichier d'exemple :

```bash
copy .env.example .env
```

Ouvrez `.env` et renseignez au minimum :

```env
APP_NAME="NSIA GRC"
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost:8000

DB_CONNECTION=pgsql
DB_HOST=127.0.0.1
DB_PORT=5432
DB_DATABASE=nsia_grc
DB_USERNAME=postgres
DB_PASSWORD=votre_mot_de_passe_postgres

SANCTUM_STATEFUL_DOMAINS=localhost,127.0.0.1,localhost:5173,127.0.0.1:5173
SESSION_DOMAIN=localhost

# Les emails ne sont pas réellement envoyés : ils sont écrits dans storage/logs/laravel.log
MAIL_MAILER=log
```

Générez la clé de l'application :

```bash
php artisan key:generate
```

### 7. Créer la base de données

Avec pgAdmin (clic droit sur *Databases* → *Create* → *Database*, nom : `nsia_grc`) ou en ligne de commande :

```bash
createdb -U postgres nsia_grc
```

### 8. Migrations et données de test

```bash
php artisan migrate --seed
php artisan storage:link
```

`migrate --seed` crée les 11 tables du MCD, puis insère les services, statuts, types de réclamation et 3 comptes de test.

### 9. Créer le compte administrateur

Le compte admin n'est pas créé par le seeder. Lancez :

```bash
php artisan tinker
```

puis collez :

```php
App\Models\Utilisateur::create([
    'nom_util' => 'Admin',
    'prenom_util' => 'NSIA',
    'email_util' => 'admin@test.com',
    'password' => Hash::make('password'),
    'role' => 'admin',
]);
exit
```

### 10. Lancer l'application

Ouvrez **deux terminaux** dans VS Code et laissez-les tourner :

```bash
# Terminal 1 — serveur Laravel
php artisan serve

# Terminal 2 — compilation du frontend (Vite)
npm run dev
```

Ouvrez ensuite **http://localhost:8000** (et non le port 5173).

---

## Comptes de test

| Rôle | Email | Mot de passe |
|---|---|---|
| Client | `client@test.com` | `password` |
| Agent Général | `agent.general@test.com` | `password` |
| Agent Spécifique (Service Sinistres) | `agent.sinistres@test.com` | `password` |
| Administrateur | `admin@test.com` | `password` |

> Ces comptes servent uniquement au développement. Changez les mots de passe avant tout déploiement réel.

---

## Vérification des délais (SLA)

La détection des réclamations hors délai (10 jours ouvrés) et l'envoi du courrier de dépassement se lancent avec la commande :

```bash
php artisan reclamations:verifier-delais
```

Elle est planifiée chaque jour à 08h00 (`routes/console.php`). En production, il faut ajouter la tâche cron Laravel :

```
* * * * * cd /chemin/du/projet && php artisan schedule:run >> /dev/null 2>&1
```

Pour tester sans attendre 10 jours, forcez une échéance dépassée depuis `php artisan tinker` :

```php
App\Models\Reclamation::first()->update(['date_echeance' => now()->subDay()->toDateString()]);
```

---

## Structure du projet

```
app/
├── Http/Controllers/Api/     Contrôleurs (Auth, Reclamation, Admin, Profil, Statistique)
├── Http/Middleware/          EnsureRole (client / agent / admin), HandleInertiaRequests
├── Models/                   Modèles Eloquent (reflet du MCD)
├── Services/                 Logique métier (AffectationService, SlaService, NotificationService)
├── Mail/                     Emails (changement de statut, dépassement SLA)
└── Console/Commands/         VerifierDelaisReclamations
database/
├── migrations/               11 tables + rôle admin + Sanctum
└── seeders/                  Services, statuts, types, utilisateurs de test
resources/
├── js/Pages/                 Pages Vue (Auth, Reclamations, Agent, Admin, Profil, Notifications)
├── js/Components/            BackButton, graphiques Chart.js
├── js/Layouts/               AppLayout
└── views/                    app.blade.php, emails, modèle PDF
routes/web.php                Routes de l'application
```

---

## Problèmes fréquents

| Symptôme | Solution |
|---|---|
| Page blanche ou `Vite manifest not found` | Lancer `npm run dev` (ou `npm run build`) |
| Style Tailwind absent | Vérifier que `tailwindcss()` est bien dans `vite.config.js`, puis relancer `npm run dev` |
| Une modification Vue ne s'affiche pas | Rechargement forcé du navigateur : `Ctrl + Shift + R` |
| Erreur `relation "users" does not exist` | Vérifier que `config/auth.php` pointe vers `App\Models\Utilisateur` |
| `404` sur une route qui devrait exister | `php artisan route:clear`, puis `php artisan route:list` pour vérifier |
| Erreur `Maximum execution time exceeded` sous Windows | Ajouter le dossier du projet aux exclusions de Windows Defender |
| Erreur de connexion à la base | Vérifier que le service PostgreSQL tourne et que `.env` est correct |
| Pièces jointes inaccessibles | Lancer `php artisan storage:link` |

---

## Sécurité

- Mots de passe hachés (bcrypt), protection CSRF, ORM Eloquent (requêtes préparées)
- Middleware de rôles (client, agent général, agent spécifique, admin)
- Limitation des tentatives de connexion et d'inscription (rate limiting)
- Validation du type MIME réel et de la taille des fichiers téléversés
- Traçabilité des actions dans la table `historiques`
- Ne jamais versionner le fichier `.env` (déjà exclu par le `.gitignore` de Laravel)

---

Projet académique — NSIA Vie Assurances Côte d'Ivoire.
