# n8n Lead Intake & Notification Automation

A tested n8n workflow that catches a new lead from any webhook-capable form, logs it to Google Sheets, posts an alert to Slack, and sends an automatic follow-up email — all in one run.

**Status: tested and verified.** Every branch (Sheets, Slack, email) has been run end-to-end with real test data and confirmed working as of August 2026.

## What it does

```
Webhook receives lead → Parses & validates the data → in parallel:
  → Appends a row to a Google Sheet
  → Posts a formatted alert to a Slack channel
  → Sends a personalized follow-up email to the lead
```

The parser (a Code node) accepts either structured JSON fields (`name`, `email`, `company`, `message`) or a single raw text blob, and will extract fields from the raw text if structured fields aren't present. It requires a valid email address to proceed — if none is found, it throws an error rather than silently logging incomplete leads.

## Prerequisites

- An n8n instance — self-hosted (free, e.g. `npx n8n` or Docker) or [n8n Cloud](https://n8n.io) (paid, free trial available)
- A Google account, for Sheets
- A Slack workspace where you can create/install an app with a bot token
- An SMTP-capable email account or service (Gmail with an App Password, or a transactional email provider)

## Setup

1. **Import** `workflow.json` into your n8n instance (Workflows → Import from File).
2. **Google Sheets node:**
   - Connect your Google account under Credentials.
   - Create a sheet with a header row of exactly: `Name | Email | Company | Message | Received At`
   - Name the sheet tab `Leads` (or update the node's "Sheet Name" field to match whatever you name it).
   - Replace `[INSERT_GOOGLE_SHEET_ID]` in the node with your sheet's ID (the long string in your sheet's URL between `/d/` and `/edit`).
   - If you see a `columns.schema is required` error, toggle the node's Mapping Column Mode from "Map Each Column Manually" to "Map Automatically" and back — this forces it to re-read your headers.
3. **Slack node:**
   - Create a Slack app with a bot token (`chat:write` scope) and connect it under Credentials.
   - Invite the bot to your target channel (`/invite @YourAppName` in that channel).
   - Replace `[INSERT_SLACK_CHANNEL_ID]` with your channel's ID (found in the channel URL, starts with `C`).
4. **Send Follow-up Email node:**
   - Connect an SMTP credential. For Gmail: host `smtp.gmail.com`, port `465`, SSL on, and a Google **App Password** (requires 2-Step Verification enabled) — not your normal password.
   - Replace `[INSERT_SENDER_EMAIL]` with your sending address.
5. **Test it:** click Execute Workflow, then send a POST request to the webhook's test URL with sample data:
   ```json
   {"name":"Jane Doe","email":"jane@example.com","company":"Acme Inc","message":"Interested in your service"}
   ```
   Confirm a row appears in your Sheet, a message posts in Slack, and the email actually sends — not just that the node shows no error.

## Notes & limitations

- The webhook has **no authentication** by default — anyone with the URL can trigger it. If you're putting this behind a public-facing form, add header-based auth in the Webhook node's options.
- Credentials are intentionally **not** included in the exported JSON (n8n strips them for security) — every user needs to connect their own.
- Free tiers this was tested against: n8n Cloud's free trial, Gmail's free SMTP limits, and Mailtrap's free sandbox (for safe testing without sending real email).

## License

Free to copy, modify, and reuse.
