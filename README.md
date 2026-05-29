# 🏋️ BerryExcel Gym — Subscription Management Automation

![n8n](https://img.shields.io/badge/Built%20With-n8n-FF6B6B?style=for-the-badge&logo=n8n)
![Airtable](https://img.shields.io/badge/Database-Airtable-18BFFF?style=for-the-badge&logo=airtable)
![Gmail](https://img.shields.io/badge/Email-Gmail-EA4335?style=for-the-badge&logo=gmail)
![Telegram](https://img.shields.io/badge/Alerts-Telegram-26A5E4?style=for-the-badge&logo=telegram)
![imgBB](https://img.shields.io/badge/Images-imgBB-72B626?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

> A fully automated, production-grade subscription management system built entirely on **n8n** — no backend code required. Handles user onboarding, subscription monitoring, renewal updates, and error detection end-to-end.

---

## 📋 Table of Contents

- [Overview](#-overview)
- [System Architecture](#-system-architecture)
- [Workflows](#-workflows)
  - [Workflow 1 — Onboarding + Subscription Alert](#workflow-1--onboarding--subscription-alert)
  - [Workflow 2 — Subscription Renewal Update](#workflow-2--subscription-renewal-update)
  - [Workflow 3 — Error Handler](#workflow-3--error-handler)
- [Tech Stack](#-tech-stack)
- [Prerequisites](#-prerequisites)
- [Setup Guide](#-setup-guide)
- [Environment Variables](#-environment-variables)
- [Airtable Schema](#-airtable-schema)
- [Key Learnings](#-key-learnings)
- [Folder Structure](#-folder-structure)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🚀 Overview

This project automates the **entire subscription lifecycle** for BerryExcel Gym — from the moment a user registers to subscription expiry, renewal, and error monitoring. Everything runs automatically without any manual intervention.

### What Gets Automated

| Event | Action |
|---|---|
| New user registers | Profile uploaded, record saved, welcome email sent |
| Subscription expiring in 3 days | User alerted via email, owner notified via Telegram |
| Subscription expires | Status updated to Expired, user and owner notified |
| User renews subscription | Record updated, confirmation email sent |
| Any workflow fails | Owner alerted instantly via Gmail and Telegram |

---

## 🏗 System Architecture

```
┌─────────────────────────────────────────────────────────┐
│              WORKFLOW 1 — Berrygym Onboarding             │
│                                                         │
│  [Form Trigger] → [Code Node] → [imgBB Upload]          │
│  → [Airtable Create] → [Gmail Welcome]                  │
│                                                         │
│  [Schedule Trigger 6AM] → [Airtable List]               │
│  → [IF Expiring in 3 days] → [Gmail Alert] + [Telegram] │
│  → [IF Expired] → [Airtable Update] → [Gmail] + [Telegram]│
└─────────────────────────────────────────────────────────┘
                          │
                          │ linked via Error Workflow
                          ▼
┌─────────────────────────────────────────────────────────┐
│              WORKFLOW 3 — Error Handler                  │
│                                                         │
│  [Error Trigger] → [Set Error Details]                  │
│  → [Gmail Error Report] + [Telegram Alert]              │
└─────────────────────────────────────────────────────────┘
                          ▲
                          │ linked via Error Workflow
┌─────────────────────────────────────────────────────────┐
│           WORKFLOW 2 — Gym Subscription Renewal              │
│                                                         │
│  [Renewal Form] → [Code Node] → [Airtable Search]       │
│  → [Airtable Update] → [Gmail Confirmation] + [Telegram]│
└─────────────────────────────────────────────────────────┘
```

---

## ⚙️ Workflows

### Workflow 1 — Onboarding + Subscription Alert

**Trigger:** n8n Form Trigger (onboarding) + Schedule Trigger (daily 8AM)

#### Onboarding Path

```
[n8n Form Trigger]
      ↓
[Code Node - Set Subscription Dates]
  • Calculates Start Date (today)
  • Calculates End Date (1, 2, or 3 months)
  • Sets Status = Active
  • Preserves binary image data
      ↓
[HTTP Request - imgBB Upload]
  • Uploads profile picture to imgBB
  • Returns public image URL
      ↓
[Airtable - Create Record]
  • Saves all user fields
  • Attaches profile picture URL
      ↓
[Gmail - Welcome Email]
  • Sends branded HTML welcome email
  • Includes subscription details and status
```

#### Subscription Alert Path

```
[Schedule Trigger - 8AM Daily]
      ↓
[Airtable - List All Records]
      ↓
      ├── [IF End Date = 3 days AND Status = Active]
      │         ↓
      │   [Set Node - Extract First Name]
      │         ↓
      │   [Gmail - Expiry Alert] + [Telegram - Owner Alert]
      │
      └── [IF End Date < Today AND Status = Active]
                ↓
          [Airtable - Update Status to Expired]
                ↓
          [Gmail - Expired Notice] + [Telegram - Owner Alert]
```

---

### Workflow 2 — Subscription Renewal Update

**Trigger:** n8n Form Trigger (renewal form)

```
[n8n Renewal Form Trigger]
      ↓
[Code Node - Calculate New Dates]
  • New Start Date = today
  • New End Date = today + selected plan
  • Status = Active
      ↓
[Airtable - Search Record by Email]
      ↓
[Airtable - Update Record]
  • Updates Start Date, End Date, Subscription, Status
      ↓
[Gmail - Renewal Confirmation] + [Telegram - Owner Alert]
```

---

### Workflow 3 — Error Handler

**Trigger:** Error Trigger (fires automatically on failure in Workflow 1 or 2)

```
[Error Trigger]
      ↓
[Set Node - Format Error Details]
  • Error Message
  • Workflow Name
  • Failed Node Name
  • Execution ID
  • Timestamp
      ↓
[Gmail - Error Report] + [Telegram - Instant Alert]
```

---

## 🛠 Tech Stack

| Tool | Purpose |
|---|---|
| **n8n** | Workflow automation engine |
| **Airtable** | User records database |
| **imgBB** | Profile picture hosting |
| **Gmail (OAuth2)** | Email notifications |
| **Telegram Bot** | Instant owner alerts |
| **Luxon** | Date calculations in n8n expressions |

---

## 📦 Prerequisites

Before setting up, ensure you have the following:

- [ ] **n8n** instance (cloud or self-hosted)
- [ ] **Airtable** account with a base created
- [ ] **imgBB** account with an API key
- [ ] **Gmail** account with OAuth2 credentials set up in n8n
- [ ] **Telegram Bot** created via @BotFather
- [ ] Telegram **Chat ID** for the owner account

---

## 🔧 Setup Guide

### Step 1 — Clone the Repository

```bash
git clone https://github.com/vincegwu/berryexcel-gym-automation.git
cd berryexcel-gym-automation
```

### Step 2 — Set Up Airtable

Create a base with the following table structure (see [Airtable Schema](#-airtable-schema) below).

### Step 3 — Configure imgBB

1. Sign up at [https://imgbb.com](https://imgbb.com)
2. Go to **Settings → API**
3. Generate an API key
4. Add it as a query parameter `key` in the imgBB HTTP Request node

### Step 4 — Configure Gmail OAuth2

1. In n8n, go to **Credentials → New**
2. Search for **Gmail OAuth2**
3. Follow the OAuth2 setup flow
4. Connect your Gmail account

### Step 5 — Configure Telegram Bot

1. Open Telegram and message **@BotFather**
2. Send `/newbot` and follow prompts
3. Copy the **Bot Token**
4. Get your **Chat ID** from **@userinfobot**
5. Add credentials in n8n under **Telegram API**

### Step 6 — Import Workflows

1. Open your n8n instance
2. Go to **Workflows → Import**
3. Import each workflow JSON file from the `/workflows` folder

### Step 7 — Link Error Handler

For both Workflow 1 and Workflow 2:
1. Open workflow **Settings** (gear icon)
2. Set **Error Workflow** to `Workflow 3 — Error Handler`
3. Save

### Step 8 — Activate All Workflows

Toggle the **Activate** switch to ON for all three workflows in this order:
1. Workflow 3 (Error Handler) — must be activated first
2. Workflow 1 (berrygym-onboarding)
3. Workflow 2 (gym-subscription-renewal)

---

## 🔑 Environment Variables

If self-hosting n8n, configure these in your `.env` file:

```bash
# n8n Configuration
N8N_HOST=your-domain.com
N8N_PORT=5678
N8N_PROTOCOL=https
WEBHOOK_URL=https://your-domain.com

# Execution Settings
EXECUTIONS_TIMEOUT=3600
EXECUTIONS_TIMEOUT_MAX=7200

# Database (if using external DB)
DB_TYPE=postgresdb
DB_POSTGRESDB_HOST=localhost
DB_POSTGRESDB_PORT=5432
DB_POSTGRESDB_DATABASE=n8n
DB_POSTGRESDB_USER=n8n
DB_POSTGRESDB_PASSWORD=your_password
```

---

## 🗄 Airtable Schema

### Table: Members

| Field Name | Field Type | Notes |
|---|---|---|
| `Full Name` | Single Line Text | User's full name |
| `Email` | Email | Used as unique identifier |
| `Phone Number` | Single Line Text | Accepts any format |
| `Subscription` | Single Line Text | e.g. `1 Month`, `2 Months` |
| `Start Date` | Single Line Text | Format: `dd MMM yyyy` |
| `End Date` | Single Line Text | Format: `dd MMM yyyy` |
| `Status` | Single Select | Options: `Active`, `Expired`, `Pending` |
| `Profile-Picture` | Attachment | Stores imgBB image URL |

### Status Field Options

| Option | Colour | Trigger |
|---|---|---|
| `Active` | 🟢 Green | On onboarding or renewal |
| `Expired` | 🔴 Red | When End Date passes |
| `Pending` | 🟡 Yellow | Optional — manual use |

---

## 💡 Key Learnings

Building this project involved solving several real-world automation challenges:

**Binary Data Handling**
- Binary property names from Form Trigger must be explicitly preserved through Code nodes using `binary: item.binary`
- imgBB requires binary data under the `image` parameter via `n8n Binary Data` type

**Date Calculations**
- n8n uses **Luxon** for date handling — always use plural keys (`months` not `month`) in `.plus()` and `.minus()`
- Airtable List Records returns dates as strings — use `DateTime.fromFormat()` for reliable comparison

**Airtable Data Structure**
- Fields from **Create/Update** operations: `$json['fieldName']`
- Fields from **List Records** operations: `$json.fields['fieldName']`

**Expression References**
- Always reference upstream nodes explicitly using `$('Node Name').item.json` when `$json` points to the wrong node output

**Error Handling**
- Error Workflow must be **activated** before it can be selected in workflow settings
- Error Trigger must be the **first node** in the error workflow

---

## 📁 Folder Structure

```
berryexcel-gym-automation/
│
├── workflows/
│   ├── berrygym-onboarding.json
│   ├── gym-subscription-renewal.json
│   └── error-handler.json
│
├── email-templates/
│   ├── welcome-email.html
│   ├── expiry-alert-email.html
│   ├── expired-notice-email.html
│   ├── renewal-confirmation-email.html
│   └── error-report-email.html
│
├── docs/
│   ├── airtable-schema.md
│   ├── setup-guide.md
│   └── troubleshooting.md
│
└── README.md
```

---

## 🐛 Troubleshooting

| Error | Cause | Fix |
|---|---|---|
| `Binary field 'data' not found` | Binary not preserved in Code node | Add `binary: item.binary` to return statement |
| `Invalid input for Attachment field` | URL not wrapped in array | Use `[{ url: ... }]` format |
| `Execution Stopped` | IF node filtering all records | Check date format matches Airtable values |
| `Chat not found` | Telegram bot not started | Send `/start` to bot first |
| `Error Workflow greyed out` | Error workflow not activated | Activate Workflow 3 first |
| `Status returns 200` | Code node placed after HTTP node | Move Code node before imgBB upload |
| `Unexpected string` | Unicode character in code comment | Remove special characters from comments |

---

## 🤝 Contributing

Contributions, issues and feature requests are welcome!

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m 'Add your feature'`
4. Push to the branch: `git push origin feature/your-feature`
5. Open a Pull Request

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

## 👤 Author

**Built with n8n by Egwu Vincent Oko**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0077B5?style=for-the-badge&logo=linkedin)](https://linkedin.com/in/egwu-oko)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=for-the-badge&logo=github)](https://github.com/vincegwu)

---

> ⭐ If you found this project useful, please consider giving it a star on GitHub!
