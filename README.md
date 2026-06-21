# 💰 Money Tracker — Tableau de Bord Financier Local-First

<p align="center">
  <img src="https://img.shields.io/badge/Local--First-100%25-blueviolet?style=for-the-badge&logo=sqlite" alt="Local-First 100%" />
  <img src="https://img.shields.io/badge/FastAPI-0.111-emerald?style=for-the-badge&logo=fastapi" alt="FastAPI" />
  <img src="https://img.shields.io/badge/React-18.3-blue?style=for-the-badge&logo=react" alt="React" />
  <img src="https://img.shields.io/badge/Sécurité-AES--128%20Fernet-red?style=for-the-badge&logo=letsencrypt" alt="Chiffrement Fernet" />
  <img src="https://img.shields.io/badge/Méthodologie-bMAD-orange?style=for-the-badge&logo=gitbook" alt="Méthodologie bMAD" />
</p>

---

## ⚡ ESPACE RECRUTEUR : Démo Instantanée (Zero-Setup)

> [!TIP]
> **Vous êtes en entretien ou vous examinez cette candidature ?**
> Vous pouvez tester l'application immédiatement sans installer de dépendances (Python/Node) ni configurer de base de données.
>
> 🚀 **Comment faire ?**
> 1. Téléchargez ou clonez ce dépôt.
> 2. Double-cliquez sur le fichier **[`demo.html`](demo.html)** dans votre explorateur de fichiers.
> 3. L'application se lance dans votre navigateur avec un jeu de données complet simulé et persistant (`localStorage`).
> 4. Rendez-vous sur l'onglet **"Architecture & bMAD"** pour suivre la présentation interactive intégrée !

---

## 🏗️ Architecture Technique (C4 Model)

L'application est entièrement conçue autour du paradigme **Local-First** : aucune donnée financière ne transite par un serveur cloud externe, assurant une confidentialité absolue.

```mermaid
graph TD
  %% Style definitions
  classDef front fill:#0f172a,stroke:#38bdf8,stroke-width:2px,color:#f8fafc;
  classDef back fill:#0f172a,stroke:#10b981,stroke-width:2px,color:#f8fafc;
  classDef secure fill:#1e1b4b,stroke:#8b5cf6,stroke-width:2px,color:#f8fafc;
  classDef extern fill:#0f172a,stroke:#475569,stroke-width:1px,color:#94a3b8;

  subgraph SPA [Frontend - Interface Utilisateur]
    A["UI React (Composants React 18)"]:::front -->|Appels REST Sécurisés| B["api.js (Centralisation des requêtes)"]:::front
  end

  subgraph API [Backend - Serveur Local FastAPI]
    H["Middleware (Auth & Rate Limit)"]:::back --> C["Routers (Controllers FastAPI)"]:::back
    C --> D["Services Layer (Logique métier)"]:::back
    D -->|Verrou unique Lock| E["Sync Engine (Moteur de Synchro)"]:::back
  end

  subgraph Storage [Stockage & Sécurité Système]
    F[("Base SQLite Locale (WAL Mode)")]:::secure
    G["OS Keychain (Keyring macOS/Windows)"]:::secure
  end

  subgraph Providers [Services & APIs Externes]
    I["Enable Banking API (DSP2)"]:::extern
    J["Trade Republic API (via pytr)"]:::extern
    K["yfinance & CoinGecko (Marchés)"]:::extern
  end

  B -->|Localhost + Clé API comparative| H
  C -->|Transactions ORM asynchrones| F
  D -->|Clés de chiffrement| G
  E -->|Liaison Bancaire OAuth2| I
  E -->|Liaison Portefeuille (WebSockets)| J
  E -->|Cotations en temps réel (Cache 15m)| K

  style SPA fill:#020617,stroke:none;
  style API fill:#020617,stroke:none;
  style Storage fill:#020617,stroke:none;
  style Providers fill:#020617,stroke:none;
```

---

## ✨ Points Forts de l'Implémentation & Plus-value Technique

### 1. 🛡️ Sécurité & Confidentialité au repos
- **Chiffrement au niveau colonne (Fernet AES-128)** : Les données hautement sensibles (`IBAN`, `Solde`, `Tokens`) sont chiffrées en base SQLite. L'accès en clair se fait de manière transparente dans le code Python via des propriétés encapsulées de l'ORM SQLAlchemy.
- **Intégration Trousseau OS (Keyring)** : Pour éviter le stockage de secrets et de mots de passe de comptes d'investissements dans des fichiers `.env` ou dans le code source, l'application utilise l'API système native (macOS Keychain / Windows Credential Manager) via `keyring`.

### 2. 🔄 Moteur de Synchronisation Concurrente Résilient
- **Mutex Lock Asynchrone (`asyncio.Lock`)** : SQLite gérant mal les verrous concurrents en écriture, un verrou exclusif global empêche l'apparition d'erreurs `database is locked` lors des synchronisations simultanées.
- **Mécanisme de Cooldown & Fallback** : Un cache local de 15 minutes protège les API externes du spam (Rate Limit) et permet à l'application de fonctionner parfaitement en mode dégradé hors-ligne.

### 3. 📈 Moteur de Simulation Financière
- **Intérêts Composés Dynamiques** : Calcule en moins de 10 ms l'évolution future du capital selon 3 scénarios (Pessimiste, Base, Optimiste).
- **Comparaison de Scénarios** : Permet de superposer plusieurs projections sur un graphique Chart.js interactif.

---

## 🤖 Méthodologie bMAD : Rigueur de Développement IA

Ce projet n'a pas été écrit de façon improvisée. Il est le fruit d'une collaboration structurée avec des agents IA en suivant le framework **bMAD (Behavior-Driven Model Agent Development)**, qui simule les processus de développement d'équipes Senior :

1. **PRD (Product Requirements Document)** : Définition en amont de **64 exigences fonctionnelles** et **25 exigences non-fonctionnelles** dans `prd.md`.
2. **ADR (Architectural Decision Records)** : Rédaction de **10 ADR** (d'AD-1 à AD-10) documentant et justifiant chaque décision technique (ex: choix de l'algorithme Fernet, gestion du Mutex, machine d'état hors-ligne).
3. **Sprint & Epics Planning** : Suivi de sprint découpé en épopées documenté dans `epics.md` et `sprint-status.yaml`.
4. **Boucle d'Exécution & Tests** : Exécution et tests unitaires/d'intégration automatisés par l'agent IA à chaque modification pour éliminer les régressions techniques.

---

## 💻 Structure du Projet

```
money-tracker/
├── backend/               ← API REST asynchrone Python FastAPI
│   ├── main.py            ← Point d'entrée de l'application, Lifespan et Tâches de fond
│   ├── database.py        ← Connexion SQLite, Mode WAL et Validation d'intégrité
│   ├── models.py          ← Modèles de données SQLAlchemy avec accesseurs chiffrés
│   ├── security.py        ← Moteur de chiffrement Fernet et Gestion de clé API locale
│   ├── routers/           ← Contrôleurs d'API (dashboard, accounts, subscriptions, forecast)
│   └── services/          ← Logique métier (synchro banques, TR, cache des prix yfinance)
├── frontend/              ← Client unique React SPA
│   ├── src/App.jsx        ← Machine d'état réseau et Routage hash-based
│   ├── src/api.js         ← Centralisation des requêtes Axios vers le backend local
│   └── src/pages/         ← Vues utilisateur (Dashboard, Comptes, Investissements, Abonnements...)
├── ARCHITECTURE.md        ← Spécifications de sécurité et de flux de données
└── demo.html              ← Démonstration autonome interactive avec Pitch Deck d'entretien intégré
```

---

## 🚀 Lancer l'Application Réelle

### Prérequis
- Python 3.9+ (ou 3.11+)
- Node.js 18+

### Démarrage
Pour lancer le serveur backend local et l'interface React :
```bash
chmod +x start.sh
./start.sh
```
Le backend sera lancé sur `http://127.0.0.1:8000` et le frontend sur `http://localhost:5173`.
