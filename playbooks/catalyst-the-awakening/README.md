# ⚡ Catalyst: The Awakening — Automation Playbook

Seven production-shaped **n8n** workflows covering the full lifecycle of a game / immersive-experience launch: build the audience, run the community, ship content, arm creators, survive launch day, close the feedback loop, and report on it daily.

> All workflows are importable n8n JSON (`*.n8n.json`). They ship **inactive** with placeholder credentials, base IDs, and `$env` variables — wire up your own before enabling. See [Setup](#setup) below.

| # | Workflow | Trigger | Core stack | File |
|---|----------|---------|-----------|------|
| 1 | Waitlist & Early-Access Funnel | Signup webhook | Airtable + Email | [`01-waitlist-early-access-funnel.n8n.json`](./01-waitlist-early-access-funnel.n8n.json) |
| 2 | Community Onboarding & AI Moderation | Discord gateway event | Discord API + OpenAI | [`02-community-onboarding-moderation.n8n.json`](./02-community-onboarding-moderation.n8n.json) |
| 3 | Devlog & Content Publishing | Notion status change | Notion + X + Discord + Email | [`03-content-devlog-publishing.n8n.json`](./03-content-devlog-publishing.n8n.json) |
| 4 | Launch-Day Incident Monitoring | Sentry webhook | Sentry + Slack + Jira + PagerDuty | [`04-launch-incident-monitoring.n8n.json`](./04-launch-incident-monitoring.n8n.json) |
| 5 | Feedback → Roadmap Loop | Feedback webhook | OpenAI + Airtable + Slack | [`05-feedback-roadmap-loop.n8n.json`](./05-feedback-roadmap-loop.n8n.json) |
| 6 | Creator / Streamer Key Distribution | Creator application webhook | OpenAI + Airtable + Email + Slack | [`06-creator-key-distribution.n8n.json`](./06-creator-key-distribution.n8n.json) |
| 7 | Daily Community & KPI Digest | Schedule (daily 08:00) | Airtable + Sentry + Slack + Email | [`07-daily-kpi-digest.n8n.json`](./07-daily-kpi-digest.n8n.json) |
| 8 | Playtest Scheduler | Playtest signup webhook | Airtable + Email | [`08-playtest-scheduler.n8n.json`](./08-playtest-scheduler.n8n.json) |
| 9 | Bug → Discord Tracking Thread | Confirmed-bug webhook | Discord + Airtable + Slack | [`09-bug-to-discord-thread.n8n.json`](./09-bug-to-discord-thread.n8n.json) |
| 10 | Early-Access Churn Win-Back | Schedule (weekly) | Airtable + OpenAI + Email | [`10-churn-winback.n8n.json`](./10-churn-winback.n8n.json) |

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

## 6. Creator / Streamer Key Distribution

**Goal:** vet creators, hand out early keys from a finite pool, and enforce embargo terms — without a human triaging every DM.

**Flow:** `Creator application webhook → normalize channel stats → AI vet (tier A/B/C/reject + risk flags) → if approved, reserve an available key from the Airtable pool → if a key is free, mark it assigned + email the key with embargo terms; if the pool is empty, alert #creators to top up → rejected applicants get a polite decline`.

- Keys are reserved atomically from a `Key Pool` table (oldest `Available` first), so two applicants can't be handed the same key.
- The AI vetter flags mismatched follower/view ratios — the classic bought-audience tell.
- Empty-pool detection turns "we ran out of keys" from a silent failure into a Slack ping.

**Airtable — `Key Pool` table:** `Key`, `Status` (Available / Assigned), `Assigned To`, `Creator Tier`, `Assigned At`, `Created`.

## 7. Daily Community & KPI Digest

**Goal:** every morning, one digest that tells the team how the launch is actually going — pulled from the other workflows' data.

**Flow:** `Daily 08:00 → in parallel pull new signups (24h), new feedback (24h), and Sentry error volume → aggregate into KPIs (signups, referral %, feedback count, bug count, sentiment split, error total) → format with a health traffic-light → post to #standup and email leadership`.

- It reads the same Airtable bases the waitlist (#1) and feedback (#5) workflows write to, so the digest is a free byproduct of the pipeline you already run.
- Error volume drives a 🟢/🟡/🔴 health signal (thresholds tunable in *Format Digest*).
- Referral % surfaces whether your viral loop (#1) is actually compounding day over day.

## 8. Playtest Scheduler

**Goal:** self-serve playtest booking against a finite pool of session slots, with automatic waitlisting when full.

**Flow:** `Signup webhook → find the earliest open slot with seats → if available, decrement seats + email the build/NDA/voice link + confirm; else return a waitlist response`. Seats decrement atomically so a slot never oversells.

## 9. Bug → Discord Tracking Thread

**Goal:** turn a confirmed high-severity bug into a dedicated Discord forum thread the community and team can follow.

**Flow:** `Confirmed-bug webhook → if high severity → create a Discord forum thread with repro details → link the thread back to the Airtable backlog item (status "Tracking") → ping #engineering`. Pairs with #5 (feedback loop), which is where confirmed bugs originate.

## 10. Early-Access Churn Win-Back

**Goal:** re-engage players who've gone quiet before they churn for good.

**Flow:** `Weekly → find Active players not seen in 14 days who haven't had a win-back yet → AI drafts a warm, personalized 60-word email teasing what's new → send → mark win-back sent` (so nobody gets it twice).

---

## Setup

1. **Import** — in n8n: *Workflows → Import from File* → pick a `*.n8n.json`.
2. **Credentials** — create and attach: Airtable PAT, SMTP/email (or swap the Email nodes for the [Resend](https://resend.com) node), OpenAI, Discord bot (HTTP Header Auth with `Authorization: Bot <token>`), Notion, X/Twitter OAuth2, Slack OAuth2, Jira, and generic HTTP Header Auth for Sentry/PagerDuty/Statuspage.
3. **Environment variables** — set the `$env` references used across workflows:
   - `CATALYST_ROLE_UNVERIFIED`, `CATALYST_MODLOG_CHANNEL`, `CATALYST_DISCORD_WEBHOOK_URL`, `CATALYST_NEWSLETTER_LIST`
   - `STATUSPAGE_PAGE_ID`, `PAGERDUTY_ROUTING_KEY`
   - `CATALYST_LEADERSHIP_EMAILS` (comma-separated recipients for the daily digest)
4. **Base/table IDs** — replace the placeholder Airtable base IDs (`appCatalyst…`) and Notion database IDs with your own.
5. **Test** — each webhook workflow has a copy-paste `curl` sample in the root [playbooks README](../README.md#testing-webhooks). Run in n8n *Test* mode first, then toggle **Active**.

> Prefer no-code? Every one of these maps 1:1 onto **Make** or **Zapier** — same triggers, same branches. n8n is used here because it's open-source, self-hostable, and has native AI (LLM) nodes, so the AI-moderation and AI-triage steps cost pennies at scale.
