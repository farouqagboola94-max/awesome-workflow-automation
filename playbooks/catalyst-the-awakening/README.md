# ⚡ Catalyst: The Awakening — Automation Playbook

Five production-shaped **n8n** workflows covering the full lifecycle of a game / immersive-experience launch: build the audience, run the community, ship content, survive launch day, and close the feedback loop.

> All workflows are importable n8n JSON (`*.n8n.json`). They ship **inactive** with placeholder credentials, base IDs, and `$env` variables — wire up your own before enabling. See [Setup](#setup) below.

| # | Workflow | Trigger | Core stack | File |
|---|----------|---------|-----------|------|
| 1 | Waitlist & Early-Access Funnel | Signup webhook | Airtable + Email | [`01-waitlist-early-access-funnel.n8n.json`](./01-waitlist-early-access-funnel.n8n.json) |
| 2 | Community Onboarding & AI Moderation | Discord gateway event | Discord API + OpenAI | [`02-community-onboarding-moderation.n8n.json`](./02-community-onboarding-moderation.n8n.json) |
| 3 | Devlog & Content Publishing | Notion status change | Notion + X + Discord + Email | [`03-content-devlog-publishing.n8n.json`](./03-content-devlog-publishing.n8n.json) |
| 4 | Launch-Day Incident Monitoring | Sentry webhook | Sentry + Slack + Jira + PagerDuty | [`04-launch-incident-monitoring.n8n.json`](./04-launch-incident-monitoring.n8n.json) |
| 5 | Feedback → Roadmap Loop | Feedback webhook | OpenAI + Airtable + Slack | [`05-feedback-roadmap-loop.n8n.json`](./05-feedback-roadmap-loop.n8n.json) |

---

## 1. Waitlist & Early-Access Funnel

**Goal:** turn signups into a viral referral queue that unlocks access automatically.

**Flow:** `Signup webhook → validate email + mint referral code → dedupe against Airtable → create waitlist record → send welcome + referral link → if referred, credit the referrer → auto-unlock early access at 3 referrals → send "access unlocked" email`.

- Referral code is deterministic (base64 of email) so links are stable and idempotent.
- Dedupe branch short-circuits repeat signups with an `already_registered` response.
- Unlock threshold lives in the *Credit Referrer* node — change `>= 3` to tune virality.

**Airtable — `Waitlist` table:** `Email`, `Name`, `Referral Code`, `Referred By`, `Referral Count`, `Status` (Waiting / Early Access Unlocked), `Source`, `Signed Up At`.

## 2. Community Onboarding & AI Moderation

**Goal:** greet every new member and keep the server clean without a 24/7 human mod team.

**Flow:** `Discord event → route by type`:
- **`GUILD_MEMBER_ADD`** → assign *Unverified* role → open DM → send welcome + role-picker instructions.
- **`MESSAGE_CREATE`** → OpenAI classifies (`SPAM / SCAM / TOXIC / QUESTION / CLEAN`) → violations get deleted + logged to the mod channel; genuine questions get a nudge to `#faq` and ping mods; clean messages pass through.

- Classifier runs at `temperature: 0` and returns strict JSON for reliable routing.
- SCAM label is tuned for the free-nitro / crypto-airdrop phishing that hits game Discords at launch.
- Requires a Discord bot token with `Manage Roles`, `Manage Messages`, and DM scopes, plus an event forwarder (gateway → this webhook).

## 3. Devlog & Content Publishing Pipeline

**Goal:** write once in Notion, publish everywhere at once.

**Flow:** `Notion page flips to "Ready to Publish" → pull body blocks → build per-channel variants → fan out to Blog/CMS + X + Discord + Newsletter in parallel → mark the Notion page "Published"`.

- The *Build Channel Variants* Code node flattens Notion blocks to Markdown and produces a 200-char tweet and a 500-char Discord post from the same source.
- Fan-out is genuinely parallel (four connections off one node); the Notion status update converges the loop so nothing double-publishes.
- Swap the Blog/CMS HTTP node for your own headless CMS endpoint.

## 4. Launch-Day Incident Monitoring

**Goal:** on launch day, route real incidents to humans in seconds and ignore the noise.

**Flow:** `Sentry webhook → compute severity from event volume → if High/Critical → alert #war-room + open Jira ticket + post a status-page incident → if Critical → page on-call via PagerDuty`.

- Severity is derived from event count (`>=500` or `fatal` = critical, `>=100` = high, `>=25` = medium) so a single flapping error doesn't wake anyone.
- Low/medium severities are recorded but suppressed from paging — tune thresholds in *Compute Severity*.
- Everything below "high" simply drops, keeping the war-room channel signal-rich.

## 5. Feedback → Roadmap Loop

**Goal:** every piece of player feedback becomes a triaged, de-duplicated, voted backlog item.

**Flow:** `Feedback webhook → OpenAI triage (category / sentiment / severity / tags) → search Airtable for a duplicate → if dup, increment vote count; else create backlog item → if high-severity bug, ping the product owner in Slack → thank the reporter`.

- De-dup uses a fuzzy `FIND` on the AI summary so "can't log in" and "login broken" collapse into one voted item.
- Vote count doubles as a demand signal for roadmap prioritisation.
- High-severity bugs jump the queue with a direct Slack ping; everything else waits for triage review.

**Airtable — `Backlog` table:** `Summary`, `Category`, `Sentiment`, `Severity`, `Tags`, `Raw Feedback`, `Reporter`, `Vote Count`, `Status`.

---

## Setup

1. **Import** — in n8n: *Workflows → Import from File* → pick a `*.n8n.json`.
2. **Credentials** — create and attach: Airtable PAT, SMTP/email (or swap the Email nodes for the [Resend](https://resend.com) node), OpenAI, Discord bot (HTTP Header Auth with `Authorization: Bot <token>`), Notion, X/Twitter OAuth2, Slack OAuth2, Jira, and generic HTTP Header Auth for Sentry/PagerDuty/Statuspage.
3. **Environment variables** — set the `$env` references used across workflows:
   - `CATALYST_ROLE_UNVERIFIED`, `CATALYST_MODLOG_CHANNEL`, `CATALYST_DISCORD_WEBHOOK_URL`, `CATALYST_NEWSLETTER_LIST`
   - `STATUSPAGE_PAGE_ID`, `PAGERDUTY_ROUTING_KEY`
4. **Base/table IDs** — replace the placeholder Airtable base IDs (`appCatalyst…`) and Notion database IDs with your own.
5. **Test** — each webhook workflow has a copy-paste `curl` sample in the root [playbooks README](../README.md#testing-webhooks). Run in n8n *Test* mode first, then toggle **Active**.

> Prefer no-code? Every one of these maps 1:1 onto **Make** or **Zapier** — same triggers, same branches. n8n is used here because it's open-source, self-hostable, and has native AI (LLM) nodes, so the AI-moderation and AI-triage steps cost pennies at scale.
