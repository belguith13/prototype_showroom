# Codebase Structure

**Analysis Date:** 2026-04-09

## Directory Layout

```
prototype_showroom/
├── agents/                    # Agent persona definitions
│   ├── ceo_assistant.md       # Max — CEO Assistant (strategic, weekly)
│   └── sales_assistant.md     # Alex — Sales Assistant (operational, hourly)
├── company/                   # Business context injected into all agents
│   ├── company.md             # Identity, team, contact, KPIs
│   └── goals.md               # 2025 objectives and SLA thresholds
├── knowledge/                 # Domain reference data for agent grounding
│   └── products.md            # Product catalog, price ranges, lead times
├── routines/                  # Scheduled task definitions (cron + instructions)
│   ├── hourly_email_check.md  # Alex — IMAP check every hour (Tue-Sat 10-19)
│   └── weekly_ceo_report.md   # Max — Monday 08:00 pipeline report
├── skills/                    # Reusable executable skill modules
│   ├── email_handler/
│   │   └── SKILL.md           # IMAP read, classify, extract, draft, tickets
│   └── crm_lookup/
│       └── SKILL.md           # Notion CRM lookup, create, update status
├── tools/
│   └── secrets.env.example    # Environment variable template (placeholders only)
├── .planning/
│   └── codebase/              # GSD analysis documents (this directory)
└── README.md                  # Setup guide and test flow
```

## Directory Purposes

**`agents/`:**
- Purpose: One Markdown file per agent defining identity, tone, responsibilities, constraints, available skills, and escalation rules
- Contains: Agent persona documents
- Key files: `agents/ceo_assistant.md`, `agents/sales_assistant.md`

**`company/`:**
- Purpose: Static business context automatically available to all agents as background knowledge
- Contains: Company description, team structure, objectives, KPI thresholds
- Key files: `company/company.md`, `company/goals.md`

**`knowledge/`:**
- Purpose: Domain-specific reference data agents draw on when processing requests
- Contains: Product catalogs, pricing, supplier info
- Key files: `knowledge/products.md`

**`routines/`:**
- Purpose: Cron-scheduled task scripts — each file specifies schedule, agent, and ordered instructions
- Contains: Routine Markdown files with embedded cron expressions
- Key files: `routines/hourly_email_check.md`, `routines/weekly_ceo_report.md`

**`skills/`:**
- Purpose: Reusable capability modules with YAML frontmatter (for Paperclip routing) and Python implementation
- Contains: One subdirectory per skill, each containing a `SKILL.md`
- Key files: `skills/email_handler/SKILL.md`, `skills/crm_lookup/SKILL.md`

**`tools/`:**
- Purpose: Operational tooling and configuration templates
- Contains: `secrets.env.example` — lists all required environment variables with placeholder values

## Key File Locations

**Entry Points:**
- `routines/hourly_email_check.md`: Alex's primary operational trigger
- `routines/weekly_ceo_report.md`: Max's primary reporting trigger

**Configuration:**
- `tools/secrets.env.example`: canonical list of all required secrets
- Agent configs embedded in `agents/*.md` (heartbeat schedule, budget, adapter type)

**Core Logic:**
- `skills/email_handler/SKILL.md`: IMAP connection, classification logic, draft templates, relance logic
- `skills/crm_lookup/SKILL.md`: Notion API client, prospect schema, pipeline status enumeration

**Business Rules:**
- `agents/ceo_assistant.md`: escalation thresholds (deal > 20k€, inactivity > 48h)
- `agents/sales_assistant.md`: email tone rules, follow-up cadence (J+3, J+5)
- `company/goals.md`: SLA definitions (reply < 2h, devis < 48h)

**Knowledge Base:**
- `knowledge/products.md`: product ranges, indicative prices, lead times
- `company/company.md`: team, hours, contact emails

## Naming Conventions

**Files:**
- Agent files: `{role_name}.md` in lowercase with underscore (e.g., `ceo_assistant.md`, `sales_assistant.md`)
- Skill entry: always named `SKILL.md` (uppercase) inside a `{skill_name}/` subdirectory
- Routine files: `{frequency}_{topic}.md` (e.g., `hourly_email_check.md`, `weekly_ceo_report.md`)
- Knowledge files: `{topic}.md` in lowercase

**Directories:**
- Skill dirs: lowercase with underscore matching skill name (e.g., `email_handler/`, `crm_lookup/`)

**Skill frontmatter name field:**
- Kebab-case matching directory name (e.g., `name: email-handler`, `name: crm-lookup`)

## Where to Add New Code

**New agent:**
- Create `agents/{role_name}.md` following the structure in `agents/ceo_assistant.md`
- Register via Paperclip API: `POST /api/companies/{id}/agents`

**New skill:**
- Create `skills/{skill_name}/SKILL.md` with YAML frontmatter block at top
- Frontmatter must include `name:` (kebab-case) and `description:` (when to use / not use)
- Copy to Paperclip runtime: `cp -r skills/ ~/.paperclip/companies/{company_id}/skills/`
- Add required env vars to `tools/secrets.env.example`

**New routine:**
- Create `routines/{frequency}_{topic}.md` with cron expression, agent name, and ordered instructions
- Reference existing skills by their `name:` field from frontmatter

**New knowledge:**
- Add `knowledge/{topic}.md` — plain Markdown, no special format required
- Paperclip injects context automatically if file is in the Company Folder

**New secrets:**
- Add to `tools/secrets.env.example` with `CHANGEME` placeholder
- Document in the relevant `SKILL.md` under "Variables d'environnement requises"

## Special Directories

**`.planning/`:**
- Purpose: GSD planning and analysis documents
- Generated: By GSD tooling
- Committed: Yes

**`.git/`:**
- Purpose: Version control
- Generated: Yes
- Committed: No (internal)

---

*Structure analysis: 2026-04-09*
