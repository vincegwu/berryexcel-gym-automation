# 🐛 Troubleshooting Guide

Common errors encountered when building and running the BerryExcel Gym Subscription Management Automation, with causes and fixes.

---

## Workflow Errors

### "Execution Stopped"
**Cause:** A node returned no data — n8n stops execution when a node outputs zero items.

**Fix:**
1. Go to **Executions** in the sidebar
2. Open the failed execution
3. Find the node highlighted in grey or orange
4. Check its **Input tab** — if empty, the previous node sent no data
5. Fix the previous node and re-execute

**Common causes:**
- IF node condition returns false on all records
- Airtable Search returns no matching record
- Code node returns empty array `[]`

---

### "Problem in node — Bad request"
**Cause:** The API received the request but rejected the payload.

**Fix:**
- Verify the API key is correct and not expired
- Check all required parameters are present
- Confirm the URL has no typos

---

### "Unexpected String" in Code Node
**Cause:** Unicode characters (emojis, special arrows like `←`) inside code comments or expressions.

**Fix:** Remove all special Unicode characters from code. Use only standard ASCII:
```javascript
// correct comment style — no special characters
binary: item.binary
```

---

### "Cannot read properties of undefined"
**Cause:** A field name doesn't match the actual JSON structure.

**Fix:** Add this diagnostic to your Code node to inspect available fields:
```javascript
const item = $input.first();
return [{ json: { allKeys: Object.keys(item.json) } }];
```

---

## Image Upload Errors

### "The item has no binary field 'data'"
**Cause:** Binary property name from Form Trigger doesn't match what the imgBB node expects.

**Fix:** Update the Code node to dynamically rename the binary property:
```javascript
const binaryKey = Object.keys(item.binary || {})[0];
const renamedBinary = binaryKey ? { data: item.binary[binaryKey] } : {};

return [{
  json: { ...item.json },
  binary: renamedBinary
}];
```

---

### "File not found" in imgBB node
**Cause:** Binary data is empty — the file content was not preserved through the Code node.

**Fix:** Ensure `binary: item.binary` is present in the Code node return statement:
```javascript
return [{
  json: { ...item.json },
  binary: item.binary   // this line is critical
}];
```

---

### Profile picture not showing in email
**Cause:** imgBB blocks hotlinking from email clients.

**Fix:** Use `data.image.url` instead of `data.url`:
```html
<img src="{{ $('Upload Profile Image').item.json.data.image.url }}" />
```

---

### Profile picture not showing in Airtable
**Cause:** URL passed as plain string instead of array of objects.

**Fix:** Wrap the URL correctly:
```
{{ [{ url: $('Upload Profile Image').item.json.data.url }] }}
```

---

## Airtable Errors

### "Invalid input for 'Profile-Picture' — expects array"
**Cause:** Attachment field received a plain string URL.

**Fix:**
```
{{ [{ url: $('Upload Profile Image').item.json.data.url }] }}
```

---

### "Field cannot accept the provided value — consider Typecast"
**Cause:** Phone Number or other field received an incompatible format.

**Fix:** Enable Typecast in Airtable node Options, or clean the value in a Set node:
```
{{ $json['Phone Number'].replace(/[^0-9+]/g, '') }}
```

---

### Status field shows "200" instead of "Active"
**Cause:** Code node is positioned after the imgBB HTTP Request node — `...item.json` spreads the HTTP response fields including `status: 200`.

**Fix:** Move the Code node to **before** the imgBB HTTP Request node in the workflow.

---

### Fields return `undefined` from Airtable List Records
**Cause:** List Records nests fields inside a `fields` object.

**Fix:**
```javascript
// Wrong
$json['Full Name']

// Correct
$json.fields['Full Name']
```

---

## Date and Expression Errors

### "Invalid DateTime" from Luxon
**Cause:** Date string format doesn't match the parser being used.

**Fix:** Use `DateTime.fromJSDate()` for maximum compatibility:
```
{{ DateTime.fromJSDate(new Date($json['End Date'])).plus({ months: 2 }).toFormat('dd MMM yyyy') }}
```

---

### End Date always returns same value regardless of subscription
**Cause:** Typo in Luxon — used `month` (singular) instead of `months` (plural).

**Fix:**
```javascript
// Wrong — silently ignored
$now.plus({ month: 2 })

// Correct
$now.plus({ months: 2 })
```

---

### IF node never triggers subscription alert
**Cause 1:** Date format mismatch between Airtable stored value and IF node comparison value.

**Fix:** Use ISO date comparison to eliminate format issues:
```
{{ DateTime.fromFormat($json.fields['End Date'], 'dd/M/yyyy').toISODate() }}
```

**Cause 2:** Schedule Trigger hasn't fired yet during testing.

**Fix:** Manually click **"Execute Node"** on the Schedule Trigger to test immediately.

---

## Telegram Errors

### "Chat not found"
**Cause 1:** The owner has not started the bot.
**Fix:** Search for the bot on Telegram and send `/start`.

**Cause 2:** Chat ID is incorrect.
**Fix:** Get the correct Chat ID from @userinfobot or via the getUpdates API:
```
https://api.telegram.org/botYOUR_BOT_TOKEN/getUpdates
```

---

## Error Workflow Issues

### Error Workflow greyed out in Settings
**Cause:** Error Handler workflow is not activated.

**Fix:**
1. Open the Error Handler workflow
2. Toggle Activate to ON
3. Return to main workflow Settings
4. The Error Handler now appears as selectable

---

### Error Trigger not firing
**Cause:** Error Workflow not linked to the main workflow.

**Fix:**
1. Open the main workflow
2. Click Settings gear icon
3. Set Error Workflow to the Error Handler
4. Save

---

## imgBB API Errors

### Error Code 103 — "Forbidden"
**Cause:** API key is blocked, expired, or account email not verified.

**Fix:**
1. Verify email at imgbb.com
2. Regenerate API key in Settings → API
3. Update the key in n8n

> Never share your imgBB API key publicly — regenerate immediately if exposed.

---

## General Tips

### Always Execute Nodes in Order
Never test a downstream node in isolation — always run upstream nodes first to ensure data flows correctly:
```
Form Trigger → Code Node → imgBB → Airtable → Gmail
```

### Use "Execute Previous Nodes"
Inside any node, click **"Execute previous nodes"** to run the full upstream chain with one click.

### Check Binary Tab
When debugging image issues, always check the **Binary tab** on each node's output to confirm binary data is present and correctly named.

### Use Exact Node Names in Expressions
When referencing a specific node's output, use the exact name as it appears on the canvas:
```
{{ $('Upload Profile Image').item.json.data.url }}
```
Node names are case-sensitive and space-sensitive.