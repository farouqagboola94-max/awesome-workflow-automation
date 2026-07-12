# 👟 Sneaker Fest — Automation Playbook

Two revenue-driving **n8n** workflows for a sneaker convention / vendor marketplace: automate attendee onboarding at the door and vendor booth intake behind the scenes.

> Importable n8n JSON (`*.n8n.json`), shipped **inactive** with placeholder credentials and `$env` variables. Wire up your own before enabling. See [Setup](#setup).

| # | Workflow | Trigger | Core stack | File |
|---|----------|---------|-----------|------|
| 1 | Ticket Purchase Onboarding | Order webhook (Shopify / Eventbrite) | Airtable + Email + Slack | [`01-ticket-purchase-onboarding.n8n.json`](./01-ticket-purchase-onboarding.n8n.json) |
| 2 | Vendor / Booth Application Intake | Application webhook | OpenAI + Airtable + e-sign + QuickBooks | [`02-vendor-booth-intake.n8n.json`](./02-vendor-booth-intake.n8n.json) |

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

---

## Setup

1. **Import** — n8n *Workflows → Import from File* → pick a `*.n8n.json`.
2. **Credentials** — attach: Airtable PAT, SMTP/email (or the Resend node), Slack OAuth2, OpenAI, MailerLite (or your ESP), QuickBooks OAuth2, and generic HTTP Header Auth for the e-sign provider ([DocuSeal](https://www.docuseal.com/) used here; swap for DocuSign/PandaDoc).
3. **Environment variables:** `SNEAKERFEST_BOOTH_CONTRACT_TEMPLATE` (e-sign template ID).
4. **Base IDs** — replace the placeholder Airtable base (`appSneakerFest`) with your own; create the `Attendees` and `Vendors` tables above.
5. **Point your storefront/form at the webhooks:**
   - Shopify/Eventbrite → `.../webhook/sneakerfest-ticket-purchase` (order-created event).
   - Vendor application form (Typeform, Tally, Airtable form) → `.../webhook/sneakerfest-vendor-apply`.
6. **Test** in n8n *Test* mode with the sample payloads in the root [playbooks README](../README.md#testing-webhooks), then toggle **Active**.

> These map cleanly onto **Make** or **Zapier** too. n8n is used for the native AI node (vendor screening) and self-hosting.
