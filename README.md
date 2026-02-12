# Contractor Workflow Automation

Automate your construction contracting business with an AI agent running on [OpenClaw](https://openclaw.com) + DigitalOcean. The agent monitors your email for new projects, creates tracking sheets, dispatches crew via WhatsApp, tracks material receipts, and sends invoices -- all autonomously.

**Cost**: ~$24/mo (DigitalOcean Droplet) + Anthropic API usage

## The 10-Step Workflow

1. **Monitor Emails** -- Agent checks Gmail every 15 minutes for new client messages
2. **Download Scope Doc** -- Extracts attachments or linked documents
3. **Create Project Sheet** -- Adds project to Google Sheets tracker with all details
4. **Calculate Pay** -- Splits bid: 20% to you, 80% to crew
5. **WhatsApp Crew** -- Sends job offer with details and pay to available crew
6. **Handle Replies** -- Crew accepts/declines via WhatsApp; agent updates tracker
7. **Track Materials** -- Crew sends receipt photos; agent logs costs against budget
8. **Monitor Progress** -- Daily 4 PM check-ins with active crews
9. **Invoice Client** -- Sends itemized materials invoice via email
10. **Archive & Report** -- Summarizes project financials, archives to memory

See [AGENTS.md](AGENTS.md) for the full architecture diagram and OpenClaw mapping.

## Prerequisites

- [ ] [DigitalOcean account](https://cloud.digitalocean.com/registrations/new)
- [ ] [Google account](https://accounts.google.com) (Gmail for business email)
- [ ] [Google Cloud Console](https://console.cloud.google.com) project (for OAuth)
- [ ] [Anthropic API key](https://console.anthropic.com)
- [ ] WhatsApp on your phone (for crew/client messaging)
- [ ] SSH client on your computer

## Quick Start

```bash
# 1. Clone this repo
git clone <this-repo-url> && cd contractor-workflow

# 2. Copy and fill in your environment variables
cp .env.example .env
# Edit .env with your actual values

# 3. Deploy OpenClaw on DigitalOcean (see Step 1 below)
# 4. Configure Google + WhatsApp (Steps 2-3)
# 5. Copy workspace and config to Droplet (Steps 4-5)
# 6. Run smoke tests (Testing section)
```

---

## Step 1: Deploy OpenClaw on DigitalOcean

### 1.1 Create the Droplet

1. Go to [DigitalOcean Marketplace](https://marketplace.digitalocean.com) and search for **OpenClaw**
2. Click **Create OpenClaw Droplet**
3. Select the **4 GB / 2 vCPU** plan (~$24/mo)
4. Choose a datacenter region close to you
5. Add your SSH key (or create one)
6. Click **Create Droplet**
7. Note your Droplet IP: `YOUR_DROPLET_IP`

### 1.2 Initial Setup

```bash
# SSH into your Droplet
ssh root@YOUR_DROPLET_IP

# Run the interactive setup wizard
openclaw setup

# When prompted:
# - LLM Provider: Select "Anthropic"
# - API Key: Paste YOUR_ANTHROPIC_API_KEY
# - Agent Name: "ContractorBot" (or your preference)
```

### 1.3 Set Timezone

```bash
# Set your timezone (critical for cron jobs)
timedatectl set-timezone YOUR_TIMEZONE
# Example: timedatectl set-timezone America/New_York

# Verify
date
```

---

## Step 2: Configure Google Workspace Integration

### 2.1 Create Google Cloud Project

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Click **Select a project** > **New Project**
3. Name it (e.g., "contractor-workflow") and click **Create**
4. Note the Project ID: `YOUR_GOOGLE_CLOUD_PROJECT_ID`

### 2.2 Enable APIs

In the Cloud Console, go to **APIs & Services > Library** and enable:

- Gmail API
- Google Sheets API
- Google Drive API
- Google Calendar API (optional, for scheduling)

### 2.3 Create OAuth Credentials

1. Go to **APIs & Services > Credentials**
2. Click **+ Create Credentials > OAuth client ID**
3. If prompted, configure the **OAuth consent screen** first:
   - User type: **External**
   - App name: "Contractor Workflow"
   - Support email: YOUR_GMAIL_ADDRESS
   - Scopes: Add Gmail, Sheets, Drive scopes
   - Test users: Add YOUR_GMAIL_ADDRESS
4. Back in Credentials, select **Desktop app** as application type
5. Click **Create**
6. Note: `YOUR_OAUTH_CLIENT_ID` and `YOUR_OAUTH_CLIENT_SECRET`

### 2.4 Publish to Production

> **Important**: In "test" mode, OAuth tokens expire every 7 days. You must publish to avoid constant re-authentication.

1. Go to **OAuth consent screen**
2. Click **Publish App**
3. Confirm the publishing dialog

### 2.5 Install gog Skill and Authenticate

```bash
# On your Droplet:
ssh root@YOUR_DROPLET_IP

# Install the gog (Google) skill
clawdhub install gog

# Start the OAuth flow
gog auth
```

The `gog auth` command will print a URL. Since the Droplet has no browser, use SSH port-forwarding:

```bash
# On your LOCAL machine (new terminal):
ssh -L 8080:localhost:8080 root@YOUR_DROPLET_IP

# Then open the URL in your local browser
# Complete the Google sign-in flow
# The token will be saved on the Droplet automatically
```

### 2.6 Verify Gmail Access

```bash
# On the Droplet:
gog mail list --unread --json
# Should return your unread emails (or empty array)
```

---

## Step 3: Configure WhatsApp Channel

### 3.1 Link WhatsApp

```bash
# On your Droplet:
openclaw channels login --channel whatsapp
```

This displays a QR code in the terminal. Scan it with your WhatsApp app:
1. Open WhatsApp on your phone
2. Go to **Settings > Linked Devices > Link a Device**
3. Scan the QR code in the terminal

### 3.2 Configure Allowlist

The allowlist restricts which phone numbers the agent can message. This is set in `config/openclaw.jsonc` under `channels.whatsapp.allowlist`. Ensure all crew and client numbers are listed in E.164 format (e.g., `+15551234567`).

### 3.3 Verify WhatsApp

```bash
# Send a test message to yourself
openclaw message send --channel whatsapp --to YOUR_PHONE_NUMBER --body "Test from ContractorBot"
```

---

## Step 4: Deploy Agent Workspace

### 4.1 Copy Workspace Files

```bash
# From your LOCAL machine, in the repo directory:
scp workspace/*.md root@YOUR_DROPLET_IP:/opt/openclaw/workspace/
```

### 4.2 Copy Config

```bash
# Copy main config (edit placeholders first!)
scp config/openclaw.jsonc root@YOUR_DROPLET_IP:/opt/openclaw/config/openclaw.jsonc

# Copy cron job definitions
scp config/cron/*.json root@YOUR_DROPLET_IP:/opt/openclaw/config/cron/
```

### 4.3 Fill In Placeholders

Before copying, replace all `YOUR_*` placeholders in the workspace and config files with your actual values from `.env`. You can do this locally before copying, or edit on the Droplet:

```bash
# On the Droplet, edit files as needed:
nano /opt/openclaw/config/openclaw.jsonc
nano /opt/openclaw/workspace/IDENTITY.md
nano /opt/openclaw/workspace/USER.md
```

### 4.4 Create the Project Tracking Sheet

1. Go to [Google Sheets](https://sheets.google.com) and create a new spreadsheet
2. Name it "Contractor Projects"
3. Create two tabs: **Projects** and **Materials**
4. Add headers from `templates/project-sheet.md` to each tab
5. Note the spreadsheet ID from the URL: `https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID/edit`
6. Update `YOUR_SHEET_ID` in the Droplet config

---

## Step 5: Set Up Cron Jobs

The cron job files are already deployed in Step 4. Verify they're loaded:

```bash
# On the Droplet:
openclaw cron list
```

Expected output:
| Name | Schedule | Status |
|------|----------|--------|
| email-monitor | `*/15 * * * *` | Enabled |
| morning-briefing | `0 7 * * *` | Enabled |
| progress-check | `0 16 * * 1-5` | Enabled |

If not listed, reload:
```bash
openclaw cron reload
```

---

## Testing & Verification

### Smoke Tests

Run these one at a time to verify each integration:

```bash
# Gmail -- should return emails (or empty array)
gog mail list --unread --json

# Google Sheets -- should return sheet data
gog sheets get --spreadsheet-id YOUR_SHEET_ID --range "Projects!A1:O1" --json

# WhatsApp -- should deliver a message to your phone
openclaw message send --channel whatsapp --to YOUR_PHONE_NUMBER --body "Smoke test"
```

### End-to-End Test (Mock Scenario)

Test with a mock $15,000 kitchen renovation:

1. **Send a test email** to YOUR_GMAIL_ADDRESS with subject "New Project: Kitchen Renovation" and body including:
   - Client: "Jane Smith"
   - Address: "123 Oak Street"
   - Bid: $15,000
   - Materials budget: $5,000
   - Scope: "Full kitchen renovation including cabinets, countertops, tile backsplash"

2. **Trigger the email monitor**:
   ```bash
   openclaw cron run email-monitor
   ```

3. **Verify** the project sheet has a new row with:
   - Bid: $15,000.00
   - Your Cut: $3,000.00
   - Crew Pay: $12,000.00
   - Status: OFFERED

4. **Check WhatsApp** -- crew members should receive job offer messages

5. **Reply YES** from a crew member's phone to test assignment

6. **Send a receipt photo** via WhatsApp to test materials tracking

7. **Trigger completion** and verify invoice email

### Manual Cron Triggers

```bash
openclaw cron run email-monitor
openclaw cron run morning-briefing
openclaw cron run progress-check
```

---

## Placeholder Values Reference

| Placeholder | Where Used | Example |
|---|---|---|
| `YOUR_DROPLET_IP` | SSH commands | `164.90.123.45` |
| `YOUR_ANTHROPIC_API_KEY` | `openclaw.jsonc` | `sk-ant-...` |
| `YOUR_GMAIL_ADDRESS` | `openclaw.jsonc`, `USER.md` | `contractor@gmail.com` |
| `YOUR_GOOGLE_CLOUD_PROJECT_ID` | `openclaw.jsonc` | `contractor-workflow-12345` |
| `YOUR_OAUTH_CLIENT_ID` | Google OAuth setup | `123456...apps.googleusercontent.com` |
| `YOUR_OAUTH_CLIENT_SECRET` | Google OAuth setup | `GOCSPX-...` |
| `YOUR_PHONE_NUMBER` | `openclaw.jsonc`, `USER.md` | `+15551234567` |
| `YOUR_CREW_MEMBER_1_PHONE` | `openclaw.jsonc`, `USER.md` | `+15559876543` |
| `YOUR_CREW_MEMBER_2_PHONE` | `openclaw.jsonc`, `USER.md` | `+15555551234` |
| `YOUR_CREW_MEMBER_3_PHONE` | `openclaw.jsonc`, `USER.md` | `+15555555678` |
| `YOUR_CLIENT_PHONE` | `openclaw.jsonc` | `+15550001111` |
| `YOUR_NAME` | `IDENTITY.md`, `USER.md` | `Mike Johnson` |
| `YOUR_BUSINESS_NAME` | `IDENTITY.md`, `USER.md` | `Johnson Contracting LLC` |
| `YOUR_TIMEZONE` | `openclaw.jsonc`, server setup | `America/New_York` |
| `YOUR_GATEWAY_TOKEN` | `openclaw.jsonc` | Generated during setup |
| `YOUR_SHEET_ID` | `openclaw.jsonc` | `1BxiMVs0XRA5nFMdKvBdBZjgmUUqptlbs74OgVE2upms` |

---

## Troubleshooting

### WhatsApp Session Drops

WhatsApp linked device sessions can expire. If the agent stops sending messages:

```bash
# Re-link WhatsApp
openclaw channels login --channel whatsapp
# Scan the new QR code from your phone
```

To prevent drops, the agent's workspace instructions include connectivity checks. You can also manually verify:

```bash
openclaw channels status --channel whatsapp
```

### OAuth Token Refresh Failures

If `gog` commands fail with auth errors:

1. **Check if app is published** -- Test mode tokens expire after 7 days
2. **Re-authenticate**:
   ```bash
   gog auth --force
   # Use SSH port-forwarding for the browser flow
   ```

### Headless OAuth (No Browser on Droplet)

The Droplet has no GUI browser. Always use SSH port-forwarding:

```bash
# Local machine:
ssh -L 8080:localhost:8080 root@YOUR_DROPLET_IP

# Then open the auth URL in your local browser
```

### Cron Jobs Firing at Wrong Time

```bash
# Verify timezone
timedatectl

# If wrong:
timedatectl set-timezone America/New_York  # your timezone

# Reload cron after timezone change
openclaw cron reload
```

### Agent Not Processing Emails

1. Check cron is running: `openclaw cron list`
2. Check for unread emails: `gog mail list --unread --json`
3. Check agent logs: `openclaw logs --tail 50`
4. Manually trigger: `openclaw cron run email-monitor`

### Google Sheets Errors

- Ensure the sheet ID is correct and the Google account has edit access
- Verify the sheet has the correct tab names ("Projects" and "Materials")
- Check that headers match the format in `templates/project-sheet.md`

---

## Project Structure

```
contractor-workflow/
  .env.example              # All placeholder values
  .gitignore                # Git ignore rules (keeps .env out of commits)
  AGENTS.md                 # Architecture overview + workflow mapping
  CLAUDE.md                 # Claude Code instructions
  Makefile                  # Development helpers (ralph, archive)
  README.md                 # This file
  config/
    openclaw.jsonc           # OpenClaw config template
    cron/
      email-monitor.json     # Every 15 min email check
      morning-briefing.json  # Daily 7 AM briefing
      progress-check.json    # Daily 4 PM crew check-in
  workspace/
    SOUL.md                  # Agent persona and principles
    IDENTITY.md              # Agent name and presentation
    USER.md                  # Contractor profile and crew roster
    AGENTS.md                # Workflow logic (all protocols)
    TOOLS.md                 # Tool usage conventions
  templates/
    project-sheet.md         # Google Sheets column reference
    crew-message.md          # WhatsApp message templates
    client-invoice.md        # Client invoice email template
```

## License

Private -- all rights reserved.
