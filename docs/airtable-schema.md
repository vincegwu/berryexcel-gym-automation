# 🗄 Airtable Schema

## Table: Members

This table stores all user records created during onboarding and updated during subscription renewals.

---

## Field Definitions

| Field Name | Field Type | Description |
|---|---|---|
| `Full Name` | Single Line Text | User's full name as submitted on the form |
| `Email` | Email | Primary identifier — used to locate records during renewal |
| `Phone Number` | Single Line Text | Accepts any format — typecast enabled in n8n |
| `Subscription` | Single Line Text | Subscription plan selected e.g. `1 Month`, `2 Months`, `3 Months` |
| `Start Date` | Single Line Text | Format: `dd MMM yyyy` e.g. `24 May 2026` |
| `End Date` | Single Line Text | Format: `dd MMM yyyy` e.g. `24 Jul 2026` |
| `Status` | Single Select | Current subscription status |
| `Profile-Picture` | Attachment | Stores imgBB hosted image URL as an attachment |

---

## Status Field Options

| Option | Colour | When Set |
|---|---|---|
| `Active` | Green | On onboarding form submission or subscription renewal |
| `Expired` | Red | When End Date passes today (set by Schedule Trigger daily) |
| `Pending` | Yellow | Optional — for manual use or future workflows |

---

## Important Notes

### Date Field Type
Start Date and End Date are stored as **Single Line Text** (not Airtable's native Date field) because n8n formats dates as `dd MMM yyyy` (e.g. `24 May 2026`). Airtable's native Date field only accepts ISO format (`2026-05-24`).

### Profile Picture Field
The field name uses a hyphen (`Profile-Picture`) not a space. In n8n expressions always reference it as:
```
$json['Profile-Picture']
```
And map it as an array of objects:
```
{{ [{ url: $('Upload Profile Image').item.json.data.url }] }}
```

### Email as Unique Identifier
The `Email` field is used by Workflow 2 (Renewal) to locate the correct Airtable record via the Filter by Formula:
```
{Email} = '{{ $json['Email'] }}'
```

### Accessing Fields from List Records
When data comes from the **Airtable List Records** node, fields are nested inside a `fields` object:
```
{{ $json.fields['Full Name'] }}     // correct
{{ $json['Full Name'] }}            // incorrect — returns undefined
```

---

## Airtable IDs

| ID Type | Format | Where to Find |
|---|---|---|
| Base ID | `appXXXXXXXXXXXXXX` | Browser URL or Airtable API docs |
| Table ID | `tblXXXXXXXXXXXXXX` | Browser URL (second segment) |
| Record ID | `recXXXXXXXXXXXXXX` | Available in `$json.id` from List/Search nodes |

---

## Personal Access Token (PAT) Scopes

When generating your Airtable PAT at `https://airtable.com/create/tokens`, enable these scopes:

| Scope | Required For |
|---|---|
| `data.records:read` | Listing and searching records |
| `data.records:write` | Creating and updating records |
| `schema.bases:read` | Loading field names in n8n |