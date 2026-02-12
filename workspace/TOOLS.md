# Tool Usage Conventions

## gog (Google Workspace)

### Gmail
- `gog mail list --unread --json` -- List unread emails as JSON
- `gog mail get --id <message-id>` -- Read a specific email
- `gog mail send --to <email> --subject "<subject>" --body "<body>"` -- Send an email
- Always mark processed emails as read to avoid reprocessing

### Google Sheets
- `gog sheets get --spreadsheet-id <id> --range "<range>" --json` -- Read sheet data
- `gog sheets append --spreadsheet-id <id> --range "<range>" --values '<json>'` -- Append rows
- `gog sheets update --spreadsheet-id <id> --range "<range>" --values '<json>'` -- Update cells
- Always use the spreadsheet ID from config, never hardcode
- Use A1 notation for ranges (e.g., "Sheet1!A:M")

### Google Drive
- `gog drive download --file-id <id> --dest <path>` -- Download attachments
- Save downloaded files to `/tmp/` for processing, clean up after

## WhatsApp (message tool)

- Only message numbers in the configured allowlist
- Keep messages under 1000 characters for readability
- Use line breaks for structured information
- **Cross-role confidentiality:** never share financial details across roles
  - Crew must NOT see the contractor's cut (20%)
  - Clients must NOT see crew pay (80%) or the contractor's cut
  - Only share the total bid amount with clients; only share crew pay with crew
- For job offers, always end with a clear call to action ("Reply YES to accept")
- Wait for crew responses before sending follow-ups (minimum 2 hours between messages)

## Financial Calculations

- Always compute to 2 decimal places: `round(amount, 2)`
- Contractor cut: `bid * 0.20`
- Crew pay: `bid * 0.80`
- Remaining budget: `materials_budget - total_spent`
- Verify: `contractor_cut + crew_pay == bid` (must be exact)
- Never round intermediate values -- only round the final display value
- All amounts in USD

## Memory

- Use `memory` tool to persist project context across sessions
- Store: project IDs, sheet row numbers, crew assignments, key dates
- Retrieve before processing to maintain continuity

## web_fetch

- Use to download scope documents from links in emails
- Timeout after 30 seconds
- Save content for extraction, don't store raw HTML long-term

## image

- Use to process receipt photos from crew WhatsApp messages
- Extract: store name, date, itemized list, total amount
- If OCR is unclear, ask crew to resend or type the amount manually
