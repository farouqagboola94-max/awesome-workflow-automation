# 📣 Growth — Content, Marketing & Comms Playbook

Ten **n8n** workflows that automate the content and communication grind: repurposing, scheduling, cross-promotion, review generation, inbox triage, meeting follow-ups, and newsletters.

> Importable n8n JSON, shipped **inactive** with `$env` placeholders. Map 1:1 to Zapier/Make.

| # | Workflow | Trigger | File |
|---|----------|---------|------|
| 1 | Devlog → SEO Meta + Social Snippets | New-post webhook | [`01-devlog-seo-social.n8n.json`](./01-devlog-seo-social.n8n.json) |
| 2 | Weekly Content Calendar Generator | Schedule | [`02-content-calendar-generator.n8n.json`](./02-content-calendar-generator.n8n.json) |
| 3 | Instagram Post Scheduler | Schedule | [`03-instagram-scheduler.n8n.json`](./03-instagram-scheduler.n8n.json) |
| 4 | YouTube Upload Cross-Promote | Upload webhook | [`04-youtube-cross-promote.n8n.json`](./04-youtube-cross-promote.n8n.json) |
| 5 | Post-Purchase Review Request | Purchase webhook | [`05-review-request.n8n.json`](./05-review-request.n8n.json) |
| 6 | Email Triage & Auto-Draft | Gmail trigger | [`06-email-triage-autodraft.n8n.json`](./06-email-triage-autodraft.n8n.json) |
| 7 | Meeting Notes → Action Items | Transcript webhook | [`07-meeting-notes-to-tasks.n8n.json`](./07-meeting-notes-to-tasks.n8n.json) |
| 8 | FAQ Auto-Responder | Question webhook | [`08-faq-autoresponder.n8n.json`](./08-faq-autoresponder.n8n.json) |
| 9 | Weekly Newsletter Compile & Send | Schedule | [`09-newsletter-compile.n8n.json`](./09-newsletter-compile.n8n.json) |
| 10 | 5-Star Review → Social Post | Review webhook | [`10-testimonial-to-social.n8n.json`](./10-testimonial-to-social.n8n.json) |

**Heavy-lifting removed:** write once and it fans out to SEO + socials (#1); a week of posts drafted for you (#2); IG/YouTube posting on autopilot (#3, #4); your inbox pre-triaged with draft replies (#6); meeting action items captured automatically (#7); reviews and newsletters that assemble themselves (#5, #9, #10).

**Apps:** OpenAI, Airtable, Gmail, X/Twitter, Instagram, YouTube, SMTP/Resend, Discord webhook. Note #3, #4 use Instagram/YouTube — both already connected in your Zapier if you port them there.
