# 📒 Automation Registry

Single source of truth for every automation built in the `playbooks/` set. **33 workflows** across 4 playbooks, plus one report-back loop.

- **Platform:** authored as importable **n8n** JSON (`*.n8n.json`). Each maps 1:1 to **Zapier** and **Make** — same triggers, same branches.
- **State:** all ship **inactive** with placeholder base IDs, credential names, and `$env` secrets. Nothing runs or sends until configured.
- **Last updated:** 2026-07-12

## Provisioning status

| Platform | Status | Note |
|----------|--------|------|
| n8n (definitions) | ✅ Saved | All 33 committed to `playbooks/`, JSON-validated |
| Make | 🟡 Live (capped) | Account authenticated (Farouq Agboola). **Free plan caps at 2 scenarios / 1000 ops-mo / 1 data store / no app connections.** 1 live scenario provisioned as proof (below). |
| Zapier | 🟡 Live (on-demand) | **8 apps auto-provisioned with auth bound**: Airtable, Gmail, Resend, HubSpot, GitHub, Notion, Instagram, YouTube (219 actions). MCP executes actions **on demand**; it does not create standing event-triggered Zaps. |

### What's actually live

- **Make webhook** — `https://hook.us2.make.com/8uqh16nstid9y4ubw52w5guxmk02de2q` (hook id `2566464`)
- **Make scenario** — "Catalyst — Feedback Intake (live proof)" (id `5641203`), **active**. Native modules only (webhook → JSON response), so it runs with zero app connections on the Free plan.

### Hard limits discovered (why "all 33 auto-running" isn't possible yet)

1. **Make Free plan = 2 scenarios maximum.** 33 standing scenarios need a paid tier (Core/Pro) and Make-side app connections (Airtable, Slack, OpenAI, Discord, …), which are not yet added (`hasAddedApp: false`).
2. **Zapier MCP executes actions on demand** (create record, send email, etc.) rather than creating persistent trigger→action Zaps. Standing Zaps are built in Zapier's UI/API, not this interface.

### Path to lift the caps

- **Make:** upgrade the org to a paid tier, then add app connections in Make → I can port the remaining n8n definitions to scenarios.
- **Zapier:** apps are already connected — I can execute real actions now (e.g., create the Airtable backing tables, send emails, create HubSpot deals) and save reusable **Zapier skills** per automation.
- The weekly report-back loop will re-check both and flag what's newly deployable.

## Report-back loop

A weekly **Routine** fires into the originating session every **Monday 08:00 UTC** to report PR/CI status, check whether Zapier/Make became available, and summarize this registry. Managed via the claude-code-remote Routines API (`trigger_id` recorded in session). Ask to change cadence or stop it anytime.

---

## Catalyst: The Awakening (10)

| # | Workflow | Trigger | File |
|---|----------|---------|------|
| 1 | Waitlist & Early-Access Funnel | webhook | `catalyst-the-awakening/01-waitlist-early-access-funnel.n8n.json` |
| 2 | Community Onboarding & AI Moderation | Discord event | `catalyst-the-awakening/02-community-onboarding-moderation.n8n.json` |
| 3 | Devlog & Content Publishing | Notion change | `catalyst-the-awakening/03-content-devlog-publishing.n8n.json` |
| 4 | Launch-Day Incident Monitoring | Sentry | `catalyst-the-awakening/04-launch-incident-monitoring.n8n.json` |
| 5 | Feedback → Roadmap Loop | webhook | `catalyst-the-awakening/05-feedback-roadmap-loop.n8n.json` |
| 6 | Creator / Streamer Key Distribution | webhook | `catalyst-the-awakening/06-creator-key-distribution.n8n.json` |
| 7 | Daily Community & KPI Digest | schedule | `catalyst-the-awakening/07-daily-kpi-digest.n8n.json` |
| 8 | Playtest Scheduler | webhook | `catalyst-the-awakening/08-playtest-scheduler.n8n.json` |
| 9 | Bug → Discord Tracking Thread | webhook | `catalyst-the-awakening/09-bug-to-discord-thread.n8n.json` |
| 10 | Early-Access Churn Win-Back | schedule | `catalyst-the-awakening/10-churn-winback.n8n.json` |

## Sneaker Fest (9)

| # | Workflow | Trigger | File |
|---|----------|---------|------|
| 1 | Ticket Purchase Onboarding | webhook | `sneaker-fest/01-ticket-purchase-onboarding.n8n.json` |
| 2 | Vendor / Booth Application Intake | webhook | `sneaker-fest/02-vendor-booth-intake.n8n.json` |
| 3 | Drop & Restock Alerts | webhook | `sneaker-fest/03-drop-restock-alerts.n8n.json` |
| 4 | UGC Aggregation & Hype Rewards | schedule | `sneaker-fest/04-ugc-social-aggregation.n8n.json` |
| 5 | Post-Event Nurture & Resale Loop | schedule | `sneaker-fest/05-post-event-nurture.n8n.json` |
| 6 | Door Check-In & Live Capacity | webhook | `sneaker-fest/06-door-checkin.n8n.json` |
| 7 | Vendor Deposit Reminder | schedule | `sneaker-fest/07-vendor-deposit-reminder.n8n.json` |
| 8 | Waitlist Restock Matcher (by size) | webhook | `sneaker-fest/08-waitlist-restock-matcher.n8n.json` |
| 9 | Event-Day Support & Lost-and-Found | webhook | `sneaker-fest/09-event-day-support-router.n8n.json` |

## Dev-Ops — repo automation (8)

| # | Workflow | Trigger | File |
|---|----------|---------|------|
| 1 | Issue AI Triage | webhook | `dev-ops/01-issue-ai-triage.n8n.json` |
| 2 | PR CI-Failure Notifier | webhook | `dev-ops/02-pr-ci-failure-notifier.n8n.json` |
| 3 | Release Notes On Tag | webhook | `dev-ops/03-release-notes-on-tag.n8n.json` |
| 4 | Stale Issue & PR Sweeper | schedule | `dev-ops/04-stale-issue-sweeper.n8n.json` |
| 5 | Awesome-List Dead-Link Checker | schedule | `dev-ops/05-awesome-link-checker.n8n.json` |
| 6 | Contribution Format Lint | webhook | `dev-ops/06-contribution-format-lint.n8n.json` |
| 7 | New Contributor Welcome | webhook | `dev-ops/07-new-contributor-welcome.n8n.json` |
| 8 | Council Agent Convention Validator | webhook | `dev-ops/08-council-convention-validator.n8n.json` |

## Shared — cross-event (6)

| # | Workflow | Trigger | File |
|---|----------|---------|------|
| 1 | Abandoned Checkout Recovery | webhook | `shared/01-abandoned-checkout-recovery.n8n.json` |
| 2 | AI Support Ticket Router | webhook | `shared/02-support-ticket-router.n8n.json` |
| 3 | Sponsor / Partner Lead Intake | webhook | `shared/03-sponsor-lead-intake.n8n.json` |
| 4 | Press / Influencer Outreach Tracker | schedule | `shared/04-press-outreach-tracker.n8n.json` |
| 5 | Social Content Scheduler | schedule | `shared/05-social-content-scheduler.n8n.json` |
| 6 | Email Bounce & Deliverability Handler | webhook | `shared/06-email-bounce-handler.n8n.json` |

---

## Credentials index

Across all 33 workflows you'll wire some subset of: **Airtable**, **SMTP/Email or Resend**, **Slack**, **Discord** (bot token + webhooks), **OpenAI**, **Notion**, **X/Twitter**, **Jira**, **PagerDuty**, **Statuspage**, **Sentry**, **GitHub** (PAT), **Shopify**, **MailerLite/ESP**, **HubSpot**, **QuickBooks**, **Bitly**, **DocuSeal** (or other e-sign). Secrets are always `$env.*` — never hard-coded.
