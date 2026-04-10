# External Integrations

**Analysis Date:** 2026-04-09

## APIs & External Services

**LLM / AI:**
- Anthropic Claude — powers all agent reasoning and text generation
  - Adapter: `claude_local` (configured per agent)
  - Auth: `ANTHROPIC_API_KEY`
  - Used by: all agents (`max_ceo_assistant`, `alex_sales_assistant`)

**CRM:**
- Notion API — lightweight CRM backend for prospect and deal tracking
  - SDK/Client: Python `requests` library, direct REST calls
  - Base URL: `https://api.notion.com/v1`
  - API version: `2022-06-28`
  - Auth: `NOTION_TOKEN` (Bearer token), `NOTION_CRM_DB_ID`
  - Operations: query contacts, create prospect pages, update lead status
  - Skill file: `skills/crm_lookup/SKILL.md`

## Data Storage

**Databases:**
- Notion database — sole CRM/data store
  - Connection: `NOTION_TOKEN` + `NOTION_CRM_DB_ID`
  - Client: raw `requests` HTTP calls
  - Schema properties: Nom (title), Email, Tél, Projet, Surface, Budget, Délai, Source, Statut, Assigné à

**File Storage:**
- Local filesystem only — Company Folder files on Paperclip host

**Caching:**
- None

## Authentication & Identity

**Auth Provider:**
- Paperclip platform — manages agent identity, heartbeat JWT injection
  - `PAPERCLIP_API_KEY` injected automatically at runtime (not stored manually)
- Notion — integration token (`NOTION_TOKEN`) stored in Paperclip Secrets
- Email — app password or OAuth token (`EMAIL_PASS`) stored in Paperclip Secrets

## Email

**Protocol:** IMAP (read), SMTP (send drafts for human validation)
- Host (IMAP): `EMAIL_IMAP_HOST` (default: `imap.gmail.com`), port `993` SSL
- Host (SMTP): `EMAIL_SMTP_HOST` (default: `smtp.gmail.com`), port `587`
- User: `EMAIL_USER` (contact@bellacucina.fr)
- Auth: `EMAIL_PASS`
- Skill file: `skills/email_handler/SKILL.md`
- Mailboxes monitored: `contact@bellacucina.fr` (Alex), `team@bellacucina.fr` (Max, read-only)

## Monitoring & Observability

**Error Tracking:**
- None — errors surface as Paperclip alert tickets assigned to Marc

**Logs:**
- No file logging of email content (GDPR constraint enforced in skill)
- IMAP errors → Paperclip ticket "alerte technique" assigned to Marc

## CI/CD & Deployment

**Hosting:**
- Paperclip platform (local or managed)
- Install: `npx paperclipai onboard`

**CI Pipeline:**
- None detected

## Environment Configuration

**Required env vars (configured in Paperclip Secrets UI):**
- `EMAIL_IMAP_HOST`
- `EMAIL_IMAP_PORT`
- `EMAIL_SMTP_HOST`
- `EMAIL_SMTP_PORT`
- `EMAIL_USER`
- `EMAIL_PASS`
- `NOTION_TOKEN`
- `NOTION_CRM_DB_ID`
- `ANTHROPIC_API_KEY`

**Auto-injected by Paperclip at runtime:**
- `PAPERCLIP_API_URL`
- `PAPERCLIP_API_KEY`

**Secrets location:**
- Paperclip UI → Settings → Secrets
- Example template: `tools/secrets.env.example` (placeholder values only, never commit real values)

## Webhooks & Callbacks

**Incoming:**
- None — agents poll on cron schedule (heartbeat), not event-driven webhooks

**Outgoing:**
- Notion API calls on each agent run (CRM read/write)
- IMAP polling on each agent heartbeat

## Planned / Roadmap Integrations

- Google Calendar — appointment booking (`Agent Designer` phase, 2 days effort)
- HubSpot or Airtable — alternative CRM backends (noted as compatible in `skills/crm_lookup/SKILL.md`)
- STT/TTS web app — voice interface for mobile (3 days effort)

---

*Integration audit: 2026-04-09*
