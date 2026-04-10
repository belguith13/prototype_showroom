# Technology Stack

**Analysis Date:** 2026-04-09

## Languages

**Primary:**
- Python — Skill implementation (`skills/email_handler/SKILL.md`, `skills/crm_lookup/SKILL.md`)
- Markdown — Agent definitions, routines, company knowledge, all `.md` files

**Secondary:**
- None detected beyond Python code snippets embedded in skill definitions

## Runtime

**Environment:**
- Paperclip AI platform (self-hosted or managed) — runtime host for all agents
- Python 3.x — used inside skill code blocks (IMAP, HTTP calls)

**Package Manager:**
- Not applicable — no package manifest present; this is a Paperclip "Company Folder" configuration project, not a standalone app

## Frameworks

**Core:**
- Paperclip AI — agent orchestration, heartbeat scheduling, ticket management, secrets injection
  - Onboarded via: `npx paperclipai onboard`
  - API base: `POST /api/companies`, `POST /api/companies/{id}/agents`
  - Runtime injects: `PAPERCLIP_API_URL`, `PAPERCLIP_API_KEY` via heartbeat JWT

**LLM Adapter:**
- `claude_local` — Anthropic Claude, configured per agent via `adapterType: claude_local`
  - Requires: `ANTHROPIC_API_KEY`

**Testing:**
- None detected

**Build/Dev:**
- None detected — project is declarative configuration, not built

## Key Dependencies

**Critical:**
- `imaplib` (Python stdlib) — IMAP email reading in `email_handler` skill
- `email` (Python stdlib) — email parsing in `email_handler` skill
- `requests` (Python third-party) — Notion API HTTP calls in `crm_lookup` skill

**Infrastructure:**
- Paperclip platform — all agent execution, scheduling, ticketing
- Notion API (`https://api.notion.com/v1`, version `2022-06-28`) — CRM backend

## Configuration

**Environment:**
- All secrets managed via Paperclip UI → Settings → Secrets module
- Template: `tools/secrets.env.example`
- Required vars: `EMAIL_IMAP_HOST`, `EMAIL_IMAP_PORT`, `EMAIL_SMTP_HOST`, `EMAIL_SMTP_PORT`, `EMAIL_USER`, `EMAIL_PASS`, `NOTION_TOKEN`, `NOTION_CRM_DB_ID`, `ANTHROPIC_API_KEY`
- `PAPERCLIP_API_URL` and `PAPERCLIP_API_KEY` are injected automatically at runtime

**Build:**
- Not applicable — no build step

## Platform Requirements

**Development:**
- Paperclip platform running locally (`npx paperclipai onboard` → authenticated + private mode)
- Notion workspace with integration token and CRM database
- Gmail/IMAP-compatible mailbox with app password or OAuth token

**Production:**
- Paperclip self-hosted or managed cloud
- Anthropic API access (Claude)
- IMAP/SMTP mail server
- Notion integration

---

*Stack analysis: 2026-04-09*
