# 👟 Sneaker Fest — Automation Playbook

Six **n8n** workflows for a sneaker convention / vendor marketplace — covering the full event lifecycle: sell tickets, screen vendors, drive hype, work the door, and run the post-event resale loop.

> Importable n8n JSON (`*.n8n.json`), shipped **inactive** with placeholder credentials and `$env` variables. Wire up your own before enabling. See [Setup](#setup).

| # | Workflow | Trigger | Core stack | File |
|---|----------|---------|-----------|------|
| 1 | Ticket Purchase Onboarding | Order webhook (Shopify / Eventbrite) | Airtable + Email + Slack | [`01-ticket-purchase-onboarding.n8n.json`](./01-ticket-purchase-onboarding.n8n.json) |
| 2 | Vendor / Booth Application Intake | Application webhook | OpenAI + Airtable + e-sign + QuickBooks | [`02-vendor-booth-intake.n8n.json`](./02-vendor-booth-intake.n8n.json) |
| 3 | Drop & Restock Alerts | Shopify inventory webhook | Bitly + X + Discord + Airtable + Email | [`03-drop-restock-alerts.n8n.json`](./03-drop-restock-alerts.n8n.json) |
| 4 | UGC Aggregation & Hype Rewards | Schedule (30 min) | OpenAI + Airtable + X + DM | [`04-ugc-social-aggregation.n8n.json`](./04-ugc-social-aggregation.n8n.json) |
| 5 | Post-Event Nurture & Resale Loop | Schedule (day after) | Airtable + Email + MailerLite + HubSpot | [`05-post-event-nurture.n8n.json`](./05-post-event-nurture.n8n.json) |
| 6 | Door Check-In & Live Capacity | QR scan webhook | Airtable | [`06-door-checkin.n8n.json`](./06-door-checkin.n8n.json) |
| 7 | Vendor Deposit Reminder | Schedule (daily) | Airtable + Email + Slack | [`07-vendor-deposit-reminder.n8n.json`](./07-vendor-deposit-reminder.n8n.json) |
| 8 | Waitlist Restock Matcher (by size) | Restock webhook | Airtable + Email | [`08-waitlist-restock-matcher.n8n.json`](./08-waitlist-restock-matcher.n8n.json) |
| 9 | Event-Day Support & Lost-and-Found | Help-request webhook | OpenAI + Airtable + Slack | [`09-event-day-support-router.n8n.json`](./09-event-day-support-router.n8n.json) |

---

## 1. Ticket Purchase Onboarding

**Goal:** every ticket sale instantly becomes a QR pass, a CRM record, and a segmented email contact — with premium buyers flagged to the sales team.

**Flow:** `Order webhook → parse order + classify tier (GA / VIP / Reseller / Early Bird) + mint QR token → add to Airtable attendee CRM → in parallel: add to email list + send pass with QR and venue map → if VIP/Reseller, notify the sales Slack channel`.

- Tier is inferred from the line-item title, so it works whether the order comes from Shopify, Eventbrite, or a custom checkout.
- The QR token is a base64url of `orderId:email`, encoded into a hosted QR image and pointed at your `/checkin/<token>` endpoint — scan it at the door against the same Airtable base.
- VIP emails automatically include early-entry instructions; the conditional text lives in the *Send Pass + Venue Map* node.

**Airtable — `Attendees` table:** `Name`, `Email`, `Ticket Tier`, `Order ID`, `Quantity`, `QR Token`, `Checked In` (bool), `Purchased At`.

**Pairs with:** a check-in workflow that flips `Checked In = true` when the QR is scanned (not included — it's the mirror of the token step here).

## 2. Vendor / Booth Application Intake

**Goal:** screen vendor applications with AI, then auto-run the whole approval pipeline — contract, deposit invoice, floor-plan placement — or decline politely.

**Flow:** `Application webhook → OpenAI scores fit 0–100 and recommends approve / review / reject → create vendor record in Airtable → route on recommendation`:
- **approve** → send booth contract for e-signature → invoice the booth deposit (QuickBooks) → move to floor-plan tracker.
- **reject** → send a polite decline email.
- **review** → flag to the vendors Slack channel with score, reason, and red flags for a human call.

- The scorer is explicitly instructed to cap score at 40 and raise a red flag on any hint of counterfeit / replica goods — the #1 risk for a sneaker event's reputation.
- Booth size (`S / M / L`) is suggested by the model and flows straight into the contract and floor-plan fields.
- The `review` path is the safety net: borderline or high-value applications always get human eyes rather than an auto-decision.

**Airtable — `Vendors` table:** `Brand`, `Contact Email`, `Social`, `Products`, `Fit Score`, `Recommendation`, `Booth Size`, `Red Flags`, `Status` (Applied → Contract Sent → …), `Floor Plan`.

## 3. Drop & Restock Alerts

**Goal:** the second a limited pair goes live or restocks, blast every channel and the product waitlist — with trackable links.

**Flow:** `Shopify inventory webhook → detect DROP (0→N) or RESTOCK (N→more) and whether it's limited → mint a Bitly UTM link → compose hype copy → fan out to X + Discord (@everyone) + email the per-product waitlist`.

- The *Detect* node returns nothing for irrelevant inventory changes (e.g. a sale decrementing stock), so the workflow only fires on genuine drops/restocks.
- "Limited/exclusive/deadstock/collab" tags escalate the copy (🔥🔥 + "Limited pairs") to drive urgency.
- Bitly links carry `utm_source=drop_alert` so you can attribute conversions per channel afterward.

**Airtable — `Waitlist Notify` table:** `Email`, `Product Handle` (customers opt into a specific pair's restock).

## 4. UGC Aggregation & Hype Rewards

**Goal:** continuously harvest `#SneakerFest` posts, auto-repost the best safe ones, and reward fans — no manual social scrolling.

**Flow:** `Every 30 min → fetch recent #SneakerFest mentions → split → skip already-captured → AI curate (score + safety) → store in Airtable → if score ≥ 75 and safe, repost + DM a discount code`.

- The AI curator gates on **safety** (no profanity/hate/competitor/scam) before anything is reposted to your brand account — critical for auto-reposting.
- Dedup by `Post ID` means the 30-min poll never double-processes or double-rewards.
- Swap the generic mention-provider HTTP nodes for your actual social-listening API (e.g. a provider from the [social / listening tools](../../readme.md) in the main list).

**Airtable — `UGC` table:** `Post ID`, `Author`, `Text`, `URL`, `Score`, `Safe`, `Captured At`.

## 5. Post-Event Nurture & Resale Loop

**Goal:** the day after the event, segment everyone by whether they actually showed, then run the right follow-up automatically.

**Flow:** `Scheduled (Mon 10:00 after event) → load attendees → segment on Checked In`:
- **Attended** → thank-you + survey → sync to loyalty segment → if VIP/Reseller, create a next-season lead in HubSpot.
- **No-show** → win-back email with a comeback discount.

- Uses the `Checked In` flag written by workflow #6, so attendance segmentation is real, not assumed.
- High-value buyers (VIP/Reseller) become tracked CRM leads for next season's pre-sale.
- Adjust the cron (`0 10 * * 1`) to the first business day after your event date.

## 6. Door Check-In & Live Capacity

**Goal:** scan a QR at the door, validate instantly, block duplicates, and keep a live headcount — the mirror of the token minted in workflow #1.

**Flow:** `QR scan webhook → decode token (base64url of orderId:email) → look up ticket in Airtable → switch`:
- **not found** → `404 invalid` (send to box office).
- **already checked in** → `409 duplicate` (possible screenshot/re-entry).
- **valid** → mark checked in + gate/time → `200` welcome with tier perks.

- Duplicate detection prevents one ticket walking in twice — a real problem when passes are screenshotted.
- The response payload drives the scanner UI (green welcome vs red reject) and surfaces VIP perks at the door.
- A sticky note shows how to wire a live capacity rollup for fire-safety headcount.

## 7. Vendor Deposit Reminder

**Goal:** chase unpaid booth deposits automatically and escalate the stragglers.

**Flow:** `Daily → find vendors with "Contract Sent" + unpaid deposit older than 3 days → if already reminded 3×, escalate to #vendors (consider releasing the booth); else send a payment reminder and increment the counter`. Turns deposit-chasing from a manual spreadsheet into a self-running dunning loop.

## 8. Waitlist Restock Matcher (by size)

**Goal:** the deeper cousin of #3 — notify a waitlisted buyer only when **their specific size** restocks.

**Flow:** `Restock webhook (with available sizes) → get un-notified waiters for that product → if the waiter's size is in the restock set → email them + mark notified`. No more "back in stock!" emails for a size the customer can't wear.

## 9. Event-Day Support & Lost-and-Found

**Goal:** a single on-site help channel that AI-triages and routes — medical/security escalate instantly.

**Flow:** `Help-request webhook → AI triage (type + urgency + reassuring reply) → if urgent, alarm #event-ops-urgent → always log to the Help Log → return the reply to the attendee`. Medical and security are always flagged urgent regardless of wording.

---

## Setup

1. **Import** — n8n *Workflows → Import from File* → pick a `*.n8n.json`.
2. **Credentials** — attach: Airtable PAT, SMTP/email (or the Resend node), Slack OAuth2, OpenAI, MailerLite (or your ESP), QuickBooks OAuth2, HubSpot, Bitly, X/Twitter OAuth2, and generic HTTP Header Auth for the e-sign provider ([DocuSeal](https://www.docuseal.com/) used here; swap for DocuSign/PandaDoc) and your social-listening API.
3. **Environment variables:**
   - `SNEAKERFEST_BOOTH_CONTRACT_TEMPLATE` (e-sign template ID)
   - `SNEAKERFEST_DISCORD_DROPS_WEBHOOK` (Discord webhook URL for #drops)
4. **Base IDs** — replace the placeholder Airtable base (`appSneakerFest`) with your own; create the `Attendees`, `Vendors`, `Waitlist Notify`, and `UGC` tables described above.
5. **Point your storefront/form/scanner at the webhooks:**
   - Shopify/Eventbrite → `.../webhook/sneakerfest-ticket-purchase` (order-created event).
   - Vendor application form (Typeform, Tally, Airtable form) → `.../webhook/sneakerfest-vendor-apply`.
   - Shopify inventory level update → `.../webhook/sneakerfest-inventory-event`.
   - Door scanner app → `.../webhook/sneakerfest-checkin` (POST `{ "token": "<qr>" }`).
6. **Test** in n8n *Test* mode with the sample payloads in the root [playbooks README](../README.md#testing-webhooks), then toggle **Active**.

> **Lifecycle order:** #1 mints QR passes → #6 scans them at the door and sets `Checked In` → #5 reads that flag the next day to segment follow-up. #3 and #4 run continuously through the hype cycle.

> These map cleanly onto **Make** or **Zapier** too. n8n is used for the native AI node (vendor screening) and self-hosting.
