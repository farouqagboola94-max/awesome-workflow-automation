# 🎪 Event Automation Playbooks

Ready-to-import, end-to-end automation workflows for real events — built on **[n8n](https://n8n.io/)** (open-source, self-hostable, native AI nodes). Every workflow is a valid `*.n8n.json` you can import, wire to your own credentials, and switch on.

These are working blueprints, not just diagrams: webhook triggers, branching logic, AI classification/triage steps, dedupe, and multi-channel fan-out are all implemented.

## Playbooks

| Playbook | Workflows | What it covers |
|----------|-----------|----------------|
| [⚡ Catalyst: The Awakening](./catalyst-the-awakening/) | 5 | Waitlist funnel, community moderation, content publishing, launch-day incident response, feedback→roadmap loop |
| [👟 Sneaker Fest](./sneaker-fest/) | 2 | Ticket-purchase onboarding, AI vendor/booth intake |

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

## Disclaimer

Workflow logic, brand names, endpoints, and payloads are illustrative. Review contracts, payment, and data-handling steps against your own legal/compliance requirements before going live.
