# 💼 Ops — Finance, Reporting & Admin Playbook

Ten **n8n** workflows that take the recurring back-office load off your plate: invoicing, revenue/P&L reporting, expense capture, data backup, anomaly alerts, lead routing, and cleanup.

> Importable n8n JSON, shipped **inactive** with `$env` placeholders. Map 1:1 to Zapier/Make.

| # | Workflow | Trigger | File |
|---|----------|---------|------|
| 1 | Auto-Invoice on Sale | Sale webhook | [`01-invoice-on-sale.n8n.json`](./01-invoice-on-sale.n8n.json) |
| 2 | Daily Revenue Reconciliation | Schedule | [`02-daily-revenue-reconciliation.n8n.json`](./02-daily-revenue-reconciliation.n8n.json) |
| 3 | Expense Receipt Capture | Email webhook | [`03-expense-receipt-capture.n8n.json`](./03-expense-receipt-capture.n8n.json) |
| 4 | Weekly P&L Snapshot | Schedule | [`04-weekly-pnl-snapshot.n8n.json`](./04-weekly-pnl-snapshot.n8n.json) |
| 5 | Contract & Renewal Reminder | Schedule | [`05-contract-expiry-reminder.n8n.json`](./05-contract-expiry-reminder.n8n.json) |
| 6 | Airtable → Google Sheets Backup | Schedule | [`06-airtable-sheets-backup.n8n.json`](./06-airtable-sheets-backup.n8n.json) |
| 7 | KPI Anomaly Alert | Schedule | [`07-kpi-anomaly-alert.n8n.json`](./07-kpi-anomaly-alert.n8n.json) |
| 8 | Lead Scoring & Routing | Lead webhook | [`08-lead-scoring-routing.n8n.json`](./08-lead-scoring-routing.n8n.json) |
| 9 | Duplicate Contact Cleanup | Schedule | [`09-duplicate-contact-cleanup.n8n.json`](./09-duplicate-contact-cleanup.n8n.json) |
| 10 | Cross-Project Weekly Digest | Schedule | [`10-cross-project-weekly-digest.n8n.json`](./10-cross-project-weekly-digest.n8n.json) |

**Heavy-lifting removed:** no more manual invoicing, spreadsheet reconciliation, receipt data-entry, renewal tracking, backups, or lead sorting. #10 rolls both events + the repo into one Monday email.

**Apps:** Airtable, QuickBooks, Google Sheets, Slack, HubSpot, OpenAI, SMTP/Resend. Set `$env`: `FINANCE_EMAILS`, `OWNER_EMAIL`, `BACKUP_SHEET_ID`, `EXPECTED_ORDERS_6H`.
