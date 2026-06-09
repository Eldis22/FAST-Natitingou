# Plateforme Web de Gestion Académique — FAST Natitingou

> Application web full-stack de gestion des activités académiques de la Faculté des Sciences et Techniques de Natitingou

---

## Liens de déploiement

| Composant | URL |
|-----------|-----|
| **Frontend** | https://eldis22.github.io/FAST-Natitingou/ |
| **Backend API** | https://eldis229-fast-nati.hf.space/ |
| **Documentation API (Swagger)** | https://eldis229-fast-nati.hf.space/docs |
| **Code source Backend** | https://huggingface.co/spaces/Eldis229/fast-nati/tree/main |

---

## Présentation

La plateforme centralise la gestion académique autour de **trois profils** :

| Profil | Fonctionnalités |
|--------|----------------|
| **Étudiant** | Consultation des notes, relevé de notes (export PDF), emploi du temps |
| **Enseignant** | Saisie et modification des notes, liste des étudiants par matière |
| **Administrateur** | Gestion des utilisateurs, matières, emplois du temps, validation des relevés |

---

## Architecture

```
┌─────────────────────┐     HTTPS + JWT      ┌──────────────────────┐     Supabase Client     ┌─────────────────────┐
│  Frontend Statique  │ ───────────────────► │  Backend FastAPI     │ ──────────────────────► │  PostgreSQL         │
│  (GitHub Pages)     │                       │  (Hugging Face)      │                          │  (Supabase Cloud)   │
│  HTML / CSS / JS    │ ◄─────────────────── │  Python 3.10         │ ◄────────────────────── │                     │
└─────────────────────┘       JSON            └──────────────────────┘         Réponse          └─────────────────────┘
                                                        │
                                                        │ API Brevo (SMTP)
                                                        ▼
                                                Emails transactionnels
```

---

## Stack technique

- **Backend** : [FastAPI](https://fastapi.tiangolo.com/) (Python 3.10), Uvicorn, Pydantic v2
- **Authentification** : JWT (python-jose), hachage bcrypt (passlib)
- **Base de données** : PostgreSQL via [Supabase](https://supabase.com/)
- **Emails** : [Brevo](https://www.brevo.com/) (anciennement Sendinblue)
- **Frontend** : HTML5, CSS3, JavaScript
- **Déploiement Backend** : Docker + [Hugging Face Spaces](https://huggingface.co/spaces)
- **Déploiement Frontend** : GitHub Pages

---

## Structure du projet

```
PROJET/
├── main.py                     # Point d'entrée FastAPI
├── app.py                      # Alias pour le démarrage HuggingFace
├── Dockerfile                  # Image Docker Python 3.10
├── requirements.txt            # Dépendances Python
├── PostgreSQL.txt             # Script SQL de création des tables
├── .env.example                      # Exemple de variables d'environnement (ne pas committer)
├── unstim.png                  # Logo UNSTIM
│
├── src/
│   ├── config.py               # Paramètres via pydantic-settings
│   ├── database.py             # Client Supabase
│   ├── security.py             # JWT, bcrypt, RoleChecker
│   ├── schemas.py              # Modèles Pydantic (entrée/sortie)
│   └── routers/
│       ├── auth.py             # /auth — connexion, reset mot de passe
│       ├── admin.py            # /admin — utilisateurs, matières, emplois du temps, notes
│       ├── enseignant.py       # /enseignant — saisie notes
│       ├── etudiant.py         # /etudiant — notes, relevé, emplois du temps
│       └── notifications.py    # /notifications — CRUD + lecture
│
├── index.html                  # Page de connexion
├── dashboard_admin.html        # Dashboard Administrateur
├── dashboard_enseignant.html   # Dashboard Enseignant
└── dashboard_etudiant.html     # Dashboard Étudiant
```

---

## Variables d'environnement

Créez un fichier `.env` à la racine du projet backend (ou configurez les Secrets sur Hugging Face Spaces) :

```env
# Base de données Supabase
SUPABASE_URL=https://xxxxxxxxxxxx.supabase.co
SUPABASE_KEY=votre-cle-service-role-ou-anon-public

# JWT — générez avec : python -c "import secrets; print(secrets.token_hex(32))"
JWT_SECRET=un_secret_tres_long_et_totalement_aleatoire_pour_la_securite
JWT_ALGORITHM=HS256

# Brevo (envoi d'emails)
BREVO_API_KEY=votre-cle-api-sur-brevo
BREVO_SENDER_EMAIL=votre-email-expeditrice
BREVO_SENDER_NAME=FAST Natitingou

# CORS — URL du frontend (laisser vide pour autoriser toutes origines)
FRONTEND_URL=https://eldis22.github.io
```

> **NB: Ne jamais committer le fichier `.env` dans un dépôt public.**

---

## Installation et lancement en local

### Prérequis
- Python 3.10+
- Un projet Supabase configuré (voir section Base de données)
- Un compte Brevo avec clé API

### 1. Cloner le dépôt

```bash
git clone https://huggingface.co/spaces/Eldis229/fast-nati
cd fast-nati
```

### 2. Installer les dépendances

```bash
pip install -r requirements.txt
```

### 3. Configurer les variables d'environnement

```bash
cp .env.example .env
# Éditez .env avec vos vraies valeurs
```

### 4. Lancer le serveur de développement

```bash
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

L'API est disponible sur `http://localhost:8000`  
La documentation Swagger est disponible sur `http://localhost:8000/docs`

### 5. Lancer avec Docker

```bash
docker build -t fast-nati .
docker run -p 7860:7860 --env-file .env fast-nati
```

---

## Base de données (PostgreSQL / Supabase)

### Création des tables

Connectez-vous à votre projet Supabase → **SQL Editor** et exécutez le script `PostgreSQL.txt` dans l'ordre.

Ce script crée les tables :
- `utilisateurs` — comptes admin, enseignants, étudiants
- `matieres` — catalogue des matières
- `notes` — notes des étudiants (avec contrainte d'unicité)
- `emplois_du_temps` — créneaux hebdomadaires par niveau
- `validation_emplois` — historique de validation des emplois du temps
- `notifications` — annonces par filière
- `notifications_lues` / `notifications_supprimees` — suivi de lecture

### Compte administrateur par défaut

Le script SQL insère un compte admin de démarrage :

```
Email    : admin@fast.unstim.bj
Mot de passe : (voir script — hash bcrypt inclus)
Matricule : ADM-2026-001
```

> Facultatif: Changez le mot de passe après la première connexion.

---

## Sécurité

- Mots de passe hachés avec **bcrypt** (jamais stockés en clair)
- Authentification par **JWT HS256** (token transmis en en-tête `Authorization: Bearer`)
- Contrôle d'accès **RBAC** (Role-Based Access Control) par dépendance FastAPI
- Réinitialisation de mot de passe par **code numérique 6 chiffres** valable 15 minutes, envoyé par email
- CORS configuré pour limiter les origines autorisées en production

---

## Dépendances principales

```
fastapi
uvicorn
supabase
passlib
bcrypt==4.1.2
python-jose[cryptography]
python-multipart
pydantic[email]
pydantic-settings
httpx
```

---

## Fonctionnalité email (Brevo)

Deux types d'emails sont envoyés automatiquement :

| Déclencheur | Contenu |
|-------------|---------|
| Création d'un compte | Email de bienvenue avec identifiants de connexion |
| Mot de passe oublié | Code de vérification à 6 chiffres (validité 15 min) |

---

## Licence

Projet académique réalisé dans le cadre d'un stage à la FAST Natitingou.  
© 2026 — Tous droits réservés.