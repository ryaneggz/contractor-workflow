# Project Tracking Sheet - Column Reference

## Sheet Name: `Projects`

| Column | Header | Format | Description |
|--------|--------|--------|-------------|
| A | Project Name | Text | Short descriptive name (e.g., "Smith Kitchen Reno") |
| B | Client | Text | Client full name |
| C | Client Email | Text | Client email address |
| D | Address | Text | Project site address |
| E | Bid | Currency | Total bid amount (e.g., $15,000.00) |
| F | Your Cut (20%) | Currency | `= E * 0.20` -- Contractor's share |
| G | Crew Pay (80%) | Currency | `= E * 0.80` -- Crew payment |
| H | Status | Text | One of: NEW, OFFERED, ASSIGNED, IN_PROGRESS, COMPLETED, INVOICED, ARCHIVED |
| I | Materials Budget | Currency | Estimated materials cost from scope doc |
| J | Materials Spent | Currency | Running total of receipt amounts |
| K | Materials Remaining | Currency | `= I - J` |
| L | Crew Assigned | Text | Name of assigned crew member (or "--" if unassigned) |
| M | Start Date | Date | Project start date (YYYY-MM-DD) |
| N | End Date | Date | Estimated or actual end date (YYYY-MM-DD) |
| O | Notes | Text | General notes, updates, flags |

## Status Flow

```
NEW --> OFFERED --> ASSIGNED --> IN_PROGRESS --> COMPLETED --> INVOICED --> ARCHIVED
```

- **NEW**: Project just extracted from email, sheet row created
- **OFFERED**: WhatsApp sent to crew, awaiting response
- **ASSIGNED**: Crew accepted, ready to start
- **IN_PROGRESS**: Work has begun, materials being tracked
- **COMPLETED**: Crew reports work finished
- **INVOICED**: Invoice sent to client
- **ARCHIVED**: Payment received, project closed

## Materials Log Sheet: `Materials`

| Column | Header | Format | Description |
|--------|--------|--------|-------------|
| A | Project Name | Text | Links to Projects sheet |
| B | Date | Date | Receipt date |
| C | Store | Text | Where purchased (e.g., Home Depot) |
| D | Items | Text | Brief description of items |
| E | Amount | Currency | Receipt total |
| F | Submitted By | Text | Crew member who sent the receipt |
| G | Receipt Image | Text | Reference to stored image (if available) |
