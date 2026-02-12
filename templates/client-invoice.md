# Client Invoice Email Template

## Subject Line

```
Invoice: {project_name} - {business_name}
```

## Email Body

```
Hello {client_name},

The work on {project_name} at {address} has been completed.

Below is the itemized summary of materials purchased for your project:

MATERIALS
---------
{materials_table}

Materials Total: ${materials_total}

If you have any questions about the charges above, please reply to this email or call {contractor_name} at {contractor_phone}.

Thank you for choosing {business_name}.

Best regards,
{contractor_name}
{business_name}
{contractor_phone}
{gmail_address}
```

## Materials Table Format

```
Date       | Store       | Items              | Amount
-----------|-------------|--------------------|---------
2025-03-15 | Home Depot  | Lumber, screws     | $342.18
2025-03-17 | Lowe's      | Paint, brushes     | $128.45
2025-03-20 | Home Depot  | Tile, grout        | $567.00
-----------|-------------|--------------------|---------
                                    TOTAL:     | $1,037.63
```

## Notes

- Only include materials charges in client invoices
- Crew pay and contractor cut are NEVER shown to the client
- The bid amount is the agreed project price -- it is not itemized on the invoice
- Materials are billed at actual cost (receipt totals)
- Attach receipt images if available
