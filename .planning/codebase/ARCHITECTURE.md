# Architecture

**Analysis Date:** 2026-04-09

## Pattern Overview

**Overall:** Multi-Agent AI System on Paperclip Platform (declarative "Company Folder" pattern)

**Key Characteristics:**
- No application code compiled or deployed — the project is a structured folder of Markdown definitions consumed by the Paperclip runtime
- Agents are autonomous LLM-driven workers defined declaratively; skills provide executable Python tooling
- Hierarchical agent reporting: CEO Assistant (Max) supervises Sales Assistant (Alex); both report upward to human operators
- Human-in-the-loop enforced by design: no agent sends emails or takes final commercial decisions without human validation
- All side-effects are mediated through two channels: Paperclip tickets (internal) and draft artifacts (email bodies)

## Layers

**Company Context Layer:**
- Purpose: Provides business identity, goals, and product knowledge to all agents via Paperclip context injection
- Location: `company/`, `knowledge/`
- Contains: `company/company.md` (identity), `company/goals.md` (KPIs and SLAs), `knowledge/products.md` (catalog and pricing)
- Depends on: Nothing — static reference data
- Used by: All agents at runtime (context window)

**Agent Layer:**
- Purpose: Defines agent personas, responsibilities, escalation rules, and capability boundaries
- Location: `agents/`
- Contains: `agents/ceo_assistant.md` (Max — strategic, weekly cadence), `agents/sales_assistant.md` (Alex — operational, hourly cadence)
- Depends on: Company Context Layer (injected), Skills Layer (referenced by name)
- Used by: Paperclip runtime to instantiate agent instances

**Skills Layer:**
- Purpose: Reusable executable modules providing structured I/O with external systems
- Location: `skills/`
- Contains: `skills/email_handler/SKILL.md` (IMAP read, classify, draft), `skills/crm_lookup/SKILL.md` (Notion CRM read/write)
- Depends on: Environment secrets (`EMAIL_*`, `NOTION_*`)
- Used by: Both agents reference skills by name (`email-handler`, `crm-lookup`)

**Routines Layer:**
- Purpose: Cron-scheduled task definitions that wire agents to recurring jobs
- Location: `routines/`
- Contains: `routines/hourly_email_check.md` (Alex, `0 10-19 * * 2-6`), `routines/weekly_ceo_report.md` (Max, `0 8 * * 1`)
- Depends on: Agent Layer, Skills Layer
- Used by: Paperclip scheduler

**Tooling / Secrets Layer:**
- Purpose: Environment variable template for secrets configuration
- Location: `tools/`
- Contains: `tools/secrets.env.example`
- Depends on: Nothing
- Used by: Human operators to configure Paperclip Secrets UI

## Data Flow

**Inbound Lead — Email to CRM to Ticket:**

1. Paperclip cron fires Alex's heartbeat (`0 10-19 * * 2-6`)
2. Alex calls `email-handler` → `fetch_unread()` via IMAP on `contact@bellacucina.fr`
3. Alex classifies each email (prospect_nouveau / client_existant / fournisseur / spam / autre)
4. For `prospect_nouveau`: structured data extracted (name, project, surface, budget, deadline, urgency)
5. Alex calls `crm-lookup` → `lookup_contact()` to check for duplicate in Notion
6. If new: `crm-lookup` → `create_prospect()` writes fiche to Notion database
7. Alex drafts reply email (stored as text, never sent autonomously)
8. Alex creates Paperclip ticket "Nouveau lead — [NOM]" with extracted data + draft, assigned to Sophie
9. Sophie reviews ticket, validates draft, sends from her own mailbox

**Weekly CEO Report Flow:**

1. Paperclip cron fires Max's heartbeat (`0 8 * * 1`, Mondays)
2. Max calls `crm-lookup` to aggregate pipeline metrics (leads, RDVs, devis, CA)
3. Max calls `email-handler` (read-only on `team@bellacucina.fr`) for unprocessed items
4. Max posts formatted report as comment on recurring "Rapport CEO" ticket for Marc
5. Max creates individual alert tickets for each identified attention point

**Escalation Flow:**

- Alex → Max: leads > 20k€, unhappy clients, unusual situations
- Alex → Sophie/Julie: all other operational items
- Max → Marc: strategic alerts, pipeline summaries

**State Management:**
- All persistent state lives in Notion CRM (prospect lifecycle, deal status)
- Ticket state lives in Paperclip (task assignments, drafts, alerts)
- No in-memory or file-based state between agent runs

## Key Abstractions

**Agent:**
- Purpose: An LLM persona with defined capabilities, budget cap, heartbeat schedule, and reporting hierarchy
- Examples: `agents/ceo_assistant.md`, `agents/sales_assistant.md`
- Pattern: Markdown document with identity, responsibilities, constraints, available skills, and escalation rules

**Skill:**
- Purpose: A named, reusable capability module with a YAML frontmatter description (used by Paperclip for routing) and Python implementation code
- Examples: `skills/email_handler/SKILL.md`, `skills/crm_lookup/SKILL.md`
- Pattern: `---\nname: skill-name\ndescription: when to use / not use\n---` followed by documented Python functions

**Routine:**
- Purpose: A scheduled task binding an agent to a cron trigger with explicit step-by-step instructions
- Examples: `routines/hourly_email_check.md`, `routines/weekly_ceo_report.md`
- Pattern: cron expression + agent name + ordered instruction list

**Ticket:**
- Purpose: The primary unit of human-agent handoff; created by agents, actioned by humans
- Pattern: Title format `"Nouveau lead — [NOM]"` or `"Rapport CEO"` with structured description body

## Entry Points

**Alex Heartbeat (operational):**
- Location: `routines/hourly_email_check.md`
- Triggers: Cron `0 10-19 * * 2-6`, or manual "Invoke" button in Paperclip UI
- Responsibilities: Email triage, lead capture, CRM write, draft creation, ticket creation

**Max Heartbeat (strategic):**
- Location: `routines/weekly_ceo_report.md`
- Triggers: Cron `0 8 * * 1` (Monday 08:00)
- Responsibilities: CRM aggregation, pipeline report, alert ticket creation

**Agent @mention:**
- Triggers: Any Paperclip ticket or comment mentioning an agent
- Handled by: Respective agent (Max or Alex) on-demand

## Error Handling

**Strategy:** Graceful degradation with Paperclip ticket escalation

**Patterns:**
- IMAP connection failure → create alert ticket assigned to Marc (no retry loop)
- Empty inbox → silent exit, no ticket created (rule of silence)
- Missing CRM record → create new record rather than fail
- Ambiguous email → escalate to Sophie via ticket rather than guess

## Cross-Cutting Concerns

**Logging:** No file logging; email content never logged (GDPR); errors surface as Paperclip tickets
**Validation:** Human validation required before any outbound email is sent; drafts are always intermediate artifacts
**Authentication:** All credentials via Paperclip Secrets module; `os.environ.get()` pattern enforced, never hardcoded
**Budget Control:** Per-agent monthly token budget enforced by Paperclip (`budgetCents: 4000` for Max, `5000` for Alex)
**Privacy:** Email bodies truncated to 2000 characters before LLM processing; no PII in log files

---

*Architecture analysis: 2026-04-09*
