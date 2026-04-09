# Prototype Bella Cucina — Showroom Cuisine IA
## Solution IA Clé en Main sur Paperclip

---

## Structure du projet

```
prototype_showroom/
├── company/
│   ├── company.md          → Description de l'entreprise
│   └── goals.md            → Objectifs et KPIs
├── agents/
│   ├── ceo_assistant.md    → Max — Assistant Directeur
│   └── sales_assistant.md  → Alex — Assistant Commercial
├── skills/
│   ├── email_handler/
│   │   └── SKILL.md        → Lecture IMAP, classification, drafts
│   └── crm_lookup/
│       └── SKILL.md        → Connexion CRM Notion
├── routines/
│   ├── weekly_ceo_report.md   → Rapport lundi 08h00 (Max)
│   └── hourly_email_check.md  → Vérif e-mails toutes les heures (Alex)
├── knowledge/
│   └── products.md         → Catalogue produits et prix
└── tools/
    └── secrets.env.example → Template variables d'environnement
```

---

## Démarrage rapide

### 1. Installer Paperclip
```bash
npx paperclipai onboard
# Choisir : authenticated + private (réseau local)
```

### 2. Créer la company "Bella Cucina"
```bash
# Via l'UI Paperclip ou l'API REST
POST /api/companies
{
  "name": "Bella Cucina",
  "goal": "Atteindre 1 200 000€ de CA signé en 2025 en offrant le meilleur conseil cuisine sur mesure à Paris."
}
```

### 3. Créer les agents

**Max — CEO Assistant**
```bash
POST /api/companies/{id}/agents
{
  "name": "Max",
  "title": "CEO Assistant",
  "adapterType": "claude_local",
  "capabilities": "Synthèse hebdomadaire du pipeline, alertes stratégiques pour Marc, monitoring des KPIs.",
  "budgetCents": 4000,
  "heartbeatSchedule": "0 8 * * 1"
}
```

**Alex — Sales Assistant**
```bash
POST /api/companies/{id}/agents
{
  "name": "Alex",
  "title": "Sales Assistant",
  "adapterType": "claude_local",
  "reportsTo": "{max_agent_id}",
  "capabilities": "Lecture e-mails entrants, qualification leads, rédaction de drafts de réponse, suivi relances.",
  "budgetCents": 5000,
  "heartbeatSchedule": "0 10-19 * * 2-6"
}
```

### 4. Copier les skills dans le dossier Paperclip
```bash
cp -r skills/ ~/.paperclip/companies/{company_id}/skills/
```

### 5. Configurer les secrets
Dans l'UI Paperclip → Settings → Secrets, ajouter :
- `EMAIL_IMAP_HOST`, `EMAIL_IMAP_PORT`, `EMAIL_USER`, `EMAIL_PASS`
- `NOTION_TOKEN`, `NOTION_CRM_DB_ID`
- `ANTHROPIC_API_KEY`

---

## Flux de test — Email entrant prospect

1. Envoyer un e-mail de test à contact@bellacucina.fr :
   ```
   De : jean.dupont@gmail.com
   Objet : Renseignements cuisine
   Corps : Bonjour, nous souhaitons refaire notre cuisine d'environ 12m².
           Budget environ 15 000€. Pouvez-vous nous contacter ?
           Cordialement, Jean Dupont
   ```

2. Déclencher manuellement le heartbeat d'Alex (bouton "Invoke" dans l'UI)

3. Vérifier dans Paperclip :
   - Un ticket "Nouveau lead — Jean Dupont" a été créé
   - Le ticket contient les données extraites + le draft e-mail
   - La fiche CRM a été créée dans Notion

4. Sophie valide et envoie le draft depuis sa messagerie

---

## Roadmap prototype → production

| Étape | Description | Effort |
|---|---|---|
| ✅ Prototype | Company Folder + 2 agents + email skill | Fait |
| 🔲 Test e2e | Test flux complet sur vraie boîte e-mail | 1 jour |
| 🔲 Connecteur agenda | Intégration Google Calendar pour les RDV | 2 jours |
| 🔲 Agent Designer | Léa — traitement exports Winner/DXF | 3 jours |
| 🔲 Interface vocale | App web STT/TTS pour mobile | 3 jours |
| 🔲 Dashboard client | Vue simplifiée pour Marc (non-tech) | 2 jours |
