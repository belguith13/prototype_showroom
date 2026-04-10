# Prototype Bella Cucina — Showroom Cuisine IA
## Solution IA Clé en Main sur Paperclip

---

## Prérequis

- Docker + Docker Compose installés sur le VPS/machine cible
- Compte Claude.ai (Pro ou Max) connecté via le CLI `claude`
- Clés Gmail App Password pour la boîte email

---

## 1. Connexion Claude CLI (sur le VPS)

Les agents Paperclip utilisent l'abonnement Claude.ai via le CLI `claude`. Avant de lancer Docker, connecter le CLI sur le VPS :

```bash
# Installer le CLI claude si absent
npm install -g @anthropic-ai/claude-code

# Se connecter (ouvre un navigateur ou génère un lien)
claude login

# Vérifier la connexion
claude auth status
# → "loggedIn": true, "authMethod": "claude.ai"
```

Le token est stocké dans `~/.claude.json` sur le VPS. Il sera monté en lecture seule dans le container.

---

## 2. Lancer Paperclip via Docker

```bash
git clone https://github.com/paperclipai/paperclip.git
cd paperclip

BETTER_AUTH_SECRET="$(openssl rand -base64 32)" \
PAPERCLIP_PUBLIC_URL="https://app.votredomaine.com" \
PAPERCLIP_DEPLOYMENT_MODE="authenticated" \
PAPERCLIP_DEPLOYMENT_EXPOSURE="public" \
EMAIL_IMAP_HOST="imap.gmail.com" \
EMAIL_IMAP_PORT="993" \
EMAIL_SMTP_HOST="smtp.gmail.com" \
EMAIL_SMTP_PORT="587" \
EMAIL_USER="contact@bellacucina.fr" \
EMAIL_PASS="<gmail-app-password>" \
docker compose -f docker/docker-compose.quickstart.yml \
  -f docker/docker-compose.claude-auth.yml up -d
```

> **Note** : `docker-compose.claude-auth.yml` monte `~/.claude` et `~/.claude.json` en read-only dans le container pour que les agents utilisent l'abonnement Claude.ai.

### Récupérer l'URL Board Claim (premier démarrage uniquement)

```bash
docker compose -f docker/docker-compose.quickstart.yml logs | grep board-claim
```

Visiter l'URL dans le navigateur pour créer le compte administrateur.

---

## 3. Importer la company Bella Cucina

Dans l'UI Paperclip (http://localhost:3100 ou votre domaine) :

1. **Créer une nouvelle company**
2. **Importer depuis GitHub** : `https://github.com/belguith13/prototype_showroom`
3. Vérifier l'aperçu — aucune erreur ne doit apparaître
4. Confirmer l'import

L'import crée automatiquement :
- Les agents **Max** (CEO) et **Alex** (Commercial)
- Le projet **Opérations**
- Les routines **hourly-email-check** et **weekly-ceo-report**
- Les skills email-handler, crm-lookup, kitchen-knowledge

---

## 4. Créer les Goals manuellement

Les goals ne sont pas encore importés automatiquement. Les créer dans **Settings > Goals** (scope : Company) :

| Titre | Description |
|-------|-------------|
| CA annuel 2025 | Atteindre 1 200 000€ de chiffre d'affaires signé sur l'année 2025 |
| Réactivité leads | Répondre à tout lead entrant sous 2h pendant les heures ouvrées |
| Taux de prise de RDV | Atteindre un taux de prise de RDV sur lead entrant ≥ 50% |
| Pipeline actif | Maintenir un pipeline actif d'au moins 15 projets à tout moment |
| Délai devis | Envoyer tout devis dans les 48h suivant la visite showroom |

---

## 5. Test — Flux email entrant prospect

1. Envoyer un e-mail de test à la boîte configurée :
   ```
   De : jean.dupont@gmail.com
   Objet : Renseignements cuisine
   Corps : Bonjour, nous souhaitons refaire notre cuisine d'environ 12m².
           Budget environ 15 000€. Pouvez-vous nous contacter ?
           Cordialement, Jean Dupont
   ```

2. Déclencher manuellement le heartbeat d'Alex :
   - UI Paperclip → **Routines** → `hourly-email-check` → **Run now**

3. Vérifier dans Paperclip :
   - Un ticket "Nouveau lead — Jean Dupont" créé dans Opérations
   - Le ticket contient les données extraites + draft de réponse
   - Logs de l'agent visibles dans l'onglet Activity

---

## Structure du projet

```
prototype_showroom/
├── COMPANY.md              → Manifest company (schema agentcompanies/v1)
├── .paperclip.yaml         → Config Paperclip (adapter, routines, budgets)
├── agents/
│   ├── max/
│   │   ├── AGENTS.md       → Max — CEO Assistant
│   │   ├── SOUL.md         → Personnalité et ton
│   │   └── HEARTBEAT.md    → Instructions d'exécution hebdomadaire
│   └── alex/
│       ├── AGENTS.md       → Alex — Sales Assistant
│       ├── SOUL.md         → Personnalité et ton
│       └── HEARTBEAT.md    → Instructions d'exécution horaire
├── skills/
│   ├── email-handler/SKILL.md    → IMAP, classification, drafts SMTP
│   ├── crm-lookup/SKILL.md       → Connexion CRM Notion
│   └── kitchen-knowledge/SKILL.md → Catalogue produits, prix, fournisseurs
├── projects/
│   └── operations/PROJECT.md    → Projet conteneur des routines
└── tasks/
    ├── hourly-email-check/TASK.md  → Routine Alex (mar–sam 10h–19h)
    └── weekly-ceo-report/TASK.md   → Routine Max (lundi 08h00)
```

---

## Roadmap prototype → production

| Étape | Description | Statut |
|-------|-------------|--------|
| Company Folder | 2 agents + skills + routines | ✅ Fait |
| Déploiement Docker | Mode authenticated + public | ✅ Fait |
| Test flux email | Alex lit et traite les emails entrants | ✅ Fait |
| Connecteur agenda | Google Calendar pour les RDV | 🔲 À faire |
| Agent Designer | Léa — exports Winner/DXF | 🔲 À faire |
| Interface vocale | App web STT/TTS mobile | 🔲 À faire |
| Déploiement VPS | Hostinger KVM2 + Nginx + HTTPS | 🔲 À faire |
