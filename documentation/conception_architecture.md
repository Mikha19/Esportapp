# 🏛️ Conception - GameConnect

## 📊 Modèle Conceptuel de Données (MCD)

```
UTILISATEUR (Entité faible)
┌─────────────────────────────────────┐
│ id (PK)                             │
│ email (UNIQUE, NOT NULL)            │
│ username (UNIQUE, NOT NULL)         │
│ password_hash (NOT NULL)            │
│ created_at (TIMESTAMP)              │
│ updated_at (TIMESTAMP)              │
│ is_active (BOOLEAN)                 │
│ is_admin (BOOLEAN)                  │
└─────────────────────────────────────┘
        ↓ 1,1
        │
        ├─ Crée ─────→ PROFIL UTILISATEUR
        │               ┌──────────────────────────┐
        │               │ id (PK, FK)              │
        │               │ bio (TEXT)               │
        │               │ avatar_url (VARCHAR)     │
        │               │ region (VARCHAR)         │
        │               │ timezone (VARCHAR)       │
        │               │ skill_level (ENUM)       │
        │               │ preferred_comm (ENUM)    │
        │               │ is_public (BOOLEAN)      │
        │               └──────────────────────────┘
        │
        ├─ Possède ────→ USER_GAME (0,N)
        │               ┌──────────────────────────┐
        │               │ user_id (PK, FK)         │
        │               │ game_id (PK, FK)         │
        │               │ rank (VARCHAR)           │
        │               │ hours_played (INT)       │
        │               │ last_played (TIMESTAMP)  │
        │               │ stats_json (JSON)        │
        │               └──────────────────────────┘
        │                       ↓ N,1
        │                       │
        │                       └─→ JEU
        │                          ┌──────────────────────────┐
        │                          │ id (PK)                  │
        │                          │ name (VARCHAR, UNIQUE)   │
        │                          │ genre (ENUM)             │
        │                          │ cover_url (VARCHAR)      │
        │                          │ description (TEXT)       │
        │                          │ platform (VARCHAR)       │
        │                          │ release_date (DATE)      │
        │                          └──────────────────────────┘
        │
        ├─ Envoie (0,N) ────→ MESSAGE
        │                     ┌──────────────────────────┐
        │                     │ id (PK)                  │
        │                     │ sender_id (FK, NOT NULL) │
        │                     │ recipient_id (FK)        │
        │                     │ content (TEXT)           │
        │                     │ created_at (TIMESTAMP)   │
        │                     │ read_at (TIMESTAMP)      │
        │                     │ is_deleted (BOOLEAN)     │
        │                     └──────────────────────────┘
        │
        └─ Participe à ─→ MATCH (0,N)
                         ┌──────────────────────────┐
                         │ id (PK)                  │
                         │ user1_id (FK)            │
                         │ user2_id (FK)            │
                         │ status (ENUM)            │
                         │ compatibility (DECIMAL)  │
                         │ common_games (JSON)      │
                         │ created_at (TIMESTAMP)   │
                         │ accepted_at (TIMESTAMP)  │
                         │ ended_at (TIMESTAMP)     │
                         │ notes (TEXT)             │
                         └──────────────────────────┘

RELATION EXCEPTION:
┌────────────────────────────────┐
│ USER_BLOCK (Blocage)           │
├────────────────────────────────┤
│ blocker_id (PK, FK) → UTILISATEUR
│ blocked_id (PK, FK) → UTILISATEUR
│ created_at (TIMESTAMP)         │
└────────────────────────────────┘
```

---

## 🗄️ Modèle Logique de Données (MLD)

### Schéma relationnel SQL

```sql
-- UTILISATEURS
CREATE TABLE users (
    id CHAR(36) PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    username VARCHAR(50) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE,
    is_admin BOOLEAN DEFAULT FALSE,
    KEY idx_email (email),
    KEY idx_active (is_active)
);

-- PROFILS UTILISATEURS
CREATE TABLE user_profiles (
    user_id CHAR(36) PRIMARY KEY,
    bio TEXT,
    avatar_url VARCHAR(500),
    region VARCHAR(100),
    timezone VARCHAR(50),
    skill_level ENUM('beginner', 'intermediate', 'advanced', 'professional'),
    preferred_comm ENUM('chat', 'voice', 'video'),
    is_public BOOLEAN DEFAULT TRUE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- JEUX
CREATE TABLE games (
    id CHAR(36) PRIMARY KEY,
    name VARCHAR(255) UNIQUE NOT NULL,
    genre ENUM('FPS', 'MOBA', 'RPG', 'Battle_Royale', 'Strategy', 'Other'),
    cover_url VARCHAR(500),
    description TEXT,
    platform VARCHAR(100),
    release_date DATE,
    KEY idx_name (name),
    KEY idx_genre (genre)
);

-- JEUX UTILISATEURS
CREATE TABLE user_games (
    user_id CHAR(36) NOT NULL,
    game_id CHAR(36) NOT NULL,
    rank VARCHAR(100),
    hours_played INT DEFAULT 0,
    last_played TIMESTAMP,
    stats_json JSON,
    PRIMARY KEY (user_id, game_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (game_id) REFERENCES games(id) ON DELETE CASCADE,
    KEY idx_hours (hours_played),
    KEY idx_last_played (last_played)
);

-- MATCHS
CREATE TABLE matches (
    id CHAR(36) PRIMARY KEY,
    user1_id CHAR(36) NOT NULL,
    user2_id CHAR(36) NOT NULL,
    status ENUM('pending', 'accepted', 'rejected', 'ended', 'cancelled') DEFAULT 'pending',
    compatibility DECIMAL(5,2),
    common_games JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    accepted_at TIMESTAMP NULL,
    ended_at TIMESTAMP NULL,
    notes TEXT,
    FOREIGN KEY (user1_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (user2_id) REFERENCES users(id) ON DELETE CASCADE,
    KEY idx_user1 (user1_id),
    KEY idx_user2 (user2_id),
    KEY idx_status (status),
    KEY idx_created (created_at),
    UNIQUE KEY unique_match (user1_id, user2_id) -- Un seul match par paire
);

-- MESSAGES
CREATE TABLE messages (
    id CHAR(36) PRIMARY KEY,
    sender_id CHAR(36) NOT NULL,
    recipient_id CHAR(36) NOT NULL,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    read_at TIMESTAMP NULL,
    is_deleted BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (sender_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (recipient_id) REFERENCES users(id) ON DELETE CASCADE,
    KEY idx_recipient (recipient_id),
    KEY idx_created (created_at),
    KEY idx_read (read_at)
);

-- BLOCAGES
CREATE TABLE user_blocks (
    blocker_id CHAR(36) NOT NULL,
    blocked_id CHAR(36) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (blocker_id, blocked_id),
    FOREIGN KEY (blocker_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (blocked_id) REFERENCES users(id) ON DELETE CASCADE,
    KEY idx_blocked (blocked_id)
);

-- REJETS (pour éviter réapparition 30j)
CREATE TABLE match_rejections (
    rejector_id CHAR(36) NOT NULL,
    rejected_id CHAR(36) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP,
    PRIMARY KEY (rejector_id, rejected_id),
    FOREIGN KEY (rejector_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (rejected_id) REFERENCES users(id) ON DELETE CASCADE,
    KEY idx_expires (expires_at)
);

-- LOGS D'ACTIVITÉ
CREATE TABLE activity_logs (
    id CHAR(36) PRIMARY KEY,
    user_id CHAR(36) NOT NULL,
    action VARCHAR(100),
    details JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    KEY idx_user (user_id),
    KEY idx_created (created_at)
);
```

---

## 🏗️ Architecture de l'Application

### Architecture en 3 couches

```
┌─────────────────────────────────────────────┐
│           COUCHE PRÉSENTATION                │
│    (React Frontend - SPA)                    │
│  ├─ Components (Vite)                        │
│  ├─ Pages                                    │
│  ├─ Services API (Axios)                     │
│  ├─ Contexts (Auth, Theme, Toast)            │
│  └─ Hooks personnalisés                      │
└──────────────────┬──────────────────────────┘
                   │
           HTTP/HTTPS (API REST)
                   │
┌──────────────────┴──────────────────────────┐
│          COUCHE MÉTIER (FastAPI)             │
│                                              │
│  ├─ Routes (Endpoints)                       │
│  │  ├─ /auth (register, login)               │
│  │  ├─ /profile                              │
│  │  ├─ /games                                │
│  │  ├─ /matches                              │
│  │  ├─ /messages                             │
│  │  └─ /admin                                │
│  │                                           │
│  ├─ Services (Logique métier)                │
│  │  ├─ MatchingService                       │
│  │  ├─ AuthService                           │
│  │  ├─ MessageService                        │
│  │  └─ UserService                           │
│  │                                           │
│  ├─ Models (Pydantic)                        │
│  │  ├─ UserRegister                          │
│  │  ├─ UserProfile                           │
│  │  ├─ Match                                 │
│  │  ├─ Message                               │
│  │  └─ Game                                  │
│  │                                           │
│  ├─ Middleware                               │
│  │  ├─ JWT Authentication                    │
│  │  ├─ CORS                                  │
│  │  ├─ ErrorHandler                          │
│  │  └─ Activity Logger                       │
│  │                                           │
│  └─ Utils                                    │
│     ├─ JWT handler                           │
│     ├─ Password hashing                      │
│     ├─ Email service                         │
│     └─ Validators                            │
└──────────────────┬──────────────────────────┘
                   │
        TCP/IP (Connection)
                   │
┌──────────────────┴──────────────────────────┐
│         COUCHE PERSISTANCE                   │
│      (MySQL + SQLAlchemy ORM)                │
│                                              │
│  ├─ Database Layer (SQLAlchemy)              │
│  ├─ Connection Pool                          │
│  ├─ Transactions                             │
│  │                                           │
│  └─ MySQL Database                           │
│     ├─ users                                 │
│     ├─ user_profiles                         │
│     ├─ games                                 │
│     ├─ user_games                            │
│     ├─ matches                               │
│     ├─ messages                              │
│     └─ activity_logs                         │
└──────────────────────────────────────────────┘
```

### Architecture Microservices (Future)

```
┌─────────────────────────────────────┐
│         API Gateway (Kong)           │
│   - Rate limiting                    │
│   - Load balancing                   │
│   - Authentication                   │
└──────────────┬──────────────────────┘
               │
    ┌──────────┼──────────┬──────────┐
    ▼          ▼          ▼          ▼
┌─────────┐┌─────────┐┌────────┐┌──────────┐
│Auth MS  ││Profile  ││Matching││Messaging │
│Service  ││Service  ││Service ││Service   │
└─────────┘└─────────┘└────────┘└──────────┘
    │          │          │          │
    └──────────┼──────────┼──────────┘
               │
        ┌──────┴──────┐
        ▼             ▼
    ┌─────────┐  ┌──────────┐
    │ MySQL   │  │  Redis   │
    │ DB      │  │ Cache    │
    └─────────┘  └──────────┘
```

---

## 🔄 Diagrammes de Séquences

### Flux d'authentification

```
Utilisateur          Frontend                API              Database
    │                  │                      │                  │
    │ Clique login      │                      │                  │
    ├─────────────────►│                      │                  │
    │                  │ POST /login           │                  │
    │                  ├─────────────────────►│                  │
    │                  │                      │ Récupère user     │
    │                  │                      ├────────────────►│
    │                  │                      │◄────────────────┤
    │                  │                      │ Valide password   │
    │                  │                      │ Génère JWT token  │
    │                  │ {token, user}        │                  │
    │                  │◄─────────────────────┤                  │
    │ Reçoit token     │                      │                  │
    │◄─────────────────┤                      │                  │
    │ Stocke localStorage
```

### Flux de matching

```
User1              Frontend              API              Database            User2
  │                   │                  │                   │               │
  │ Cherche matchs    │                  │                   │               │
  ├──────────────────►│                  │                   │               │
  │                   │ GET /matches/find │                  │               │
  │                   ├─────────────────►│                  │               │
  │                   │                  │ Récupère profils  │               │
  │                   │                  ├────────────────►│               │
  │                   │                  │◄────────────────┤               │
  │                   │                  │ Calcul score      │               │
  │                   │ [5 profils]       │                  │               │
  │                   │◄─────────────────┤                  │               │
  │ Accepte User2     │                  │                  │               │
  ├──────────────────►│                  │                  │               │
  │                   │ POST /matches     │                  │               │
  │                   ├─────────────────►│ Crée match        │               │
  │                   │                  ├────────────────►│               │
  │                   │ {match_id}        │◄────────────────┤               │
  │                   │◄─────────────────┤                  │               │
  │                   │                  │ Notification ────────────────────►│
  │                   │                  │                   │ Match reçu   │
  │                   │                  │                   │◄─────────────┤
  │                   │                  │                   │ Accepte      │
  │                   │                  │◄────────────────┤               │
  │                   │ Notification      │                  │               │
  │◄──────────────────┤                  │                  │               │
  │ Vous êtes matchés!
```

---

## 🎯 Patterns de conception utilisés

### 1. **MVC** (Model-View-Controller)
- **Models** : Pydantic schemas + SQLAlchemy ORM
- **Views** : React components
- **Controllers** : FastAPI route handlers

### 2. **Repository Pattern**
```python
class UserRepository:
    def get_by_id(self, user_id) -> User
    def get_by_email(self, email) -> User
    def create(self, user_data) -> User
    def update(self, user_id, data) -> User
    def delete(self, user_id) -> bool
```

### 3. **Service Layer Pattern**
```python
class AuthService:
    def register(self, user_data) -> User
    def login(self, email, password) -> str  # returns JWT
    def verify_token(self, token) -> User

class MatchingService:
    def find_matches(self, user_id) -> List[MatchCandidate]
    def calculate_compatibility(self, user1, user2) -> float
    def create_match(self, user1_id, user2_id) -> Match
```

### 4. **Dependency Injection**
```python
@app.get("/profile")
async def get_profile(
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    pass
```

### 5. **JWT Token Pattern**
```python
def create_jwt_token(data: dict, expires_in: timedelta) -> str
def verify_jwt_token(token: str) -> dict
```

---

## 📐 Maquette Interface (Wireframe)

### Page d'accueil - Dashboard

```
┌───────────────────────────────────────────────────────────────┐
│  Logo        🔔 📧 👤                                          │
├───────────────────────────────────────────────────────────────┤
│                                                                │
│  Bienvenue, Marcus! 👋                                         │
│                                                                │
│  ┌─────────────────────┐  ┌─────────────────────┐             │
│  │ 🎮 Jeux (3)         │  │ ⚡ Matchs en attente │             │
│  │ League of Legends   │  │ Sarah - 85%         │             │
│  │ Valorant            │  │ Matcher - 72%       │             │
│  │ CS:GO               │  │                     │             │
│  └─────────────────────┘  └─────────────────────┘             │
│                                                                │
│  ┌──────────────────────────────────────────────────────┐    │
│  │ 🤝 TROUVER DES MATCHS                 [CTA Button]   │    │
│  │                                                       │    │
│  │  Profil 1        Profil 2        Profil 3          │    │
│  │  Avatar           Avatar          Avatar            │    │
│  │  Sophie          Tom              Emma              │    │
│  │  FPS/MOBA         RPG/MOBA         Casual            │    │
│  │  Score: 78%      Score: 65%       Score: 92% ✓      │    │
│  │  [❌ Rejeter]     [❌ Rejeter]      [✅ Accepter]    │    │
│  │                                                       │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                                │
│  ┌─────────────────────┐  ┌─────────────────────┐             │
│  │ 💬 Conversations (2) │  │ 📊 Mes statistiques │             │
│  │ Sarah - "Dispo?"    │  │ Matchs: 5           │             │
│  │ Tom - "GG!"         │  │ Messages: 23        │             │
│  └─────────────────────┘  └─────────────────────┘             │
│                                                                │
└───────────────────────────────────────────────────────────────┘
```

### Page Messagerie

```
┌───────────────────────────────────────────────────────────────┐
│  Logo        🔔 📧 👤                                          │
├───────────┬───────────────────────────────────────────────────┤
│ Contacts  │  Sarah - Valorant                                 │
│           ├───────────────────────────────────────────────────┤
│ Sarah ✓   │                                                    │
│ Tom       │  Sarah (14:22)                                    │
│ Emma      │  "T'es dispo pour un match?"                      │
│ Alice     │                                                    │
│ Bob       │  Vous (14:25)                                     │
│           │  "Oui! Rank? 💪"                                  │
│           │                                                    │
│           │  Sarah (14:26)                                    │
│           │  "Gold 1, toi?"                                   │
│           │                                                    │
│           │  Vous (14:27)                                     │
│           │  "Plat 2, on peut combo!"                         │
│           │                                                    │
│           │  ┌─────────────────────────────────────────────┐ │
│           │  │ Tapez votre message...             [Envoyer] │ │
│           │  └─────────────────────────────────────────────┘ │
│           │                                                    │
│           │                                                    │
│           │  Avatar Sarah                                     │
│           │  Sarah (Gold 1)                                   │
│           │  Valorant • On line                               │
│           │                                                    │
└───────────┴───────────────────────────────────────────────────┘
```

---

## 🔐 Architecture de sécurité

```
Client Request
    │
    ├─► CORS Middleware
    │   ├─ Vérifier origine
    │   └─ Ajouter headers
    │
    ├─► Rate Limiting
    │   ├─ Limiter par IP
    │   └─ Limiter par utilisateur
    │
    ├─► JWT Validation
    │   ├─ Token présent?
    │   ├─ Signature valide?
    │   └─ Expiré?
    │
    ├─► Request Validation
    │   ├─ Format JSON?
    │   ├─ Champs requis?
    │   └─ Types corrects?
    │
    ├─► Business Logic
    │   ├─ Autorisé pour cette action?
    │   └─ Données valides?
    │
    └─► Response Security
        ├─ Filtrer données sensibles
        ├─ HTTPS only
        └─ Headers sécurité
```

---

**Version** : 1.0  
**Dernière mise à jour** : Mai 2026  
**Status** : ✅ Approuvé par tech lead
