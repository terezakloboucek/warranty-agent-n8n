# Airtable schema

Create one base (e.g. "Warranties") with three tables. **Table and field names must match exactly.**

## Table `Settings` (one row)

| Field | Type |
|---|---|
| Full name | Single line text (primary) |
| Address | Long text |
| Email | Email |
| Phone | Phone number |
| Bank account | Single line text |
| Telegram user ID | Single line text |

Fill in one row. Get your **Telegram user ID** by messaging `@userinfobot` in Telegram. The other fields are not used yet; they are reserved for pre-filling warranty claim forms in a future version.

## Table `Receipts`

| Field | Type |
|---|---|
| Store | Single line text (primary) |
| Store type | Single select (`online`, `physical`, `unknown`) |
| Purchase date | Date (ISO format) |
| Receipt number | Single line text |
| Total | Number (2 decimals) or Currency |
| Currency | Single line text |
| File | Attachment |
| Telegram file ID | Single line text |
| Products | Link to `Products` (created automatically when you add the `Receipt` link field in Products) |

## Table `Products`

| Field | Type |
|---|---|
| Name | Single line text (primary) |
| Name on receipt | Single line text |
| Manufacturer | Single line text |
| Model | Single line text |
| Category | Single select (`appliance`, `electronics`, `tools`, `clothing`, `footwear`, `household`, `sport`, `other`) |
| Description | Long text |
| Price | Number (2 decimals) or Currency |
| Quantity | Number (integer) |
| Store | Single line text |
| Purchase date | Date (ISO format) |
| Warranty (months) | Number (integer) |
| Warranty source | Single select (`Receipt`, `Statutory (default)`, `To verify`) |
| Warranty until | Formula: `DATEADD({Purchase date}, {Warranty (months)}, 'months')` |
| Days left | Formula: `DATETIME_DIFF({Warranty until}, TODAY(), 'days')` |
| Validity | Formula: `IF({Days left} < 0, "Expired", "Valid")` |
| Status | Single select (`Active`, `Claimed`, `Retired`) |
| Needs photo | Checkbox |
| Product photo | Attachment |
| Notified 6m | Checkbox |
| Notified 1m | Checkbox |
| Notified 1w | Checkbox |
| Receipt | Link to `Receipts` |
| Note | Long text |

The base ID is in the base URL: `https://airtable.com/appXXXXXXXXXXXXXX/...`. It starts with `app`.
