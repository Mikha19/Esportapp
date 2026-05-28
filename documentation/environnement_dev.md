# 🛠️ Réalisation - GameConnect

## 🖥️ Environnement de développement

### Prérequis système

| Composant | Version | Notes |
|-----------|---------|-------|
| **Python** | 3.10+ | Backend principal |
| **Node.js** | 18+ | Frontend build tools |
| **MySQL** | 8.0+ | Base de données |
| **Git** | 2.35+ | Contrôle de version |
| **Docker** | 20.10+ | Containerisation (optionnel) |
| **VS Code** | Latest | Éditeur recommandé |

### Installation locale - Windows/Mac/Linux

#### 1. Cloner le dépôt

```bash
git clone https://github.com/your-org/gameconnect.git
cd gameconnect
```

#### 2. Configuration Backend (Python FastAPI)

```bash
# Naviguer dans dossier API
cd API

# Créer environnement virtuel
python -m venv venv

# Activer l'environnement
# Windows:
venv\Scripts\activate
# Mac/Linux:
source venv/bin/activate

# Installer dépendances
pip install -r requirements.txt

# Créer fichier .env
cp .env.example .env

# Remplir les variables d'environnement:
# DB_HOST=localhost
# DB_USER=root
# DB_PASS=your_password
# DB_NAME=esport_social
# JWT_SECRET=your-super-secret-key
# CORS_ORIGIN=http://localhost:5173
```

#### 3. Configuration Base de données

```bash
# Créer base de données
mysql -u root -p -e "CREATE DATABASE esport_social CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# Importer schéma
mysql -u root -p esport_social < database.sql

# Importer données test (optionnel)
mysql -u root -p esport_social < test_data.sql
```

#### 4. Lancer le serveur backend

```bash
# Option 1: Direct (pour développement)
cd API
python app/main.py

# Option 2: Avec Uvicorn (recommandé)
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Docs API disponibles à: http://localhost:8000/docs
```

#### 5. Configuration Frontend (React + Vite)

```bash
# Depuis root du projet
cd frontend

# Installer dépendances
npm install

# Créer fichier .env.local
echo "VITE_API_URL=http://localhost:8000/api" > .env.local

# Lancer serveur de développement
npm run dev

# Frontend disponible à: http://localhost:5173
```

### Dossiers clés

```
gameconnect/
├── API/                          # Backend FastAPI
│   ├── app/
│   │   ├── main.py              # Entry point
│   │   ├── config.py            # Configuration
│   │   ├── database.py          # Connexion DB
│   │   ├── models/              # SQLAlchemy ORM
│   │   ├── routes/              # API endpoints
│   │   ├── services/            # Business logic
│   │   ├── middleware/          # Auth, CORS, etc
│   │   └── schemas/             # Pydantic models
│   ├── tests/
│   ├── requirements.txt
│   ├── .env.example
│   └── database.sql
│
├── frontend/                     # Frontend React
│   ├── src/
│   │   ├── main.jsx             # Entry point
│   │   ├── App.jsx              # Root component
│   │   ├── components/
│   │   ├── pages/
│   │   ├── services/            # API calls
│   │   ├── contexts/            # State management
│   │   ├── hooks/
│   │   └── index.css            # Global styles
│   ├── public/                   # Assets statiques
│   ├── package.json
│   ├── vite.config.js
│   ├── tailwind.config.js
│   └── .env.example
│
├── documentation/               # Ce dossier
└── localsetup/                  # Scripts d'installation rapide
    ├── database.sql
    ├── test_data.sql
    ├── start.sh                 # Lancer tout en local
    └── quick-setup.sh           # Setup rapide
```

### Variables d'environnement

#### Backend (.env)

```env
# Database
DB_HOST=localhost
DB_USER=root
DB_PASS=your_mysql_password
DB_NAME=esport_social
DB_PORT=3306

# Security
JWT_SECRET=your-secret-key-minimum-32-characters
JWT_ALGORITHM=HS256
JWT_EXPIRE_HOURS=168  # 7 days

# Server
API_HOST=0.0.0.0
API_PORT=8000
DEBUG=False

# CORS
CORS_ORIGIN=http://localhost:5173

# Email (optional)
SMTP_SERVER=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your_email@gmail.com
SMTP_PASS=your_app_password
```

#### Frontend (.env.local)

```env
VITE_API_URL=http://localhost:8000/api
VITE_APP_NAME=GameConnect
VITE_APP_VERSION=1.0.0
```

### Commandes courantes

```bash
# Backend - Lint & format
cd API
black app/        # Formatter
flake8 app/       # Lint
mypy app/         # Type checking

# Backend - Tests
pytest tests/
pytest tests/ --cov  # Avec couverture

# Backend - Migration DB (si utilise Alembic)
alembic upgrade head

# Frontend - Lint & format
cd frontend
npm run lint      # ESLint
npm run format    # Prettier

# Frontend - Build production
npm run build
npm run preview   # Preview build

# Frontend - Tests
npm test
npm run test:e2e
```

---

## 🗄️ Base de Données

### Script de création schéma

Voir fichier complet: [database.sql](../../../localsetup/database.sql)

```sql
-- Résumé des tables principales
CREATE TABLE users (...);
CREATE TABLE user_profiles (...);
CREATE TABLE games (...);
CREATE TABLE user_games (...);
CREATE TABLE matches (...);
CREATE TABLE messages (...);
CREATE TABLE user_blocks (...);
CREATE TABLE match_rejections (...);
CREATE TABLE activity_logs (...);
```

### Données de test

Fichier: [test_data.sql](../../../localsetup/test_data.sql)

Contient:
- 10 utilisateurs de test
- 20 jeux populaires
- 5 matchs d'exemple
- Messages de test

### Migrations

Fichiers dans: `localsetup/migrations/`

| Fichier | Description |
|---------|-------------|
| `001_add_indexes_and_banner.sql` | Indices de performance + colonne banner |
| `002_notifications_table.sql` | Table notifications (Phase 2) |

---

## 🔄 Flux de développement Git

### Branching strategy: Git Flow

```
                    PRODUCTION
                        ↑
                    [main]
                    ↓   ↑
                 [hotfix-*]
                        ↑
                     [release-*]
                        ↑
                    [develop]
                    ↓   ↑   ↑   ↑
           [feature-*] [bug-*] [test-*]
```

### Workflow quotidien

```bash
# 1. Créer branche feature
git checkout -b feature/us-001-inscription

# 2. Développer
git add .
git commit -m "feat: Ajouter formulaire inscription"
git commit -m "test: Ajouter tests authentification"

# 3. Push
git push origin feature/us-001-inscription

# 4. Pull Request
# (Via GitHub interface)

# 5. Après merge
git checkout develop
git pull origin develop
git branch -d feature/us-001-inscription
```

### Conventions de commit

```
Format: <type>: <description>

Types:
- feat:    Nouvelle fonctionnalité
- fix:     Correction bug
- test:    Ajout tests
- docs:    Documentation
- style:   Formatage
- refactor: Restructuration

Exemples:
feat: Ajouter système de matching
fix: Corriger validation email
test: Tests unitaires AuthService
docs: Mise à jour README
```

---

## 📦 Dépendances principales

### Backend Python

```
fastapi==0.95.0           # Framework API
uvicorn==0.21.0          # ASGI server
sqlalchemy==2.0.0        # ORM
pydantic==1.10.0         # Validation données
pyjwt==2.6.0            # JWT tokens
bcrypt==4.0.0           # Password hashing
python-dotenv==0.20.0   # Env variables
mysqlclient==2.1.0      # MySQL driver
```

### Frontend JavaScript

```
react==18.2.0            # Framework UI
react-router-dom==6.0   # Routing
axios==1.4.0            # HTTP client
tailwindcss==3.3.0      # CSS framework
vite==4.0.0             # Build tool
lucide-react==0.263.0   # Icons
```

---

## 🔄 Processus d'intégration continue

### GitHub Actions workflow

```yaml
name: CI/CD Pipeline

on: [push, pull_request]

jobs:
  backend-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.10'
      - name: Install dependencies
        run: pip install -r API/requirements.txt
      - name: Lint
        run: flake8 API/app
      - name: Type check
        run: mypy API/app
      - name: Run tests
        run: pytest API/tests --cov

  frontend-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Node
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install dependencies
        run: cd frontend && npm install
      - name: Lint
        run: cd frontend && npm run lint
      - name: Build
        run: cd frontend && npm run build
      - name: Test
        run: cd frontend && npm test
```

---

## 🐳 Déploiement avec Docker

### Dockerfile Backend

```dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY API/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY API/ .

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Dockerfile Frontend

```dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY frontend/package*.json ./
RUN npm install
COPY frontend/ .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Docker Compose

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: esport_social
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
      - ./localsetup/database.sql:/docker-entrypoint-initdb.d/init.sql

  api:
    build: ./API
    ports:
      - "8000:8000"
    environment:
      DB_HOST: mysql
      DB_USER: root
      DB_PASS: root
      DB_NAME: esport_social
    depends_on:
      - mysql

  frontend:
    build: ./frontend
    ports:
      - "3000:80"
    environment:
      VITE_API_URL: http://localhost:8000/api

volumes:
  mysql_data:
```

Lancer avec:
```bash
docker-compose up -d
```

---

## 📊 Structure des répertoires complet

```
gameconnect/
├── API/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py
│   │   ├── config.py
│   │   ├── database.py
│   │   ├── models/
│   │   │   ├── __init__.py
│   │   │   ├── user.py
│   │   │   ├── game.py
│   │   │   ├── match.py
│   │   │   └── message.py
│   │   ├── routes/
│   │   │   ├── __init__.py
│   │   │   ├── auth.py
│   │   │   ├── games.py
│   │   │   ├── matching.py
│   │   │   ├── messages.py
│   │   │   ├── profile.py
│   │   │   └── admin.py
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   ├── auth.py
│   │   │   ├── matching.py
│   │   │   ├── user.py
│   │   │   └── message.py
│   │   ├── middleware/
│   │   │   ├── __init__.py
│   │   │   ├── auth.py
│   │   │   ├── cors.py
│   │   │   └── logging.py
│   │   └── schemas/
│   │       ├── __init__.py
│   │       ├── user.py
│   │       ├── game.py
│   │       ├── match.py
│   │       └── message.py
│   ├── tests/
│   │   ├── __init__.py
│   │   ├── test_auth.py
│   │   ├── test_matching.py
│   │   └── test_api.py
│   ├── requirements.txt
│   ├── .env.example
│   ├── database.sql
│   └── Dockerfile
│
├── frontend/
│   ├── src/
│   │   ├── App.jsx
│   │   ├── main.jsx
│   │   ├── index.css
│   │   ├── components/
│   │   │   ├── common/
│   │   │   ├── layout/
│   │   │   ├── auth/
│   │   │   ├── matching/
│   │   │   └── messages/
│   │   ├── pages/
│   │   │   ├── Home.jsx
│   │   │   ├── Login.jsx
│   │   │   ├── Register.jsx
│   │   │   ├── Profile.jsx
│   │   │   ├── Games.jsx
│   │   │   ├── Matching.jsx
│   │   │   ├── Messages.jsx
│   │   │   └── Dashboard.jsx
│   │   ├── services/
│   │   │   ├── api.js
│   │   │   ├── auth.js
│   │   │   ├── matching.js
│   │   │   └── messages.js
│   │   ├── contexts/
│   │   │   ├── AuthContext.jsx
│   │   │   ├── ThemeContext.jsx
│   │   │   └── ToastContext.jsx
│   │   ├── hooks/
│   │   │   └── useToast.js
│   │   └── constants/
│   │       └── imageGallery.js
│   ├── public/
│   ├── package.json
│   ├── vite.config.js
│   ├── tailwind.config.js
│   ├── .env.example
│   └── Dockerfile
│
├── documentation/
│   ├── 01_contexte/
│   ├── 02_analyse/
│   ├── 03_conception/
│   ├── 04_realisation/
│   ├── 05_tests/
│   └── 06_documentation/
│
├── localsetup/
│   ├── database.sql
│   ├── test_data.sql
│   ├── migrations/
│   ├── start.sh
│   └── quick-setup.sh
│
├── docker-compose.yml
├── .gitignore
├── README.md
└── LICENSE
```

---

## 🚀 Checklist de lancement

Avant chaque release:

- [ ] Tous les tests passent (> 80% couverture)
- [ ] Code reviewed et approuvé
- [ ] Base de données migrée
- [ ] Variables d'env configurées
- [ ] Build optimisé testé
- [ ] Documentation à jour
- [ ] Performance testée (< 100ms API)
- [ ] Sécurité auditée
- [ ] Staging deployé et validé

---

**Version** : 1.0  
**Dernière mise à jour** : Mai 2026  
**Status** : ✅ Environnement prêt
