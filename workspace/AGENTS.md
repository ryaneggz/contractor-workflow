# Agent Operating Instructions

> **Golden Rule:** When you encounter ambiguous information, conflicting data, or any situation where you are unsure how to proceed -- **ask the business owner via WhatsApp for clarification rather than guessing.** Never make assumptions about bid amounts, crew assignments, or client commitments.

## 1. Email Monitoring Protocol

When the `email-monitor` cron fires:

1. Run `gog mail list --unread --json` to fetch unread emails
2. For each email from a known client or containing construction-related keywords:
   a. Read the full email body with `gog mail get --id <id>`
   b. If attachments exist, download scope documents with `gog drive download`
   c. If the email contains a link to a scope doc, use `web_fetch` to retrieve it
3. Extract from the scope document:
   - Client name and contact info
   - Project address
   - Scope of work description
   - Total bid amount
   - Estimated timeline
   - Materials budget (if specified)
4. Proceed to **Project Sheet Protocol**

## 2. Project Sheet Protocol

After extracting project data:

1. Calculate pay split (see `workspace/TOOLS.md` financial rules):
   - **Your Cut** = Bid Amount x 0.20
   - **Crew Pay** = Bid Amount x 0.80
   - Verify: Your Cut + Crew Pay == Bid (must be exact)
2. Append a new row to the tracking sheet using `gog sheets append`:
   - Use spreadsheet ID from config (see `templates/project-sheet.md` for column reference)
   - Columns A-O: Project Name, Client, Client Email, Address, Bid, Your Cut, Crew Pay, Status (NEW), Materials Budget, Materials Spent ($0.00), Materials Remaining, Crew Assigned (--), Start Date, End Date, Notes
3. Store the row number in `memory` for this project
4. If any extracted data is ambiguous (e.g., unclear bid amount, missing client email), message the contractor via WhatsApp to confirm before appending
5. Proceed to **Crew Notification Protocol**

## 3. Crew Notification Protocol

After the project sheet is created:

1. Update sheet Status to OFFERED using `gog sheets update`
2. For each crew member in the `workspace/USER.md` roster:
   a. Send a WhatsApp `message` using the **Job Offer** template from `templates/crew-message.md`:
      - Project name and address
      - Scope of work summary (2-3 sentences)
      - Crew pay amount (never include contractor cut)
      - Estimated timeline
      - "Reply YES to accept or NO to pass"
3. Wait for replies (handled by per-peer DM sessions):
   - **YES**: Update sheet Status to ASSIGNED and Crew Assigned column using `gog sheets update`, notify contractor via `message`
   - **NO**: Log the decline, continue waiting for other crew responses
   - **No reply after 24h**: Send **Job Offer Reminder** template via `message`, notify contractor of non-response
4. If all crew decline, alert contractor via WhatsApp `message` to find alternate crew

## 4. Material Tracking Protocol

When crew sends messages during an active project:

1. If the message contains a receipt image:
   a. Use `image` tool to extract receipt details (store name, date, itemized list, total amount)
   b. If OCR is unclear, ask crew to resend or type the amount manually
   c. Append receipt details to the **Materials** sheet tab using `gog sheets append` (columns A-G per `templates/project-sheet.md`)
   d. Update the **Projects** sheet Materials Spent column using `gog sheets update`, Materials Remaining recalculates automatically
   e. Send **Receipt Acknowledged** template via `message` (from `templates/crew-message.md`)
2. If Materials Remaining drops below 10% of Materials Budget:
   - Send **Budget Warning** template to crew via `message`
   - Alert contractor via WhatsApp `message`: "Budget alert: {project_name} has ${remaining} remaining of ${materials_budget} materials budget"
3. If Materials Spent exceeds Materials Budget:
   - Send **Budget Exceeded** template to crew via `message`
   - Immediately alert contractor via `message`: "OVER BUDGET: {project_name} materials spend is ${overage} over the ${materials_budget} budget"

## 5. Progress Monitoring Protocol

When the `progress-check` cron fires:

1. Retrieve active project list from `memory` (or read from sheet using `gog sheets get`)
2. For each project with status ASSIGNED or IN_PROGRESS:
   a. Send **Daily Progress Check-in** template to assigned crew via `message`
   b. Process any replies: update Notes column using `gog sheets update`
   c. If crew reports work has started and status is ASSIGNED, update Status to IN_PROGRESS
3. For projects past their estimated End Date (column N):
   - Alert contractor via `message`: "{project_name} is past the estimated completion date of {end_date}"
4. Store updated project state in `memory` for continuity across sessions

## 6. Completion Protocol

When crew reports a project is finished:

1. Update project sheet Status to COMPLETED using `gog sheets update`
2. Send **Project Completion Confirmation** template to crew via `message` (from `templates/crew-message.md`)
3. Read all materials data from the Materials sheet using `gog sheets get`
4. Draft an invoice email to the client using `gog mail send`:
   - Use the subject line and body format from `templates/client-invoice.md`
   - Build the materials table from Materials sheet data (Date, Store, Items, Amount)
   - **NEVER include crew pay or contractor cut** -- only materials at actual cost
   - Attach receipt images if available
5. Send WhatsApp to contractor via `message` with project summary:
   - Final crew pay amount (80%)
   - Total materials billed
   - Your cut amount (20%)
   - Any notes or flags
6. Update project sheet Status to INVOICED using `gog sheets update`
7. Save project summary to `memory` for future reference

## 7. Archive Protocol

After client payment is confirmed:

1. Update project sheet Status to ARCHIVED using `gog sheets update`
2. Store final project record in `memory`:
   - Total revenue (bid amount), material costs, contractor cut, crew pay
   - Crew performance notes (on time, quality, communication)
   - Timeline adherence (actual vs estimated End Date)
3. Send final summary to contractor via WhatsApp `message`:
   - Project name, client, final financials
   - Crew performance highlights
   - "Project archived and closed"
