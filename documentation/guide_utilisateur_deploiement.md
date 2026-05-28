# 📖 Documentation - GameConnect

## 👥 Documentation utilisateur

### Guide de démarrage - Joueurs

#### 1. S'inscrire sur la plateforme

**Étapes:**

1. Accédez à [gameconnect.com/register](https://gameconnect.com/register)
2. Remplissez le formulaire:
   - **Email** : Votre adresse email valide
   - **Pseudo** : Votre identité gaming (3-20 caractères)
   - **Mot de passe** : Min 8 caractères (1 maj, 1 min, 1 chiffre)
3. Cliquez **S'inscrire**
4. Vérifiez votre email (lien de confirmation)
5. C'est fait! 🎉

**Conseils:**
- Utilisez un email que vous consultez régulièrement
- Choisissez un mot de passe unique et sécurisé
- Gardez votre pseudo memorable pour que les autres vous trouvent

#### 2. Compléter votre profil

Après inscription, complétez rapidement votre profil:

**Informations de base:**
- **Avatar** : Ajoutez une photo de profil (JPG/PNG, max 5MB)
- **Bio** : Présentez-vous en 500 caractères max
  - Exemple: "Passionate FPS player 🎮 EU, Gold Valorant, looking for team mates!"

**Préférences gaming:**
- **Région** : Où vous jouez habituellement (Europe, US, APAC, etc.)
- **Fuseau horaire** : Important pour matcher avec des joueurs proche de votre heure
- **Niveau de compétence** :
  - 🟢 Débutant: Moins de 50 heures de jeu
  - 🟡 Intermédiaire: 50-500 heures
  - 🟠 Avancé: 500+ heures
  - 🔴 Professionnel: Joueur compétitif/pro

**Communication:**
- **Préférence** : Chat texte / Vocal / Vidéo
  - **Chat texte** : Sans engagement, casual
  - **Vocal** : Better coordination
  - **Vidéo** : For team building

**Confidentialité:**
- Profil Public : Tous peuvent voir
- Profil Privé : Vous contrôlez qui voit

✅ **Sauvegardez** une fois terminé

---

### 3. Ajouter vos jeux

**Où les ajouter:**

1. Allez à **"Mes Jeux"** dans le menu
2. Cliquez **"+ Ajouter un jeu"**
3. Recherchez votre jeu parmi nos 50+ titres supportés
4. Renseignez votre niveau dans ce jeu:
   - **Rank/Elo** : Gold, Diamond, etc. (selon le jeu)
   - **Heures jouées** : Estimation du temps investi
5. Validez

**Jeux supportés (Phase 1):**
- League of Legends
- Valorant
- CS:GO / CS2
- Apex Legends
- Call of Duty
- Fortnite
- Dota 2
- Overwatch 2
- ...et plus (50+ jeux)

💡 **Tips:** Ajoutez 3-5 jeux pour de meilleurs matchs

---

### 4. Trouver des coéquipiers (Matching)

**Le système de matching de GameConnect:**

1. Naviguez vers **"Trouver des matchs"**
2. Vous verrez 5 profils potentiels
3. Pour chaque profil, vous voyez:
   - Avatar & pseudo
   - Jeux en commun
   - **Score de compatibilité** (%)
   - Région & fuseau horaire
   - Niveau de compétence

**Score de compatibilité :**
- `100%` : Parfait match (jeux + niveau + zone horaire identiques)
- `70%+` : Excellent match
- `50%+` : Bon match potentiel
- `-` : Pas assez en commun

**Actions:**
- ✅ **Accepter** : Vous aimeriez jouer avec eux
- ❌ **Rejeter** : Pas intéressé (ne réapparaît pas pendant 30j)

**Important:** Un match devient **confirmé** quand les 2 joueurs s'acceptent mutuellement! 🤝

---

### 5. Communiquer avec vos matchs

**Messagerie GameConnect:**

Après un match confirmé:

1. Allez à **"Messages"** (ou 💬 dans le menu)
2. Sélectionnez la conversation
3. Tapez votre message (max 1000 caractères)
4. Pressez **Envoyer** ou **Entrée**

**Conseils:**
- Soyez respectueux et amical
- Organisez vos parties directement en private
- Utilisez Discord après pour des discussions plus longues
- Les emojis sont supportés! 🎮⚡

**Signaler un utilisateur:**
Si quelqu'un est toxique/spam/inapproprié:
- Cliquez sur **⋯** (menu message)
- Sélectionnez **"Signaler"**
- Décrivez le problème
- Notre équipe examine dans les 24h

---

### 6. Dashboard - Votre vue d'ensemble

À chaque connexion, vous voyez:

```
┌─────────────────────────────────────┐
│ Bienvenue, [Pseudo] 👋              │
├─────────────────────────────────────┤
│                                      │
│ 🎮 Mes Jeux (3)          ⚡ Matchs   │
│ LOL • Valorant • Apex    en attente │
│                           (2)        │
│                                      │
│ 🤝 TROUVER DES MATCHS [CTA]        │
│ Score: 85% | Score: 72%  | Score:  │
│                                      │
│ 💬 Messages (3 non-lus)  📊 Stats  │
│ Sarah: "Dispo?"          Matchs: 5  │
│ Tom: "OK pour demain?"   Score avg: │
│ Emma: "GG!"              Messages: 23
│                                      │
└─────────────────────────────────────┘
```

---

## 🏠 Administration

### Guide administrateur

#### Accès admin

1. Connectez-vous avec compte admin
2. Allez à **/admin** dans le menu (pour admins uniquement)
3. Dashboard admin avec statistiques

#### Responsabilités

**Modération:**
- Examiner signalements utilisateurs
- Avertir/Bannir utilisateurs toxiques
- Effacer contenu inapproprié

**Gestion contenu:**
- Ajouter/Modifier/Supprimer jeux du catalogue
- Valider images de profil
- Gérer catégories/genres

**Analytics:**
- Voir nombre d'utilisateurs actifs
- Tracker matches créés/jour
- Surveiller messages/jour
- Voir taux de rétention

**Maintenance:**
- Générer rapports
- Gérer incidents
- Mettre à jour conditions d'utilisation

---

## 🛠️ Guide de déploiement

### Prérequis

- Domaine web configuré
- Certificat SSL/TLS
- Serveur Linux (Ubuntu 20.04+ recommandé)
- Docker & Docker Compose (optionnel)

### Déploiement en production

#### Option 1: Docker Compose (Recommandé)

```bash
# 1. Cloner le dépôt
git clone https://github.com/your-org/gameconnect.git
cd gameconnect

# 2. Configurer fichiers .env
cp .env.production.example .env.production

# Éditer .env avec paramètres production:
# - DB_PASS: Mot de passe complexe
# - JWT_SECRET: Clé secrète aléatoire (32+ caractères)
# - DOMAIN: votre-domaine.com

# 3. Déployer
docker-compose -f docker-compose.prod.yml up -d

# 4. Initialiser base de données
docker-compose -f docker-compose.prod.yml exec api python -m alembic upgrade head

# 5. Créer utilisateur admin
docker-compose -f docker-compose.prod.yml exec api python scripts/create_admin.py

# 6. Vérifier
curl https://votre-domaine.com/api/health
```

#### Option 2: Déploiement manuel

```bash
# 1. Provisionner serveur Ubuntu 20.04

# 2. Installer dépendances
sudo apt-get update
sudo apt-get install -y \
  python3.10 python3-pip python3-venv \
  nodejs npm \
  mysql-server \
  nginx \
  certbot python3-certbot-nginx

# 3. Cloner et configurer
git clone https://github.com/your-org/gameconnect.git /opt/gameconnect
cd /opt/gameconnect

# Backend
cd API
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cp .env.production.example .env

# 4. Créer service systemd pour API
sudo tee /etc/systemd/system/gameconnect-api.service > /dev/null <<EOF
[Unit]
Description=GameConnect API
After=network.target

[Service]
Type=notify
User=gameconnect
WorkingDirectory=/opt/gameconnect/API
ExecStart=/opt/gameconnect/API/venv/bin/uvicorn app.main:app --host 127.0.0.1 --port 8000
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

# 5. Configurer Nginx comme reverse proxy
sudo tee /etc/nginx/sites-available/gameconnect > /dev/null <<EOF
upstream gameconnect_api {
    server 127.0.0.1:8000;
}

server {
    listen 80;
    server_name votre-domaine.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name votre-domaine.com;
    
    ssl_certificate /etc/letsencrypt/live/votre-domaine.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/votre-domaine.com/privkey.pem;
    
    # Frontend
    location / {
        root /opt/gameconnect/frontend/dist;
        try_files $uri $uri/ /index.html;
    }
    
    # Backend API
    location /api/ {
        proxy_pass http://gameconnect_api/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
EOF

# 6. Activer Nginx
sudo ln -s /etc/nginx/sites-available/gameconnect /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

# 7. SSL avec Let's Encrypt
sudo certbot certonly --nginx -d votre-domaine.com

# 8. Démarrer services
sudo systemctl start gameconnect-api
sudo systemctl enable gameconnect-api
```

### Configuration production

#### Variables d'environnement

```env
# Database
DB_HOST=database.internal
DB_USER=gameconnect_prod
DB_PASS=SECURE_PASSWORD_CHANGE_ME
DB_NAME=esport_social_prod

# Security
JWT_SECRET=generate-with: openssl rand -hex 16
JWT_ALGORITHM=HS256
JWT_EXPIRE_HOURS=168

# Server
API_HOST=127.0.0.1
API_PORT=8000
DEBUG=False

# Domain
DOMAIN=gameconnect.com
CORS_ORIGIN=https://gameconnect.com

# Email (SendGrid)
SENDGRID_API_KEY=SG.xxxxx
SENDGRID_FROM=noreply@gameconnect.com

# Sentry (Error tracking)
SENTRY_DSN=https://xxxxx@sentry.io/xxxxx

# Redis (Cache)
REDIS_URL=redis://redis:6379/0
```

#### Backup base de données

```bash
# Backup manuel
mysqldump -u root -p esport_social_prod > /backups/backup-$(date +%Y%m%d).sql

# Script backup automatique (cron)
# /usr/local/bin/backup-db.sh
#!/bin/bash
BACKUP_DIR="/backups/mysql"
DB_NAME="esport_social_prod"
DB_USER="gameconnect_prod"
DATE=$(date +%Y%m%d_%H%M%S)

mysqldump -u $DB_USER -p$DB_PASS $DB_NAME | gzip > $BACKUP_DIR/backup_$DATE.sql.gz

# Nettoyer anciens backups (>30 jours)
find $BACKUP_DIR -name "backup_*.sql.gz" -mtime +30 -delete

# Ajouter à crontab:
# 0 2 * * * /usr/local/bin/backup-db.sh (chaque jour à 2h)
```

#### Monitoring

```bash
# Vérifier logs API
sudo journalctl -u gameconnect-api -f

# Vérifier logs Nginx
sudo tail -f /var/log/nginx/error.log

# Monitor ressources
watch -n 1 'free -h && df -h'

# Health check
curl https://gameconnect.com/api/health
```

### Checklist de déploiement

Avant de passer en production:

- [ ] Base de données migrée et testée
- [ ] Variables d'env configurées et sécurisées
- [ ] SSL/HTTPS activé
- [ ] Certificat SSL configuré (valide 90j+)
- [ ] Backups automatisés configurés
- [ ] Monitoring et alertes mis en place
- [ ] Load tests passés (1000+ users)
- [ ] Security audit effectué
- [ ] Documentation à jour
- [ ] Rollback plan documenté
- [ ] Équipe alertée et ready
- [ ] DNS pointe sur nouveau serveur

### Rollback en cas de problème

```bash
# 1. Stop les services
docker-compose -f docker-compose.prod.yml down
# ou
sudo systemctl stop gameconnect-api nginx

# 2. Restaurer version précédente
git checkout previous-commit
docker-compose -f docker-compose.prod.yml up -d

# 3. Si issue DB, restaurer backup
mysql esport_social < /backups/backup-YYYYMMDD.sql

# 4. Redémarrer
sudo systemctl restart gameconnect-api
```

---

## 🚨 Troubleshooting

### Erreurs courantes

#### Backend refuse de démarrer

```
Error: Cannot connect to MySQL
```

**Solution:**
```bash
# Vérifier MySQL tourne
sudo systemctl status mysql

# Vérifier identifiants .env
mysql -u root -p -h localhost

# Vérifier port 3306 ouvert
sudo netstat -tlnp | grep 3306
```

#### API répond lentement

```bash
# Vérifier query performance
SLOW_QUERY_LOG sur MySQL

# Vérifier indices
SHOW INDEX FROM matches;
SHOW INDEX FROM messages;

# Ajouter index si absent
CREATE INDEX idx_matches_status ON matches(status);
```

#### Messages ne s'envoient pas

```bash
# Vérifier logs API
docker-compose -f docker-compose.prod.yml logs api

# Vérifier permission utilisateur
SELECT user(), DATABASE();

# Test message direct
curl -X POST http://localhost:8000/api/messages \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -d '{\"recipient_id\": \"...\", \"content\": \"test\"}'
```

---

**Version** : 1.0  
**Dernière mise à jour** : Mai 2026  
**Status** : ✅ Production ready
