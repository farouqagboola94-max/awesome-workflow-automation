# 📒 Automation Registry

Single source of truth for every automation built in the `playbooks/` set. **53 workflows** across 6 playbooks, plus one report-back loop.

- **Platform:** authored as importable **n8n** JSON (`*.n8n.json`). Each maps 1:1 to **Zapier** and **Make** — same triggers, same branches.
- **State:** all ship **inactive** with placeholder base IDs, credential names, and `$env` secrets. Nothing runs or sends until configured.
- **Last updated:** 2026-07-12

## Provisioning status

| Platform | Status | Note |
|----------|--------|------|
| n8n (definitions) | ✅ Saved | All 53 committed to `playbooks/`, JSON-validated |
| Make | 🟡 Live (capped) | Account authenticated (Farouq Agboola). **Free plan caps at 2 scenarios / 1000 ops-mo / 1 data store / no app connections.** 1 live scenario provisioned as proof (below). |
| Zapier | 🟢 Quota reset (2026-08-24) | **8 apps auto-provisioned with auth bound**: Airtable, Gmail, Resend, HubSpot, GitHub, Notion, Instagram, YouTube. As of the 2026-08-24 weekly check, execute calls no longer return `402 insufficient tasks` — the monthly task quota has reset, so the 7 pending fields + `appShared` tables + the †-skills are now ready to finish. |

### What's actually live

**Make (us2.make.com):**
- **Webhook** — `https://hook.us2.make.com/8uqh16nstid9y4ubw52w5guxmk02de2q` (hook id `2566464`)
- **Scenario** — "Catalyst — Feedback Intake (live proof)" (id `5641203`), **active**. Native modules only (webhook → JSON response), runs with zero app connections on the Free plan.

**Zapier — 8 connected apps (auth bound):** Airtable, Gmail, Resend, HubSpot, GitHub (`farouqagboola94-max`), Notion, Instagram, YouTube.

**Airtable backing tables created with FULL field schemas (real, in your existing bases):**
| Base | Table | ID | Fields | Powers |
|------|-------|----|--------|--------|
| CATALYST Events (`app6ru0ATU07D5eSG`) | Waitlist | `tblOUbsG9GEHpg0lY` | Email, Name, Referral Code, Referred By, Referral Count, Status, Source, Signed Up At | Catalyst 01, 07 |
| CATALYST Events (`app6ru0ATU07D5eSG`) | Feedback Backlog | `tblgXeLLfE67mRua5` | Summary, Category, Sentiment, Severity, Tags, Raw Feedback, Reporter, Vote Count, Status, Discord Thread | Catalyst 05, 07, 09 |
| SNEAKFEST 2026 (`appCmR6YaOmAD5N98`) | Attendees | `tblaj0m3AqDDBrk93` | Email, Name, Ticket Tier, Order ID, Quantity, QR Token, Checked In, Checked In At, Gate, Purchased At | Sneaker Fest 01, 05, 06 |
| CATALYST Events (`app6ru0ATU07D5eSG`) | Key Pool | `tblketM7S2I06aFWu` | Key, Status, Assigned To, Creator Tier, Assigned At ✅ | Catalyst 06 |
| CATALYST Events (`app6ru0ATU07D5eSG`) | Playtest Slots | `tbleF18grtq3TgxAv` | Session, Starts At, Seats Left, Status, Session URL ✅ | Catalyst 08 |
| CATALYST Events (`app6ru0ATU07D5eSG`) | Players | `tbl5ay1d9aaa6IaO6` | Email, Name, Status, Last Seen, Last Milestone, Winback Sent, Winback Sent At ✅ | Catalyst 10 |
| SNEAKFEST 2026 (`appCmR6YaOmAD5N98`) | Waitlist Notify | `tblu2h5dHGw2UsS4a` | Email, Product Handle ✅ — **add manually:** Size, Notified, Notified At | Sneaker Fest 03, 08 |
| SNEAKFEST 2026 (`appCmR6YaOmAD5N98`) | Help Log | `tblmRWs4fBnRPc2pT` | Summary ✅ — **add manually:** Type, Location, Urgent, Status | Sneaker Fest 09 |

> **Zapier task quota reached.** Field creation runs as Zapier "tasks"; the free plan's monthly allotment was exhausted mid-batch, so 7 fields (Waitlist Notify: Size, Notified, Notified At; Help Log: Type, Location, Urgent, Status) still need to be added — 30 seconds each in the Airtable UI, or re-run next month / after a Zapier upgrade.

> The SNEAKFEST base already has Tickets, Exhibitors, Vendors & Logistics, Marketing, Media & Press, Team, Speakers, Budget/P&L, Master Timeline; the CATALYST base already has Events, Vendors, Attendees, Resources.

**37 Zapier skills saved (run any by name):**

| Skill | Automation | Apps |
|-------|-----------|------|
| `catalyst waitlist signup` | Catalyst 01 | Airtable + Resend |
| `catalyst feedback intake` | Catalyst 05 | Airtable |
| `catalyst creator keys` | Catalyst 06 | Airtable + Resend |
| `catalyst kpi digest` | Catalyst 07 | Airtable + Resend |
| `catalyst playtest scheduler` | Catalyst 08 | Airtable + Resend |
| `catalyst churn winback` | Catalyst 10 | Airtable + Resend |
| `sneakerfest ticket onboarding` | Sneaker Fest 01 | Airtable + Resend |
| `sneakerfest post-event followup` | Sneaker Fest 05 | Airtable + Resend |
| `sneakerfest door checkin` | Sneaker Fest 06 | Airtable |
| `sneakerfest restock matcher` | Sneaker Fest 03/08 | Airtable + Resend |
| `sneakerfest event support` | Sneaker Fest 09 | Airtable + Resend |
| `github issue triage` | Dev-Ops 01 | GitHub |
| `github ci failure alert` | Dev-Ops 02 | GitHub |
| `github release notes` | Dev-Ops 03 | GitHub |
| `github stale sweeper` | Dev-Ops 04 | GitHub |
| `github link check report` | Dev-Ops 05 | GitHub |
| `github contribution lint` | Dev-Ops 06 | GitHub |
| `github contributor welcome` | Dev-Ops 07 | GitHub |
| `council convention check` | Dev-Ops 08 | GitHub |
| `sponsor lead intake` | Shared 03 | HubSpot |
| `email triage autodraft` | Growth 06 | Gmail |
| `instagram scheduler` | Growth 03 | Instagram + Airtable |
| `youtube cross promote` | Growth 04 | YouTube + Instagram + Gmail |
| `lead scoring routing` | Ops 08 | HubSpot |
| `testimonial to instagram` | Growth 10 | Instagram |
| `cross project digest` | Ops 10 | Airtable + GitHub + Resend — **runnable now** |
| `contract renewal reminder` | Ops 05 | Airtable + Resend — **runnable now** |
| `post purchase review request` | Growth 05 | Resend — **runnable now** |
| `revenue reconciliation` | Ops 02 | Airtable + Resend † |
| `weekly pnl snapshot` | Ops 04 | Airtable + Resend † |
| `expense receipt capture` | Ops 03 | Airtable † |
| `devlog seo social` | Growth 01 | Airtable † |
| `content calendar generator` | Growth 02 | Airtable † |
| `meeting notes to tasks` | Growth 07 | Airtable † |
| `faq autoresponder` | Growth 08 | Airtable † |
| `newsletter compile` | Growth 09 | Airtable + Resend † |
| `duplicate contact cleanup` | Ops 09 | Airtable † |

**Not skill-ified** (need an app that isn't connected in Zapier): Ops 01 invoicing (QuickBooks), Ops 06 Sheets backup (Google Sheets), Ops 07 KPI alert (Slack).

† Depends on an `appShared` base/table (`Orders`, `Expenses`, `Contacts`, `Tasks`, `FAQ`, `Content Calendar`) that still needs creating — blocked by the exhausted Zapier task quota. The skill is saved and ready; it executes once those tables exist and the quota resets.

> `sneakerfest restock matcher` and `sneakerfest event support` are fully wired but their tables (Waitlist Notify, Help Log) are missing the 7 fields above — add those to make them execute cleanly.
>
> The 5 skills above map the new Ops/Growth automations onto already-connected apps (Gmail, Instagram, YouTube, HubSpot). Gmail/YouTube skills create **drafts** (never auto-send). Execution consumes Zapier tasks, so they run once the quota resets or is upgraded.

### Remaining automations — what each still needs

37 of the 53 automations are skill-ified. The rest are blocked by an **unconnected app** or the **Zapier task quota** (which gates creating the last `appShared` tables). Current status:

| Automation(s) | Blocker |
|---------------|---------|
| Catalyst 02 community moderation, 09 bug→Discord, 03 devlog publishing | **Discord / X** not connected |
| Catalyst 04 incident monitoring | **Sentry / Jira / PagerDuty / Slack** not connected |
| Sneaker Fest 02 vendor intake | **e-sign + QuickBooks** not connected (Airtable/email parts runnable) |
| Sneaker Fest 03 drop alerts, 04 UGC | **X / social listening** not connected |
| Sneaker Fest 07 deposit reminder | uses the existing `Vendors` table — skill can be added on request |
| Shared 01 cart recovery, 02 support router, 04 outreach, 05 scheduler, 06 bounce; Ops/Growth items marked † | need `appShared` tables — **blocked by Zapier task quota** |

**Note:** Catalyst 06/08/10 and Sneaker Fest 08/09 are now skill-ified (their tables were created this session). Sneaker Fest 08 & 09 need the 7 remaining fields (see above) to run cleanly.

### Connecting more apps (to unlock the last automations)

Connect any of these in **Zapier** (Zapier → My Apps → Add connection), then ask me to skill-ify — no code needed on your side:

| App | Unlocks |
|-----|---------|
| **Slack** | Ops 07 KPI alert; all Slack alert steps (incident, sales pings, ops reminders) |
| **QuickBooks** | Ops 01 auto-invoicing; Sneaker Fest 02 vendor deposit invoicing |
| **Google Sheets** | Ops 06 Airtable→Sheets backup |
| **Discord** | Catalyst 02 moderation, 09 bug threads; Sneaker Fest drop announcements |
| **X / Twitter** | Catalyst 03 devlog publishing; Sneaker Fest 03/04 drop + UGC |

For **Make** (to run standing scenarios instead of on-demand skills): upgrade past the Free 2-scenario cap, then add the same app connections in Make → I port the n8n definitions to live scenarios.

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

## Ops — finance, reporting & admin (10)

| # | Workflow | Trigger | File |
|---|----------|---------|------|
| 1 | Auto-Invoice on Sale | webhook | `ops/01-invoice-on-sale.n8n.json` |
| 2 | Daily Revenue Reconciliation | schedule | `ops/02-daily-revenue-reconciliation.n8n.json` |
| 3 | Expense Receipt Capture | webhook | `ops/03-expense-receipt-capture.n8n.json` |
| 4 | Weekly P&L Snapshot | schedule | `ops/04-weekly-pnl-snapshot.n8n.json` |
| 5 | Contract & Renewal Reminder | schedule | `ops/05-contract-expiry-reminder.n8n.json` |
| 6 | Airtable → Google Sheets Backup | schedule | `ops/06-airtable-sheets-backup.n8n.json` |
| 7 | KPI Anomaly Alert | schedule | `ops/07-kpi-anomaly-alert.n8n.json` |
| 8 | Lead Scoring & Routing | webhook | `ops/08-lead-scoring-routing.n8n.json` |
| 9 | Duplicate Contact Cleanup | schedule | `ops/09-duplicate-contact-cleanup.n8n.json` |
| 10 | Cross-Project Weekly Digest | schedule | `ops/10-cross-project-weekly-digest.n8n.json` |

## Growth — content, marketing & comms (10)

| # | Workflow | Trigger | File |
|---|----------|---------|------|
| 1 | Devlog → SEO Meta + Social Snippets | webhook | `growth/01-devlog-seo-social.n8n.json` |
| 2 | Weekly Content Calendar Generator | schedule | `growth/02-content-calendar-generator.n8n.json` |
| 3 | Instagram Post Scheduler | schedule | `growth/03-instagram-scheduler.n8n.json` |
| 4 | YouTube Upload Cross-Promote | webhook | `growth/04-youtube-cross-promote.n8n.json` |
| 5 | Post-Purchase Review Request | webhook | `growth/05-review-request.n8n.json` |
| 6 | Email Triage & Auto-Draft | Gmail trigger | `growth/06-email-triage-autodraft.n8n.json` |
| 7 | Meeting Notes → Action Items | webhook | `growth/07-meeting-notes-to-tasks.n8n.json` |
| 8 | FAQ Auto-Responder | webhook | `growth/08-faq-autoresponder.n8n.json` |
| 9 | Weekly Newsletter Compile & Send | schedule | `growth/09-newsletter-compile.n8n.json` |
| 10 | 5-Star Review → Social Post | webhook | `growth/10-testimonial-to-social.n8n.json` |

> These 20 are saved as importable definitions. Several map directly onto your connected Zapier apps (Gmail, Airtable, HubSpot, Instagram, YouTube, Resend) — I can turn them into Zapier skills once the Zapier task quota resets or is upgraded.

---

## Credentials index

Across all 33 workflows you'll wire some subset of: **Airtable**, **SMTP/Email or Resend**, **Slack**, **Discord** (bot token + webhooks), **OpenAI**, **Notion**, **X/Twitter**, **Jira**, **PagerDuty**, **Statuspage**, **Sentry**, **GitHub** (PAT), **Shopify**, **MailerLite/ESP**, **HubSpot**, **QuickBooks**, **Bitly**, **DocuSeal** (or other e-sign). Secrets are always `$env.*` — never hard-coded.
