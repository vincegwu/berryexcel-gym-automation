# 🔧 Setup Guide

Complete step-by-step instructions for deploying the BerryExcel Gym Subscription Management Automation system.

---

## Prerequisites

Before starting, ensure you have the following accounts and tools ready:

- [ ] **n8n** instance (cloud at `https://app.n8n.cloud` or self-hosted)
- [ ] **Airtable** account at `https://airtable.com`
- [ ] **imgBB** account at `https://imgbb.com`
- [ ] **Gmail** account for sending notifications
- [ ] **Telegram** account for owner alerts

---

## Step 1 — Set Up Airtable

### 1.1 Create a New Base
1. Log into Airtable
2. Click **"Add a base"**
3. Name it `BerryExcel Gym` (or your preferred name)

### 1.2 Create the Members Table
Add the following fields (see `airtable-schema.md` for full details):

| Field | Type |
|---|---|
| `Full Name` | Single Line Text |
| `Email` | Email |
| `Phone Number` | Single Line Text |
| `Subscription` | Single Line Text |
| `Start Date` | Single Line Text |
| `End Date` | Single Line Text |
| `Status` | Single Select |
| `Profile-Picture` | Attachment |

### 1.3 Configure Status Options
In the `Status` Single Select field, add:
- `Active` (Green)
- `Expired` (Red)
- `Pending` (Yellow)

### 1.4 Get Your Airtable Credentials
1. Go to `https://airtable.com/create/tokens`
2. Click **"Create new token"**
3. Enable scopes: `data.records:read`, `data.records:write`, `schema.bases:read`
4. Copy the Personal Access Token (PAT)
5. Get your Base ID and Table ID from the browser URL:
```
https://airtable.com/appXXXXXXXXXXXXXX/tblYYYYYYYYYYYYYY/...
```

---

## Step 2 — Set Up imgBB

1. Sign up at `https://imgbb.com`
2. Verify your email address
3. Go to **Settings → API**
4. Click **"Get API Key"**
5. Copy the API key
6. Add it as a query parameter `key` in the imgBB HTTP Request node in n8n

> **Security:** Store the API key in n8n Credentials (Generic Credential → Query Auth) rather than hardcoding it in the node URL.

---

## Step 3 — Set Up Gmail OAuth2

1. In n8n, go to **Credentials → New**
2. Search for **"Gmail OAuth2"**
3. Follow the OAuth2 setup flow:
   - Go to Google Cloud Console
   - Create a new project
   - Enable the Gmail API
   - Create OAuth2 credentials
   - Add the n8n callback URL as an authorised redirect URI
4. Connect your Gmail account
5. Test the credential

---

## Step 4 — Set Up Telegram Bot

### 4.1 Create the Bot
1. Open Telegram and search for **@BotFather**
2. Send `/newbot`
3. Choose a name for your bot (e.g. `BerryExcel Alerts`)
4. Choose a username ending in `bot` (e.g. `berryexcel_alerts_bot`)
5. Copy the **Bot Token** provided

### 4.2 Get Your Chat ID
1. Search for **@userinfobot** on Telegram
2. Send `/start`
3. Copy your **Chat ID**

### 4.3 Start the Bot
Before any messages can be sent, the owner must start the bot:
1. Search for your bot name on Telegram
2. Click **Start** or send `/start`

### 4.4 Add Credentials in n8n
1. Go to **Credentials → New**
2. Search for **"Telegram API"**
3. Paste your **Bot Token**
4. Save

---

## Step 5 — Import Workflows into n8n

1. Open your n8n instance
2. Go to **Workflows → Import from File**
3. Import in this order:
   - `workflow-3-error-handler.json` (must be imported and activated first)
   - `workflow-1-onboarding-alert.json`
   - `workflow-2-renewal-update.json`

---

## Step 6 — Configure Workflow Credentials

After importing, update credentials in each node:

### Workflow 1
| Node | Credential |
|---|---|
| Upload Profile Image (imgBB) | imgBB API Key |
| Edit Airtable Fields | Airtable PAT |
| Airtable List Records | Airtable PAT |
| Gmail Welcome Email | Gmail OAuth2 |
| Gmail Expiry Alert | Gmail OAuth2 |
| Gmail Expired Notice | Gmail OAuth2 |
| Telegram Owner Alert | Telegram API |

### Workflow 2
| Node | Credential |
|---|---|
| Airtable Search Record | Airtable PAT |
| Airtable Update Record | Airtable PAT |
| Gmail Renewal Confirmation | Gmail OAuth2 |
| Telegram Owner Alert | Telegram API |

### Workflow 3
| Node | Credential |
|---|---|
| Gmail Error Report | Gmail OAuth2 |
| Telegram Error Alert | Telegram API |

---

## Step 7 — Update Node Configurations

### imgBB HTTP Request Node
| Setting | Value |
|---|---|
| URL | `https://api.imgbb.com/1/upload` |
| Query Parameter `key` | `YOUR_IMGBB_API_KEY` |
| Body Parameter `image` | n8n Binary Data → `data` |

### Telegram Nodes
Replace `YOUR_CHAT_ID` with your actual Chat ID in all Telegram nodes.

### Gmail Nodes
Replace `support@yourcompany.com` with your actual support email address.

### Renewal Form URL
In `expiry-alert-email.html` and `expired-notice-email.html`, replace `YOUR_RENEWAL_FORM_URL` with your Workflow 2 production form URL.

---

## Step 8 — Link Error Handler Workflow

For both Workflow 1 and Workflow 2:
1. Open the workflow canvas
2. Click the **Settings** gear icon (top right)
3. Find **"Error Workflow"**
4. Select **"Workflow 3 — Error Handler"**
5. Click **Save**

> The Error Handler must be **activated** before it appears in the dropdown.

---

## Step 9 — Activate All Workflows

Activate in this exact order:

1. **Workflow 3** (Error Handler) — activate first
2. **Workflow 1** (Onboarding + Alert) — activate second
3. **Workflow 2** (Renewal Update) — activate third

Toggle the **Activate** switch to ON (top right of canvas) for each workflow.

---

## Step 10 — Share Form URLs

After activation, get the production URLs for each form:

### Onboarding Form URL
1. Open Workflow 1
2. Click the **n8n Form Trigger** node
3. Copy the **Production URL**
4. Share with new users

### Renewal Form URL
1. Open Workflow 2
2. Click the **n8n Form Trigger** node
3. Copy the **Production URL**
4. Share with existing users via expiry alert emails

---

## Step 11 — Test the Full System

### Test Onboarding
1. Open the Onboarding Form URL
2. Fill in all fields and upload a profile picture
3. Submit the form
4. Verify:
   - [ ] Record created in Airtable with Status = Active
   - [ ] Profile picture visible in Airtable
   - [ ] Welcome email received with correct details
   - [ ] Image visible in email

### Test Subscription Alert
1. Create a test record in Airtable with End Date = 3 days from today
2. Manually execute the Schedule Trigger node
3. Verify:
   - [ ] Expiry alert email received
   - [ ] Telegram owner alert received

### Test Expired Status Update
1. Create a test record with End Date = yesterday
2. Manually execute the Schedule Trigger node
3. Verify:
   - [ ] Status updated to Expired in Airtable
   - [ ] Expired notice email received
   - [ ] Telegram owner alert received

### Test Renewal
1. Open the Renewal Form URL
2. Submit with an existing user's email
3. Verify:
   - [ ] Airtable record updated with new dates and Status = Active
   - [ ] Renewal confirmation email received
   - [ ] Telegram owner alert received

### Test Error Handler
1. Temporarily enter a wrong API key in any node
2. Execute the workflow
3. Verify:
   - [ ] Error report email received by owner
   - [ ] Telegram error alert received
4. Restore the correct API key

---

## Troubleshooting

See `troubleshooting.md` for common errors and fixes.