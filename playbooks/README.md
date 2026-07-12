# 🎪 Event Automation Playbooks

Ready-to-import, end-to-end automation workflows for real events — built on **[n8n](https://n8n.io/)** (open-source, self-hostable, native AI nodes). Every workflow is a valid `*.n8n.json` you can import, wire to your own credentials, and switch on.

These are working blueprints, not just diagrams: webhook triggers, branching logic, AI classification/triage steps, dedupe, and multi-channel fan-out are all implemented.

## Playbooks

| Playbook | Workflows | What it covers |
|----------|-----------|----------------|
| [⚡ Catalyst: The Awakening](./catalyst-the-awakening/) | 7 | Waitlist funnel, community moderation, content publishing, incident response, feedback→roadmap loop, creator-key distribution, daily KPI digest |
| [👟 Sneaker Fest](./sneaker-fest/) | 6 | Ticket onboarding, AI vendor/booth intake, drop/restock alerts, UGC aggregation, post-event nurture, door check-in |

**13 workflows total** spanning webhooks, schedules, AI classification/triage/vetting, dedupe, multi-channel fan-out, and closed-loop reporting.

## How to use

1. **Pick a workflow** and open its `*.n8n.json`.
2. **Import** into n8n: *Workflows → Import from File*.
3. **Attach credentials** and set the environment variables listed in each playbook's README.
4. **Test** with the samples below, then toggle the workflow **Active**.

> Not on n8n? Every workflow here maps 1:1 onto **[Make](https://www.make.com/)** or **[Zapier](https://zapier.com/)** — identical triggers and branches. n8n is the reference because it's open-source and its LLM nodes make the AI steps (moderation, feedback triage, vendor screening) cost pennies at scale. See the main [readme](../readme.md) for the full tool landscape.

## Conventions

- Workflows ship **inactive** with **placeholder** base IDs, credential names, and `$env` variables — nothing runs or sends until you configure it.
- Secrets are always `$env.*` references, never hard-coded.
- AI nodes run at `temperature: 0` with strict-JSON output for deterministic routing.
- Each webhook path is namespaced by event (`catalyst-*`, `sneakerfest-*`).

## Testing webhooks

After importing, copy the **Test URL** from the trigger node (n8n shows both a test and a production URL). Sample payloads:

**Catalyst — waitlist signup**
```bash
curl -X POST "$N8N_TEST_URL/catalyst-waitlist-signup" \
  -H 'Content-Type: application/json' \
  -d '{"email":"player@example.com","name":"Ada","ref":"CAT-9XYZ12AB","source":"twitter"}'
```

**Catalyst — feedback intake**
```bash
curl -X POST "$N8N_TEST_URL/catalyst-feedback" \
  -H 'Content-Type: application/json' \
  -d '{"email":"player@example.com","message":"Game crashes when I open the map on level 3."}'
```

**Catalyst — Discord event (simulated MESSAGE_CREATE)**
```bash
curl -X POST "$N8N_TEST_URL/catalyst-discord-event" \
  -H 'Content-Type: application/json' \
  -d '{"type":"MESSAGE_CREATE","channel_id":"123","id":"456","author":{"id":"789"},"content":"Free nitro here → scam-link.example"}'
```

**Sneaker Fest — ticket purchase**
```bash
curl -X POST "$N8N_TEST_URL/sneakerfest-ticket-purchase" \
  -H 'Content-Type: application/json' \
  -d '{"order_id":"SF-1001","email":"buyer@example.com","name":"Sam Lee","line_items":[{"title":"VIP Weekend Pass","quantity":1}]}'
```

**Sneaker Fest — vendor application**
```bash
curl -X POST "$N8N_TEST_URL/sneakerfest-vendor-apply" \
  -H 'Content-Type: application/json' \
  -d '{"brand":"HeatCheck Kicks","email":"vendor@example.com","social":"@heatcheckkicks","products":"Deadstock Jordans, custom laces","followers":48000,"booth_size":"M","notes":"Been vending 3 years"}'
```

**Sneaker Fest — inventory drop**
```bash
curl -X POST "$N8N_TEST_URL/sneakerfest-inventory-event" \
  -H 'Content-Type: application/json' \
  -d '{"title":"Air Jordan 1 x Fest Collab","handle":"aj1-fest","tags":"limited,collab","old_available":0,"available":40,"price":"220.00"}'
```

**Sneaker Fest — door check-in** *(token is base64url of `orderId:email`)*
```bash
curl -X POST "$N8N_TEST_URL/sneakerfest-checkin" \
  -H 'Content-Type: application/json' \
  -d "{\"token\":\"$(printf 'SF-1001:buyer@example.com' | basenc --base64url | tr -d '=')\",\"gate\":\"main\"}"
```

**Catalyst — creator application**
```bash
curl -X POST "$N8N_TEST_URL/catalyst-creator-apply" \
  -H 'Content-Type: application/json' \
  -d '{"email":"streamer@example.com","handle":"@playscatalyst","platform":"twitch","followers":52000,"avg_views":1800,"channel_url":"https://twitch.tv/playscatalyst","region":"EU"}'
```

## Disclaimer

Workflow logic, brand names, endpoints, and payloads are illustrative. Review contracts, payment, and data-handling steps against your own legal/compliance requirements before going live.
