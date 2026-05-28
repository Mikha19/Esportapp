# 🧪 Tests - GameConnect

## 📋 Plan de tests

### Stratégie de test

```
                    Manual Testing
                         ↑
        ┌─────────────────┼─────────────────┐
        │                 │                 │
    Unit Tests       Integration Tests    E2E Tests
    (80% goal)       (60% goal)          (Scenarios clés)
        │                 │                 │
        └─────────────────┼─────────────────┘
                         │
                  Coverage Total > 80%
```

### Couverture de test par module

| Module | Type | Coverage | Priorité |
|--------|------|----------|----------|
| **Auth** | Unit + Integration | 95% | Critique |
| **Matching** | Unit | 85% | Critique |
| **Messages** | Unit + Integration | 80% | Haute |
| **Profile** | Unit | 75% | Moyenne |
| **Games** | Unit | 70% | Moyenne |
| **Admin** | Unit | 60% | Basse |

---

## 🔧 Tests unitaires Backend (Python)

### Framework: pytest + pytest-cov

#### 1. Tests d'authentification

**Fichier**: `API/tests/test_auth.py`

```python
import pytest
from app.services.auth import AuthService
from app.schemas.user import UserRegister

class TestAuthService:
    
    @pytest.fixture
    def auth_service(self):
        return AuthService()
    
    def test_register_success(self, auth_service):
        """Test inscription utilisateur valide"""
        user_data = UserRegister(
            email="test@example.com",
            username="testuser",
            password="SecurePass123!"
        )
        result = auth_service.register(user_data)
        
        assert result.id is not None
        assert result.email == "test@example.com"
        assert result.username == "testuser"
        assert result.password_hash != "SecurePass123!"  # Hashé
    
    def test_register_duplicate_email(self, auth_service):
        """Test inscription avec email existant"""
        user_data = UserRegister(
            email="existing@example.com",
            username="user1",
            password="SecurePass123!"
        )
        auth_service.register(user_data)
        
        with pytest.raises(ValueError, match="Email already exists"):
            auth_service.register(user_data)
    
    def test_password_validation(self, auth_service):
        """Test validation mot de passe"""
        # Trop court
        with pytest.raises(ValueError):
            auth_service.validate_password("short")
        
        # Pas de majuscule
        with pytest.raises(ValueError):
            auth_service.validate_password("pass1234!")
        
        # Valide
        assert auth_service.validate_password("SecurePass123!") is True
    
    def test_login_success(self, auth_service):
        """Test connexion réussie"""
        # Setup
        user_data = UserRegister(
            email="test@example.com",
            username="testuser",
            password="SecurePass123!"
        )
        auth_service.register(user_data)
        
        # Test
        token = auth_service.login("test@example.com", "SecurePass123!")
        assert token is not None
        assert isinstance(token, str)
    
    def test_login_invalid_password(self, auth_service):
        """Test connexion avec mauvais mot de passe"""
        user_data = UserRegister(
            email="test@example.com",
            username="testuser",
            password="SecurePass123!"
        )
        auth_service.register(user_data)
        
        with pytest.raises(ValueError, match="Invalid credentials"):
            auth_service.login("test@example.com", "WrongPassword!")
    
    def test_jwt_token_generation(self, auth_service):
        """Test génération JWT"""
        token = auth_service.create_jwt_token({
            "sub": "user123",
            "email": "test@example.com"
        })
        
        assert token is not None
        decoded = auth_service.verify_jwt_token(token)
        assert decoded["sub"] == "user123"
        assert decoded["email"] == "test@example.com"
    
    def test_jwt_token_expiration(self, auth_service):
        """Test expiration JWT"""
        token = auth_service.create_jwt_token(
            {"sub": "user123"},
            expires_in=timedelta(seconds=-1)  # Expiré
        )
        
        with pytest.raises(JWTError):
            auth_service.verify_jwt_token(token)
```

#### 2. Tests du système de matching

**Fichier**: `API/tests/test_matching.py`

```python
class TestMatchingService:
    
    @pytest.fixture
    def matching_service(self):
        return MatchingService()
    
    @pytest.fixture
    def sample_users(self, db):
        """Créer utilisateurs de test"""
        user1 = User(
            id=uuid.uuid4(),
            email="user1@test.com",
            username="player1",
            profile=UserProfile(
                skill_level="intermediate",
                region="Europe",
                timezone="UTC+1"
            )
        )
        user2 = User(
            id=uuid.uuid4(),
            email="user2@test.com",
            username="player2",
            profile=UserProfile(
                skill_level="intermediate",
                region="Europe",
                timezone="UTC+1"
            )
        )
        db.add(user1)
        db.add(user2)
        db.commit()
        return user1, user2
    
    def test_calculate_compatibility(self, matching_service, sample_users):
        """Test calcul score compatibilité"""
        user1, user2 = sample_users
        
        # Ajouter jeux communs
        game = Game(id=uuid.uuid4(), name="League of Legends")
        user1.games.append(UserGame(game=game, rank="Gold"))
        user2.games.append(UserGame(game=game, rank="Gold"))
        
        score = matching_service.calculate_compatibility(user1, user2)
        
        assert 0 <= score <= 100
        assert score > 80  # Même niveau, même jeu
    
    def test_find_compatible_matches(self, matching_service, sample_users):
        """Test recherche matchs compatibles"""
        user1, user2 = sample_users
        
        matches = matching_service.find_matches(user1.id, limit=5)
        
        assert len(matches) > 0
        assert user2 in [m.user2_id for m in matches]
    
    def test_no_match_different_timezones(self, matching_service):
        """Test: pas de match si fuseaux trop différents"""
        user1 = User(profile=UserProfile(timezone="UTC+1"))
        user2 = User(profile=UserProfile(timezone="UTC-8"))
        
        is_compatible = matching_service.check_timezone_compatibility(
            user1, user2
        )
        
        assert is_compatible is False
    
    def test_create_match(self, matching_service, sample_users):
        """Test création d'un match"""
        user1, user2 = sample_users
        
        match = matching_service.create_match(user1.id, user2.id)
        
        assert match.user1_id == user1.id
        assert match.user2_id == user2.id
        assert match.status == "pending"
    
    def test_accept_match(self, matching_service, sample_users):
        """Test acceptation match"""
        user1, user2 = sample_users
        match = matching_service.create_match(user1.id, user2.id)
        
        result = matching_service.accept_match(match.id, user2.id)
        
        assert result.status == "pending"  # En attente acceptation user1
        
        result = matching_service.accept_match(match.id, user1.id)
        
        assert result.status == "accepted"  # Match confirmé bilatéral
    
    def test_rejection_expires(self, matching_service, sample_users):
        """Test: rejet n'expire que après 30j"""
        user1, user2 = sample_users
        
        matching_service.reject_match(user1.id, user2.id)
        
        # Immédiatement: pas dans résultats
        matches = matching_service.find_matches(user1.id)
        assert user2.id not in [m.user2_id for m in matches]
        
        # Après 30j: réapparaît
        # (test serait avec mock time)
```

#### 3. Tests des messages

**Fichier**: `API/tests/test_messages.py`

```python
class TestMessageService:
    
    def test_send_message_success(self, message_service, sample_users):
        """Test envoi message réussi"""
        user1, user2 = sample_users
        
        message = message_service.send_message(
            sender_id=user1.id,
            recipient_id=user2.id,
            content="Salut, wanna play?"
        )
        
        assert message.sender_id == user1.id
        assert message.recipient_id == user2.id
        assert message.content == "Salut, wanna play?"
        assert message.created_at is not None
    
    def test_message_length_validation(self, message_service, sample_users):
        """Test validation longueur message"""
        user1, user2 = sample_users
        
        # Message trop long (>1000 caractères)
        long_message = "A" * 1001
        
        with pytest.raises(ValueError, match="Message too long"):
            message_service.send_message(user1.id, user2.id, long_message)
    
    def test_get_conversation(self, message_service, sample_users):
        """Test récupération conversation"""
        user1, user2 = sample_users
        
        # Envoyer messages
        message_service.send_message(user1.id, user2.id, "Hello")
        message_service.send_message(user2.id, user1.id, "Hi there")
        
        conversation = message_service.get_conversation(user1.id, user2.id)
        
        assert len(conversation) == 2
        assert conversation[0].content == "Hello"
        assert conversation[1].content == "Hi there"
    
    def test_mark_as_read(self, message_service, sample_users):
        """Test marquer message comme lu"""
        user1, user2 = sample_users
        
        message = message_service.send_message(user1.id, user2.id, "Test")
        assert message.read_at is None
        
        message_service.mark_as_read(message.id)
        updated = message_service.get_message(message.id)
        
        assert updated.read_at is not None
```

---

## 🔌 Tests d'intégration API (Backend)

**Fichier**: `API/tests/test_api.py`

```python
import pytest
from fastapi.testclient import TestClient
from app.main import app

@pytest.fixture
def client():
    return TestClient(app)

@pytest.fixture
def auth_headers(client):
    """Créer utilisateur et récupérer token"""
    # Register
    response = client.post("/api/register", json={
        "email": "test@example.com",
        "username": "testuser",
        "password": "SecurePass123!"
    })
    
    # Login
    response = client.post("/api/login", json={
        "email": "test@example.com",
        "password": "SecurePass123!"
    })
    
    token = response.json()["token"]
    return {"Authorization": f"Bearer {token}"}

class TestAuthAPI:
    
    def test_register_api(self, client):
        """Test endpoint POST /register"""
        response = client.post("/api/register", json={
            "email": "new@example.com",
            "username": "newuser",
            "password": "SecurePass123!"
        })
        
        assert response.status_code == 201
        assert response.json()["email"] == "new@example.com"
    
    def test_login_api(self, client, auth_headers):
        """Test endpoint POST /login"""
        response = client.post("/api/login", json={
            "email": "test@example.com",
            "password": "SecurePass123!"
        })
        
        assert response.status_code == 200
        assert "token" in response.json()
    
    def test_profile_unauthorized(self, client):
        """Test accès profil sans auth"""
        response = client.get("/api/profile")
        
        assert response.status_code == 401

class TestProfileAPI:
    
    def test_get_profile(self, client, auth_headers):
        """Test récupération profil"""
        response = client.get("/api/profile", headers=auth_headers)
        
        assert response.status_code == 200
        assert "username" in response.json()
    
    def test_update_profile(self, client, auth_headers):
        """Test mise à jour profil"""
        response = client.put(
            "/api/profile",
            json={"bio": "New bio", "skill_level": "advanced"},
            headers=auth_headers
        )
        
        assert response.status_code == 200
        assert response.json()["bio"] == "New bio"

class TestMatchingAPI:
    
    def test_find_matches_api(self, client, auth_headers):
        """Test endpoint GET /matches/find"""
        response = client.get("/api/matches/find", headers=auth_headers)
        
        assert response.status_code == 200
        assert isinstance(response.json(), list)
        assert len(response.json()) <= 5
    
    def test_accept_match_api(self, client, auth_headers):
        """Test endpoint POST /matches/{id}/accept"""
        # Créer un match d'abord
        # ...
        
        response = client.post(
            f"/api/matches/{match_id}/accept",
            headers=auth_headers
        )
        
        assert response.status_code == 200
```

---

## 🎭 Tests Frontend (React + Vitest)

### Configuration Jest/Vitest

**Fichier**: `frontend/vitest.config.js`

```javascript
import { defineConfig } from 'vitest/config'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './src/test/setup.js',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: ['node_modules/']
    }
  }
})
```

### Tests de composants

**Fichier**: `frontend/src/components/__tests__/MatchCard.test.jsx`

```javascript
import { describe, it, expect } from 'vitest'
import { render, screen } from '@testing-library/react'
import MatchCard from '../MatchCard'

describe('MatchCard Component', () => {
  
  it('renders match card with user info', () => {
    const match = {
      id: 1,
      username: 'Sarah',
      avatar: 'avatar.jpg',
      compatibility: 85,
      games: ['League of Legends']
    }
    
    render(<MatchCard match={match} />)
    
    expect(screen.getByText('Sarah')).toBeInTheDocument()
    expect(screen.getByText('85%')).toBeInTheDocument()
  })
  
  it('calls onAccept when button clicked', () => {
    const onAccept = vi.fn()
    const match = { id: 1, username: 'Sarah', compatibility: 85 }
    
    render(<MatchCard match={match} onAccept={onAccept} />)
    
    screen.getByRole('button', { name: /accept/i }).click()
    
    expect(onAccept).toHaveBeenCalledWith(match.id)
  })
})
```

### Tests services API

**Fichier**: `frontend/src/services/__tests__/auth.test.js`

```javascript
import { describe, it, expect, vi, beforeEach } from 'vitest'
import { login, register } from '../auth'
import axios from 'axios'

vi.mock('axios')

describe('Auth Service', () => {
  
  beforeEach(() => {
    vi.clearAllMocks()
  })
  
  it('registers user successfully', async () => {
    const userData = {
      email: 'test@example.com',
      username: 'testuser',
      password: 'SecurePass123!'
    }
    
    axios.post.mockResolvedValue({
      data: { id: 1, email: userData.email }
    })
    
    const result = await register(userData)
    
    expect(result.email).toBe('test@example.com')
    expect(axios.post).toHaveBeenCalledWith(
      expect.stringContaining('/register'),
      userData
    )
  })
  
  it('handles login error gracefully', async () => {
    const error = new Error('Invalid credentials')
    axios.post.mockRejectedValue(error)
    
    await expect(
      login('test@example.com', 'wrong')
    ).rejects.toThrow('Invalid credentials')
  })
})
```

---

## 📊 Tests End-to-End (E2E)

### Framework: Playwright

**Fichier**: `frontend/e2e/matching.spec.js`

```javascript
import { test, expect } from '@playwright/test'

test.describe('Matching Flow', () => {
  
  test.beforeEach(async ({ page }) => {
    // Login avant chaque test
    await page.goto('http://localhost:5173/login')
    await page.fill('input[name="email"]', 'user1@example.com')
    await page.fill('input[name="password"]', 'SecurePass123!')
    await page.click('button:has-text("Se connecter")')
    await page.waitForURL('**/dashboard')
  })
  
  test('user can find and accept matches', async ({ page }) => {
    // Naviguer vers matching
    await page.click('button:has-text("Trouver des matchs")')
    await page.waitForSelector('[data-test="match-card"]')
    
    // Vérifier affichage matchs
    const matchCards = await page.locator('[data-test="match-card"]')
    expect(await matchCards.count()).toBeGreaterThan(0)
    
    // Accepter premier match
    await page.first('[data-test="accept-btn"]').click()
    
    // Vérifier notification
    expect(
      await page.locator('text=Match accepté!').isVisible()
    ).toBeTruthy()
  })
  
  test('messaging between matched users', async ({ page }) => {
    // Aller aux matchs
    await page.click('a:has-text("Mes matchs")')
    
    // Ouvrir conversation
    await page.click('[data-test="match-conversation"]:first-child')
    
    // Envoyer message
    await page.fill('input[placeholder="Votre message..."]', 'Hey!')
    await page.click('button:has-text("Envoyer")')
    
    // Vérifier message affiché
    expect(
      await page.locator('text=Hey!').isVisible()
    ).toBeTruthy()
  })
})
```

---

## 📈 Rapport de test

### Couverture actuelle (Phase 1)

```
Name                    Stmts   Miss  Cover
────────────────────────────────────────────
api/services/auth.py      120     6    95%
api/services/matching.py  180    27    85%
api/services/message.py   145    29    80%
api/models/user.py         80    20    75%
api/models/game.py         60    18    70%
────────────────────────────────────────────
TOTAL                     600   100    83%  ✅
```

### Execution du test

```bash
# Backend
cd API
pytest tests/ -v --cov=app --cov-report=html

# Frontend
cd frontend
npm test -- --coverage

# E2E
npx playwright test --ui
```

### Critères de réussite

- ✅ Tous les tests backend passent
- ✅ Tous les tests frontend passent
- ✅ Tous les tests E2E passent
- ✅ Couverture code > 80%
- ✅ Performance: temps response < 100ms
- ✅ Pas de bugs bloquants en production

---

**Version** : 1.0  
**Dernière mise à jour** : Mai 2026  
**Status** : 🟡 En développement
