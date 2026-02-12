# Agent Operating Instructions

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

1. Calculate pay split:
   - **Your Cut** = Bid Amount x 0.20
   - **Crew Pay** = Bid Amount x 0.80
2. Append a new row to the tracking sheet using `gog sheets append`:
   - Use spreadsheet ID from config
   - Columns: Project Name, Client, Address, Bid, Your Cut, Crew Pay, Status (NEW), Materials Budget, Spent ($0.00), Remaining, Crew Assigned (--), Start Date, End Date
3. Store the row number in memory for this project
4. Proceed to **Crew Notification Protocol**

## 3. Crew Notification Protocol

After the project sheet is created:

1. For each crew member in the USER.md roster:
   a. Send a WhatsApp message using the `message` tool with:
      - Project name and address
      - Scope of work summary (2-3 sentences)
      - Crew pay amount
      - Estimated timeline
      - "Reply YES to accept or NO to pass"
2. Wait for replies (handled by per-peer DM sessions):
   - **YES**: Update sheet Status to ASSIGNED, Crew Assigned to responder's name, notify contractor
   - **NO**: Log the decline, continue waiting for other crew responses
   - **No reply after 24h**: Send reminder, notify contractor of non-response
3. If all crew decline, alert contractor via WhatsApp to find alternate crew

## 4. Material Tracking Protocol

When crew sends messages during an active project:

1. If the message contains a receipt image:
   a. Use `image` tool to extract receipt details (store, items, total)
   b. Update the project sheet: add to Spent column, recalculate Remaining
   c. Append receipt details to a materials log row
2. If Remaining drops below 10% of Materials Budget:
   - Alert contractor via WhatsApp: "Budget alert: [Project] has $X remaining of $Y materials budget"
3. If Spent exceeds Materials Budget:
   - Immediately alert contractor: "OVER BUDGET: [Project] materials spend is $X over the $Y budget"

## 5. Progress Monitoring Protocol

When the `progress-check` cron fires:

1. Read all active projects from the tracking sheet
2. For each project with status ASSIGNED or IN_PROGRESS:
   a. Send WhatsApp to assigned crew: "How's [Project] going? Any updates or receipts to share?"
   b. Process any replies (update sheet with progress notes)
3. For projects past their estimated end date:
   - Alert contractor: "[Project] is past the estimated completion date"

## 6. Completion Protocol

When crew reports a project is finished:

1. Update project sheet Status to COMPLETED
2. Read all materials data from the sheet
3. Draft an invoice email to the client using `gog mail send`:
   - Itemized materials list with costs
   - Total materials cost
   - Professional formatting (see templates/client-invoice.md)
4. Send WhatsApp to contractor with project summary:
   - Final crew pay amount
   - Total materials billed
   - Your cut amount
   - Any notes
5. Update project sheet Status to INVOICED
6. Save project summary to memory for future reference

## 7. Archive Protocol

After client payment is confirmed:

1. Update project sheet Status to ARCHIVED
2. Store final project record in memory:
   - Total revenue, costs, profit
   - Crew performance notes
   - Timeline adherence
3. Send final summary to contractor via WhatsApp
