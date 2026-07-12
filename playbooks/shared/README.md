# 🔁 Shared — Cross-Event Automation Playbook

Six **n8n** workflows that apply to **both** Sneaker Fest and Catalyst (and most events/stores): commerce recovery, support routing, partnerships, PR, social scheduling, and email hygiene. Build once, point at either event's data.

> Importable n8n JSON, shipped **inactive** with `$env` placeholders. Map 1:1 to Zapier/Make.

| # | Workflow | Trigger | File |
|---|----------|---------|------|
| 1 | Abandoned Checkout Recovery | Checkout-abandoned webhook | [`01-abandoned-checkout-recovery.n8n.json`](./01-abandoned-checkout-recovery.n8n.json) |
| 2 | AI Support Ticket Router | Inbound support webhook | [`02-support-ticket-router.n8n.json`](./02-support-ticket-router.n8n.json) |
| 3 | Sponsor / Partner Lead Intake | Form webhook | [`03-sponsor-lead-intake.n8n.json`](./03-sponsor-lead-intake.n8n.json) |
| 4 | Press / Influencer Outreach Tracker | Schedule (6h) | [`04-press-outreach-tracker.n8n.json`](./04-press-outreach-tracker.n8n.json) |
| 5 | Social Content Scheduler | Schedule (hourly) | [`05-social-content-scheduler.n8n.json`](./05-social-content-scheduler.n8n.json) |
| 6 | Email Bounce & Deliverability Handler | ESP event webhook | [`06-email-bounce-handler.n8n.json`](./06-email-bounce-handler.n8n.json) |

## Highlights

- **#1 Abandoned Checkout Recovery** — waits 1h, re-checks whether the order completed, and only then emails a recovery nudge — so buyers who finished never get spammed.
- **#2 Support Ticket Router** — AI classifies inbound messages (billing/tech/vendor/general + urgency), logs them, routes to the right Slack channel, and returns an instant auto-reply.
- **#4 Outreach Tracker** — drips personalized, AI-drafted press/influencer emails from an Airtable queue and advances each contact's stage — a lightweight PR CRM.
- **#5 Social Scheduler** — publishes queued posts from an Airtable content calendar to X and Discord when their scheduled time arrives.
- **#6 Bounce Handler** — suppresses hard-bounces and spam complaints immediately (upsert to a suppression list) to protect sender reputation across both events' sends.

## Setup

1. **Import** each workflow.
2. **Credentials:** Airtable PAT, SMTP/email (or Resend), Slack OAuth2, OpenAI, X/Twitter OAuth2, HubSpot.
3. **Environment variables:** `SHARED_DISCORD_WEBHOOK`.
4. **Base:** create the placeholder `appShared` base with tables `Orders`, `Tickets`, `Outreach`, `Content Calendar`, `Contacts` (see field names inside each workflow).
5. **Webhooks:** point your store's checkout-abandoned, support inbox, sponsor form, and ESP (Resend/Mailgun/SendGrid) event webhooks at the matching n8n paths.
6. Test, then toggle **Active**.
