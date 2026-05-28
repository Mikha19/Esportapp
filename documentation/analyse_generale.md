# 📊 Analyse - GameConnect

## 🔍 Analyse des besoins

### Besoins fonctionnels identifiés

#### 1. Gestion des utilisateurs
- Authentification sécurisée (JWT)
- Profils gaming détaillés
- Historique matchmaking
- Préférences utilisateur

#### 2. Système de matching intelligent
- Appariement par jeux en commun
- Compatibilité niveau compétence
- Fuseau horaire/localisation
- Score de compatibilité

#### 3. Communication
- Messagerie directe P2P
- Forums communautaires
- Notifications temps réel
- Historique de messages

#### 4. Gestion des jeux
- Catalogue jeux supportés
- Profils gaming utilisateurs
- Statistiques per-game
- Rangs/niveaux

### Besoins non-fonctionnels

| Besoin | Niveau requis |
|--------|---------------|
| Performance | < 100ms réponse API |
| Disponibilité | 99.5% uptime |
| Sécurité | OWASP Top 10 mitigé |
| Scalabilité | 100k+ users |
| Modifiabilité | Code maintenable |
| Testabilité | > 80% couverture tests |

---

## 🏗️ Diagrammes UML

### 1. Diagramme des Cas d'Utilisation

```
┌─────────────────────────────────────────────────────────────────┐
│                      GameConnect System                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────────────┐         ┌──────────────────┐               │
│  │  Joueur          │         │  Admin           │               │
│  └────────┬─────────┘         └────────┬─────────┘               │
│           │                            │                         │
│           ├─── S'authentifier          │                         │
│           ├─── Créer profil            ├─── Modérer contenu     │
│           ├─── Ajouter jeux            ├─── Gérer catalogue     │
│           ├─── Trouver matchs    ◄─────┤                         │
│           ├─── Accepter match    ◄─────┤                         │
│           ├─── Envoyer message         │                         │
│           ├─── Voir statistiques       └─────────────────────────┤
│           └─── Participer forums                                 │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

### 2. Diagramme de classe (Entités principales)

```
┌──────────────────────┐
│     User             │
├──────────────────────┤
│ - id: UUID           │
│ - email: String      │
│ - username: String   │
│ - password_hash: Str │
│ - created_at: Date   │
├──────────────────────┤
│ + register()         │
│ + login()            │
│ + updateProfile()    │
└──────────┬───────────┘
           │ 1
           │
           ├─── * ┌──────────────────┐
           │      │   UserProfile    │
           │      ├──────────────────┤
           │      │ - bio: String    │
           │      │ - avatar: Blob   │
           │      │ - region: Str    │
           │      │ - timezone: Str  │
           │      │ - level: Enum    │
           │      └──────────────────┘
           │
           ├─── * ┌──────────────────┐
           │      │   UserGame       │
           │      ├──────────────────┤
           │      │ - rank: String   │
           │      │ - hours: Int     │
           │      │ - stats: JSON    │
           │      └──────────────────┘
           │
           └─── * ┌──────────────────┐
                  │   Message        │
                  ├──────────────────┤
                  │ - sender_id: FK  │
                  │ - recipient_id   │
                  │ - content: Text  │
                  │ - created_at     │
                  │ - read_at: Date  │
                  └──────────────────┘

┌──────────────────────┐
│   Game               │
├──────────────────────┤
│ - id: UUID           │
│ - name: String       │
│ - genre: Enum        │
│ - cover: Blob        │
│ - description: Text  │
└──────────────────────┘

┌──────────────────────┐
│   Match              │
├──────────────────────┤
│ - id: UUID           │
│ - user1_id: FK       │
│ - user2_id: FK       │
│ - status: Enum       │
│ - compatibility: Num │
│ - created_at: Date   │
│ - common_games: JSON │
└──────────────────────┘
```

---

## 📋 Règles métier

### Authentification
- **RM-001** : Mot de passe doit avoir ≥8 caractères avec 1 majuscule, 1 minuscule, 1 chiffre
- **RM-002** : JWT token valide 7 jours
- **RM-003** : Email doit être unique dans le système
- **RM-004** : Session invalide si token absent ou expiré

### Profil utilisateur
- **RM-010** : Username doit être unique et alphanumérique (3-20 caractères)
- **RM-011** : Bio limitée à 500 caractères
- **RM-012** : Avatar doit être JPG/PNG < 5MB
- **RM-013** : Utilisateur ne peut voir que profils publics (sauf amis)

### Jeux et compétences
- **RM-020** : Max 20 jeux par utilisateur
- **RM-021** : Chaque jeu a niveaux: Débutant/Intermédiaire/Avancé/Pro
- **RM-022** : Temps de jeu cumule depuis première ajout
- **RM-023** : Suppression jeu ne supprime pas historique matchs

### Matching
- **RM-030** : Match nécessite ≥1 jeu en commun
- **RM-031** : Fuseau horaire dans même région (±3h)
- **RM-032** : Score compatibilité = (jeux communs + proximité niveau) / 2
- **RM-033** : Même profil ne peut pas réapparaître en résultats pendant 30 jours après rejet
- **RM-034** : Match bilatéral = acceptation mutuelle
- **RM-035** : Utilisateur peut "défaire" un match à tout moment

### Communication
- **RM-040** : Message limité 1000 caractères
- **RM-041** : Conversation effacée = messages effacés pour l'utilisateur seulement
- **RM-042** : Blocage d'utilisateur empêche envoi/réception messages

### Modération
- **RM-050** : Contenu signalé examiné dans 24h
- **RM-051** : Utilisateur bannis ne peut pas se reconnecter
- **RM-052** : Les admins peuvent voir tous les messages à titre de modération

---

## 🛠️ Justification des choix technologiques

### Backend : Python FastAPI

**Avantages** :
✅ Développement rapide avec TypeHints  
✅ Framework moderne et performant  
✅ Async/Await intégré (notifications temps réel)  
✅ Documentation Swagger auto-générée  
✅ Excellent écosystème (SQLAlchemy, Pydantic)  

**Alternatives rejetées** :
- Node.js/Express : Moins type-safe
- Django : Plus lourd pour une API simple
- PHP Laravel : Moins adapté à l'async

### Base de données : MySQL 8.0

**Avantages** :
✅ Relatif et normalisé (perfect pour notre modèle)  
✅ ACID transactions (matchs bilatéraux)  
✅ Indices performants (queries matching complexes)  
✅ MariaDB compatible (portabilité)  

**Alternatives rejetées** :
- PostgreSQL : Overkill pour phase 1
- MongoDB : Les relations complexes nécessitent SQL
- Redis : Complément, pas remplacement

### Frontend : React 18 + Vite

**Avantages** :
✅ React dominance + grande communauté  
✅ Composants réutilisables (jeux, matchs, messages)  
✅ Vite build ultra-fast (feedback développeur)  
✅ Hooks modernes + Context API suffisant (pas Redux phase 1)  

**Alternatives rejetées** :
- Vue.js : Moins d'écosystème
- Angular : Trop lourd pour MVP
- Svelte : Trop jeune/risqué pour équipe

### Authentification : JWT (JSON Web Tokens)

**Avantages** :
✅ Stateless (scalabilité horizontale)  
✅ Sécurisé avec HS256 signing  
✅ Standard industrie  
✅ Facile intégration mobile future  

**Alternatives rejetées** :
- Sessions PHP : État serveur (problème scalabilité)
- OAuth2 : Surcharge pour MVP interne

### CSS Framework : Tailwind CSS

**Avantages** :
✅ Productivité rapide  
✅ Design system consistent  
✅ Très performant post-build  
✅ Responsive par défaut  

**Alternatives rejetées** :
- Bootstrap : Trop lourd, boilerplate CSS
- Material-UI : Plus pour Material Design

### Déploiement : Docker + Cloud

**Avantages** :
✅ Environnement identique dev/prod  
✅ Scalabilité horizontale facile  
✅ CI/CD simplifié  
✅ Infrastructure-as-code  

---

## 🔄 Flux métier principal

### Flux Matching

```
1. Utilisateur clique "Trouver des matchs"
   ↓
2. API récupère profils potentiels:
   - Jeux en commun: ≥1
   - Fuseau horaire: ±3h
   - Niveau compétence: ±1 niveau
   - Non rejetés 30j derniers
   - Non déjà matchés
   ↓
3. Calcul score compatibilité
   ↓
4. Affichage top 5 profils triés par score
   ↓
5. Utilisateur accepte ou rejette chacun
   ↓
6. Si acceptation:
   - Notification envoyée à destinataire
   - Passe à "Matchs en attente"
   ↓
7. Destinataire accepte = Match confirmé
```

### Flux Messagerie

```
1. Match confirmé visible en "Mes matchs"
   ↓
2. Clic "Envoyer message"
   ↓
3. Zone chat ouverte
   ↓
4. Utilisateur type + envoie message
   ↓
5. Message stocké BD
   ↓
6. Destinataire reçoit notification
   ↓
7. Historique persistent consultable
```

---

## 📊 Volume de données estimé

| Entité | Phase 1 | Phase 2 | Notes |
|--------|---------|---------|-------|
| Users | 1 000 | 50 000 | Croissance 50x |
| Messages | 50 000 | 2M+ | Très volumétrique |
| Matchs | 10 000 | 500k+ | Réduction avec temps |
| Jeux | 50 | 200+ | Ajout progressif |

### Stratégies d'optimisation nécessaires
- Pagination messages (25 par page)
- Indices sur user_id, created_at
- Archivage conversations >6 mois
- Cache Redis pour catalogue jeux
- Compression images avatar

---

## 🔐 Analyse des risques techniques

| Risque | Probabilité | Mitigation |
|--------|------------|-----------|
| Bottleneck DB avec + utilisateurs | Moyen | Read replicas, sharding |
| Notifications pas temps réel | Faible | WebSocket fallback |
| Matching trop lent | Moyen | Cache profils, algos optimisés |
| Sécurité JWT compromis | Faible | Rotation secrèts, audit |
| Performance CSS/JS | Moyen | Code splitting, lazy loading |

---

**Version** : 1.0  
**Dernière mise à jour** : Mai 2026  
**Status** : ✅ Approuvé
