# PRD: Contractor Workflow Automation with OpenClaw on DigitalOcean

## Introduction

An automated construction contractor workflow powered by OpenClaw, an open-source AI agent framework deployed as a DigitalOcean 1-Click Droplet (~$24/mo). The system eliminates administrative overhead for a solo contractor by autonomously monitoring Gmail for new projects, creating Google Sheets tracking, dispatching crew via WhatsApp, tracking material receipts, and sending invoices.

This is a **documentation and configuration-only MVP** -- no custom code. The agent runs entirely on OpenClaw's built-in capabilities (cron jobs, gog skill for Google Workspace, WhatsApp channel, memory). All deliverables are setup guides, config templates, and workspace files that a contractor can deploy on a fresh DigitalOcean account.

When the agent encounters any ambiguous situation (unclear scope doc, missing bid amount, unexpected crew response), it asks the business owner for help via WhatsApp/Slack rather than guessing.

## Goals

- Provide a complete, step-by-step setup guide (README.md) for deploying OpenClaw on DigitalOcean with all required integrations
- Deliver production-ready config templates (OpenClaw config, cron jobs) with placeholder values for easy customization
- Deliver agent workspace files (persona, identity, workflow logic, tool conventions) that define the 10-step contractor workflow
- Deliver message and sheet templates (WhatsApp messages, invoice email, project sheet columns) as reference documents
- Support 1-3 concurrent projects for a solo contractor with a small crew
- Ensure all placeholder values are documented in a single reference table for easy find-and-replace

## User Stories

### US-001: Create environment variable template
**Description:** As a contractor deploying the system, I want a single `.env.example` file listing every placeholder value so I know exactly what credentials and config I need before starting.

**Acceptance Criteria:**
- [ ] `.env.example` exists at repo root
- [ ] Contains all placeholder values: Droplet IP, Anthropic API key, Gmail address, Google Cloud project ID, OAuth client ID/secret, phone numbers (contractor + 3 crew + client), contractor name/business name, timezone, gateway token, sheet ID
- [ ] Each variable has a descriptive comment
- [ ] Phone numbers use E.164 format examples
- [ ] File is gitignored (`.env*` pattern in `.gitignore`)

### US-002: Create OpenClaw config template
**Description:** As a contractor deploying the system, I want an annotated OpenClaw config file so I can copy it to my Droplet and fill in my values.

**Acceptance Criteria:**
- [ ] `config/openclaw.jsonc` exists with JSONC comments explaining each section
- [ ] Sections: LLM provider (Anthropic), agent identity, channels (WhatsApp with allowlist), skills (gog), tools (web_fetch, image, message, memory), cron config, gateway auth
- [ ] All values use `YOUR_*` placeholders matching `.env.example`
- [ ] WhatsApp allowlist includes all crew and client phone placeholders

### US-003: Create cron job definitions
**Description:** As a contractor deploying the system, I want pre-built cron job JSON files so the agent automatically checks email, sends briefings, and follows up with crews on schedule.

**Acceptance Criteria:**
- [ ] `config/cron/email-monitor.json` -- runs every 15 minutes, prompts agent to check unread emails and process new projects
- [ ] `config/cron/morning-briefing.json` -- runs daily at 7 AM, prompts agent to summarize active projects and send WhatsApp briefing to contractor
- [ ] `config/cron/progress-check.json` -- runs weekdays at 4 PM, prompts agent to check in with active crews via WhatsApp
- [ ] Each file has: name, schedule (cron syntax), description, prompt, tools list, enabled flag
- [ ] Schedules respect the configured timezone

### US-004: Create agent persona and identity workspace files
**Description:** As a contractor deploying the system, I want workspace files that define the agent's personality, name, and presentation so it communicates professionally with crew and clients.

**Acceptance Criteria:**
- [ ] `workspace/SOUL.md` -- defines construction PM persona with principles: financial precision, clear communication, proactive tracking, reliability, confidentiality
- [ ] `workspace/IDENTITY.md` -- defines agent name, emoji, tagline, and presentation rules (how it introduces itself to crew vs. clients, never claims to be human)
- [ ] `workspace/USER.md` -- contractor profile template with placeholders for name, business, phone, email, timezone, pay structure (20/80 split), crew roster, and scheduling preferences
- [ ] All `YOUR_*` placeholders match `.env.example`

### US-005: Create agent workflow logic workspace file
**Description:** As a contractor deploying the system, I want a workspace file that defines the complete 10-step workflow logic so the agent knows exactly how to process emails, manage projects, and coordinate crew.

**Acceptance Criteria:**
- [ ] `workspace/AGENTS.md` contains protocols for all workflow steps:
  - Email Monitoring Protocol (steps 1-2): check unread, download scope docs, extract project data
  - Project Sheet Protocol (step 3-4): calculate 80/20 split, append to tracking sheet
  - Crew Notification Protocol (steps 5-6): send WhatsApp offers, handle YES/NO, 24h reminder
  - Material Tracking Protocol (step 7): process receipt images, update budget, alert on overspend
  - Progress Monitoring Protocol (step 8): daily check-ins, flag overdue projects
  - Completion Protocol (step 9): draft invoice email, summarize financials
  - Archive Protocol (step 10): store to memory, final WhatsApp summary
- [ ] Each protocol references specific tools by name (gog, message, memory, image, web_fetch)
- [ ] Ambiguous situations explicitly instruct agent to ask the business owner via WhatsApp/Slack for clarification

### US-006: Create tool usage conventions workspace file
**Description:** As a contractor deploying the system, I want a workspace file that defines how the agent should use each tool so it follows consistent patterns for Gmail, Sheets, WhatsApp, and financial calculations.

**Acceptance Criteria:**
- [ ] `workspace/TOOLS.md` covers: gog (Gmail, Sheets, Drive commands with examples), WhatsApp (message rules, allowlist enforcement, character limits), financial calculations (2 decimal precision, 80/20 formula, verification), memory (what to persist), web_fetch (timeout, cleanup), image (receipt OCR, fallback to manual)
- [ ] Financial calculation rules emphasize: never round intermediate values, verify contractor_cut + crew_pay == bid
- [ ] WhatsApp rules: only message allowlisted numbers, never share financial details cross-role

### US-007: Create project tracking sheet template
**Description:** As a contractor deploying the system, I want a reference document defining the Google Sheets column structure so I can create the spreadsheet with correct headers before deploying.

**Acceptance Criteria:**
- [ ] `templates/project-sheet.md` defines two sheet tabs: Projects (columns A-O) and Materials (columns A-G)
- [ ] Projects columns: Project Name, Client, Client Email, Address, Bid, Your Cut (20%), Crew Pay (80%), Status, Materials Budget, Materials Spent, Materials Remaining, Crew Assigned, Start Date, End Date, Notes
- [ ] Materials columns: Project Name, Date, Store, Items, Amount, Submitted By, Receipt Image
- [ ] Status flow documented: NEW > OFFERED > ASSIGNED > IN_PROGRESS > COMPLETED > INVOICED > ARCHIVED
- [ ] Formulas noted where applicable (Your Cut = Bid * 0.20, etc.)

### US-008: Create WhatsApp message templates
**Description:** As a contractor deploying the system, I want pre-written WhatsApp message templates so the agent sends consistent, professional messages to crew for job offers, check-ins, receipt confirmations, and budget alerts.

**Acceptance Criteria:**
- [ ] `templates/crew-message.md` contains templates for: job offer, job offer reminder (24h), job confirmed, daily progress check-in, receipt acknowledged, budget warning (<10%), budget exceeded, project completion
- [ ] Each template uses `{placeholder}` variables for dynamic content
- [ ] Messages are concise (<1000 characters) and end with clear calls to action where applicable

### US-009: Create client invoice email template
**Description:** As a contractor deploying the system, I want an invoice email template so the agent sends professional, itemized invoices to clients upon project completion.

**Acceptance Criteria:**
- [ ] `templates/client-invoice.md` contains subject line template and email body template
- [ ] Includes itemized materials table format (date, store, items, amount)
- [ ] Explicitly notes: never show crew pay or contractor cut to clients
- [ ] Uses `{placeholder}` variables for dynamic content

### US-010: Expand AGENTS.md with architecture overview
**Description:** As a developer reviewing the repo, I want the root AGENTS.md to include an OpenClaw architecture overview so I understand how the workflow maps to OpenClaw components without reading every config file.

**Acceptance Criteria:**
- [ ] `AGENTS.md` retains the original mermaid flowchart (updated with tool names)
- [ ] New section: OpenClaw Architecture Overview with directory structure diagram
- [ ] New section: Workflow-to-OpenClaw Mapping table (10 rows: step, mechanism, tools)
- [ ] New section: Integration Dependency Matrix (which integrations are needed for which steps)
- [ ] New section: Configuration File Reference (table linking files to purposes)

### US-011: Rewrite README.md as complete setup guide
**Description:** As a contractor deploying the system for the first time, I want a comprehensive README with step-by-step setup instructions so I can go from a fresh DigitalOcean account to a running agent.

**Acceptance Criteria:**
- [ ] Overview section with 10-step workflow summary
- [ ] Prerequisites checklist (DigitalOcean account, Google account, Anthropic key, WhatsApp, SSH client)
- [ ] Quick Start section (5-line summary)
- [ ] Step 1: Deploy OpenClaw on DigitalOcean (Marketplace 1-Click, SSH setup, timezone)
- [ ] Step 2: Configure Google Workspace (Cloud Console project, enable APIs, OAuth credentials, publish to Production, install gog skill, SSH port-forwarding for headless OAuth)
- [ ] Step 3: Configure WhatsApp (channel login, QR scan, allowlist)
- [ ] Step 4: Deploy Agent Workspace (scp files, fill placeholders, create Google Sheet)
- [ ] Step 5: Set Up Cron Jobs (verify loaded, reload command)
- [ ] Testing & Verification section (per-integration smoke tests, end-to-end mock with $15K kitchen reno scenario, manual cron triggers)
- [ ] Placeholder Values Reference table (all `YOUR_*` values with descriptions and examples)
- [ ] Troubleshooting section (WhatsApp session drops, OAuth token refresh, headless auth, timezone issues, agent not processing emails, Sheets errors)
- [ ] Project Structure tree

## Functional Requirements

- FR-1: All config files must use `YOUR_*` placeholder values that match the `.env.example` reference
- FR-2: Cron schedules must use standard 5-field cron syntax and respect the configured timezone
- FR-3: The WhatsApp allowlist must be the sole access control for outbound messaging -- the agent must never message numbers not on the list
- FR-4: Financial calculations must use the formula: Contractor Cut = Bid * 0.20, Crew Pay = Bid * 0.80, with 2-decimal precision and verification that the sum equals the bid
- FR-5: The agent must ask the business owner (via WhatsApp or Slack) for clarification on any ambiguous situation rather than guessing or skipping
- FR-6: Client-facing communications must never include crew pay amounts or contractor cut percentages
- FR-7: The project tracking sheet must support the full status lifecycle: NEW > OFFERED > ASSIGNED > IN_PROGRESS > COMPLETED > INVOICED > ARCHIVED
- FR-8: The email monitor cron must mark processed emails as read to prevent duplicate processing
- FR-9: Receipt photos sent via WhatsApp must be processed for itemized data and logged against the project's materials budget
- FR-10: Budget alerts must trigger at <10% remaining and immediately on budget exceeded

## Non-Goals

- No custom application code, web dashboard, or API server -- this is config and documentation only
- No multi-tenant support -- designed for a single contractor
- No automated payment processing (Stripe, PayPal, etc.)
- No calendar/scheduling integration (crew availability tracking is manual)
- No automated bid estimation -- bids come from the client email
- No mobile app -- all interaction is via WhatsApp and email
- No backup/disaster recovery automation beyond OpenClaw's built-in memory
- No support for more than 3 concurrent projects in the initial version

## Technical Considerations

- **Platform**: OpenClaw on DigitalOcean (4GB/2vCPU Droplet, ~$24/mo)
- **LLM**: Anthropic Claude (via OpenClaw's Anthropic provider integration)
- **Headless OAuth**: Droplet has no browser; Google OAuth requires SSH port-forwarding (`ssh -L 8080:localhost:8080`) for the initial auth flow
- **WhatsApp session persistence**: Linked device sessions can expire; workspace instructions should include periodic connectivity checks
- **Google OAuth test mode**: Tokens expire after 7 days unless the app is published to Production in Google Cloud Console
- **Timezone**: Must be set on the Droplet OS level (`timedatectl set-timezone`) AND in the OpenClaw cron config; mismatch causes cron jobs to fire at wrong times
- **LLM arithmetic**: Financial calculations rely on the LLM; workspace instructions emphasize precision and verification formulas

## Success Metrics

- A contractor with no prior OpenClaw experience can deploy a working agent within 1 hour using only the README
- All 3 cron jobs fire at correct times (email check every 15 min, briefing at 7 AM, progress check at 4 PM weekdays)
- End-to-end smoke test passes: send test email > agent creates sheet row with correct 80/20 split > WhatsApp sent to crew > crew reply processed > receipt tracked > invoice generated
- Zero placeholder values remain after setup (all `YOUR_*` replaced with real values)
- Agent correctly refuses to message non-allowlisted numbers
- Agent asks the business owner for help (instead of guessing) when it encounters ambiguous project data

## Open Questions

- Should the system support Slack as an alternative to (or alongside) WhatsApp for contractor notifications?
- What is the fallback if the WhatsApp session drops mid-project and crew messages are missed?
- Should the materials budget be a fixed field from the scope doc, or should the contractor be able to adjust it after project creation?
- Is there a preferred format for receipt photos (e.g., must be clear enough for OCR), or should the agent always ask crew to type the amount as fallback?
- Should the morning briefing include projects in INVOICED status (awaiting payment) or only active projects?
