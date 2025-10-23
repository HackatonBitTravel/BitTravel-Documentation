
# Documentation de BitTravel 

**Application de réservation de tickets de transport avec paiement Bitcoin Lightning et Mobile Money**

BitTravel est une solution innovante permettant aux voyageurs au Sénégal et en Afrique de l'Ouest de réserver et payer leurs tickets de transport en ligne, avec support du Bitcoin Lightning Network et des services de Mobile Money locaux.

-----------

## Table des matières

1.  [Architecture du système]
2.  [Configuration]
3.  [Démarrage rapide]
4.  [Contraintes techniques]
5.  [Prochaine itération]

----------

##    Architecture du système

### Schéma global

```
┌─────────────────────────────────────────────────────────────┐
│             UTILISATEURS / AGENCES / CONTROLLEURS           │  
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                   FRONTEND (React/Vite)                     │
│  • Interface utilisateur (recherche, réservation)           │
│  • Dashboard agence                                         │
│  • Intégration widgets paiement                             │
│  • Génération/scan QR codes                                 │
└──────────────────────┬──────────────────────────────────────┘
                       │ 
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                    BACKEND (FastAPI)                        │
│  • API REST                                                 │
│  • Authentification JWT                                     │
│  • Gestion réservations                                     │
│  • Signatures cryptographiques (secp256k1)                  │
│  • Génération tickets PDF                                   │
└─────┬──────────────┬──────────────┬────────────────────_────┘
      │              │              │
      ▼              ▼              ▼
┌──────────┐  ┌──────────┐  ┌──────────────────┐
│PostgreSQL│  │  KKiapay │  │  BTCPay Server   │
│   DB     │  │  (MTN,   │  │  (Lightning)     │
│          │  │   Moov,  │  │                  │
│          │  │   Wave)  │  │                  │
└──────────┘  └──────────┘  └──────────────────┘
      ▲
      │
      ▼
┌─────────────────────────────────────────────────────────────┐
│                  BITBOT (IA Chatbot)                        │
│  • Assistant conversationnel (FR/WO/EN)                     │
│  • RAG avec FAISS vectorstore                               │
│  • Modèle Gemini (Google)                                   │
│  • Support multilingue                                      │
└─────────────────────────────────────────────────────────────┘

```

### Stack technologique

**Frontend**
React, TypeScript, Vite, Tailwind CSS, shadcn/ui

**Backend**
Python 3.10+, FastAPI, SQLAlchemy, PostgreSQL

**IA/Chatbot**
LangChain, FAISS, Google Gemini, Hugging Face

**Paiements**
KKiapay API, BTCPay Server (Lightning)

**Sécurité**
JWT, secp256k1 (signatures), HTTPS

----------

## Configuration

### 1. Prérequis système

-   **Python** : 3.10 ou supérieur
-   **Node.js** : 18+ (avec npm/pnpm)
-   **PostgreSQL** : 13+
-   **Comptes tiers** :
    -   Compte KKiapay (sandbox pour tests)
    -   Instance BTCPay Server configurée
    -   Clé API Google Gemini
    -   Token Hugging Face (optionnel)

### 2. Variables d'environnement

#### Backend (.env)

```env
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/bittravel

# Application
APP_NAME=BitTravel API
DEBUG=True
API_VERSION=v1
SECRET_KEY=votre-cle-secrete-super-longue-et-aleatoire

# KKiapay
KKIAPAY_PUBLIC_KEY=votre_cle_publique
KKIAPAY_PRIVATE_KEY=votre_cle_privee
KKIAPAY_SECRET=votre_secret
KKIAPAY_SANDBOX=True

# BTCPay Server
BTCPAY_URL=https://votre-btcpay.com
BTCPAY_API_KEY=votre_api_key
BTCPAY_STORE_ID=votre_store_id

```

#### BitBot (.env)

```env
# Google Gemini
GOOGLE_API_KEY=votre_cle_api_gemini

# Hugging Face (pour vectorstore)
HF_TOKEN=votre_token_huggingface
HF_REPO_ID=bitbot/vectorstore

```

#### Frontend (.env)

```env
VITE_API_URL=http://localhost:8000
VITE_KKIAPAY_PUBLIC_KEY=votre_cle_publique

```

----------

##  Démarrage rapide

### Installation complète

#### 1. Backend API

```bash
# Cloner et configurer
cd bittravel-backend
python -m venv venv
source venv/bin/activate  # ou venv\Scripts\activate (Windows)
pip install -r requirements.txt

# Configurer .env
cp .env.example .env
# Éditer .env avec vos vraies clés

# Créer la base de données
psql -U postgres -c "CREATE DATABASE bittravel;"

# Lancer le serveur
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

```

**API accessible sur** : http://localhost:8000  
**Documentation** : http://localhost:8000/docs

#### 2. Frontend

```bash
cd bittravel-frontend
npm install

# Configurer .env
echo "VITE_API_URL=http://localhost:8000" > .env

# Lancer en développement
npm run dev

```

**Frontend accessible sur** : http://localhost:5173

#### 3. BitBot (Chatbot IA)

```bash
cd bitbot
python -m venv .venv
source .venv/bin/activate  # ou .venv\Scripts\activate (Windows)
pip install -r requirements.txt

# Configurer .env avec clés Gemini et HF
# Préparer le vectorstore FAISS (voir README BitBot)

# Lancer le chatbot
python main.py

```

### Architecture locale complète

Une fois tous les services lancés :

-   **Backend API** : http://localhost:8000
-   **Frontend** : http://localhost:5173
-   **BitBot** : Terminal interactif
-   **PostgreSQL** : localhost:5432

----------

##  Contraintes techniques

### Contraintes fonctionnelles

**Multi-paiement**
Support obligatoire de KKiapay (Mobile Money) ET Bitcoin Lightning

**Réduction utilisateur**
10% de réduction automatique pour les utilisateurs authentifiés

**Tickets infalsifiables**
Signatures cryptographiques secp256k1 sur tous les tickets

**Multilingue**
Support Français/Wolof/anglais pour l'assistant BitBot

**Temps réel**
Vérification des paiements Lightning en temps réel

### Contraintes techniques

**Performance**
API < 500ms pour recherche trajets
***Impact :*** Expérience utilisateur

**Sécurité**
JWT valides 7 jours max
***Impact :*** Authentification

**Scalabilité**
Support 100+ réservations/heure
***Impact :*** Infrastructure

**Disponibilité**
***Impact :*** Production

**Responsivité**
 Interface s'adaptant à plusieurs résolutions
***Impact :*** Expérience utilisateur 
### Contraintes de sécurité

-   **HTTPS obligatoire** en production
-   **Rate limiting** : 100 requêtes/heure 
-   **Validation stricte** : Tous les inputs utilisateur validés côté backend
-   **Signatures cryptographiques** : Impossible de falsifier un ticket

### Contraintes d'infrastructure
-   **CORS** : Origines autorisées configurables par environnement

----------
## Indicateurs de suivi (à completer)
----------
##  Prochaine itération
### [1] Dashboard de monitoring 
```
┌─────────────────────────────────────┐
│   BitTravel - Monitoring Dashboard  │
├─────────────────────────────────────┤
│ Réservations aujourd'hui       247  │
│ Paiements Lightning            42   │
│ Paiements Mobile Money         205  │
│ Temps de réponse moyen        380ms │
│ Erreurs API (24h)             3     │
│ Tickets générés               238   │
│ Tickets scannés               156   │
└─────────────────────────────────────┘
```
### [2] Fonctionnalités utilisateur
-   **Gestion des favoris**
    -   Enregistrer trajets fréquents
    -   Rappels pour réservations récurrentes
    -   Suggestions personnalisées
    
### [3] Backend et sécurité

-    **Système de remboursement**
        -   Annulation jusqu'à 2h avant départ
        -   Remboursement automatique (Mobile Money ou Lightning)
        -   Politique flexible par agence
-    **Authentification 2FA**
        -   SMS ou TOTP pour agences
        -   Protection contre fraudes

###  [4] Intelligence artificielle

-    **BitBot avancé**
        -   Réservation vocale (speech-to-text)
        -   Suggestions proactives basées sur historique
        -   Support de plus de langues (espagnol, fon, bambara, peul)
-    **Prédiction de demande**
        -   Machine learning pour anticiper affluence
        -   Ajustement dynamique des prix (yield management)
        -   Recommandations d'horaires aux agences

### [5] Expansion

-    **Support multi-pays**
        -   Traductions complètes
        -   Conformité réglementaire locale
-    **Marketplace étendu**
        -   Location de véhicules
        -   Excursions touristiques

