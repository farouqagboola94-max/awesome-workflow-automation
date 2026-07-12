# 🛠️ Dev-Ops — Repo Automation Playbook

Eight **n8n** workflows that automate maintenance of the GitHub repos themselves — issue triage, CI alerts, releases, list hygiene, and contributor experience. Tuned for an **awesome-list** repo and the **council-of-high-intelligence** repo, but generic enough for any project.

> Importable n8n JSON, shipped **inactive** with `$env` placeholders. GitHub steps use n8n's GitHub node (or generic HTTP + a PAT). Map 1:1 to Zapier/Make with their GitHub apps.

| # | Workflow | Trigger | File |
|---|----------|---------|------|
| 1 | Issue AI Triage | Issue-opened webhook | [`01-issue-ai-triage.n8n.json`](./01-issue-ai-triage.n8n.json) |
| 2 | PR CI-Failure Notifier | workflow_run webhook | [`02-pr-ci-failure-notifier.n8n.json`](./02-pr-ci-failure-notifier.n8n.json) |
| 3 | Release Notes On Tag | Tag-push webhook | [`03-release-notes-on-tag.n8n.json`](./03-release-notes-on-tag.n8n.json) |
| 4 | Stale Issue & PR Sweeper | Daily schedule | [`04-stale-issue-sweeper.n8n.json`](./04-stale-issue-sweeper.n8n.json) |
| 5 | Awesome-List Dead-Link Checker | Weekly schedule | [`05-awesome-link-checker.n8n.json`](./05-awesome-link-checker.n8n.json) |
| 6 | Contribution Format Lint | PR-opened webhook | [`06-contribution-format-lint.n8n.json`](./06-contribution-format-lint.n8n.json) |
| 7 | New Contributor Welcome | PR-opened webhook | [`07-new-contributor-welcome.n8n.json`](./07-new-contributor-welcome.n8n.json) |
| 8 | Council Agent Convention Validator | PR webhook (council repo) | [`08-council-convention-validator.n8n.json`](./08-council-convention-validator.n8n.json) |

## Highlights

- **#1 Issue AI Triage** — labels/prioritizes new issues, distinguishes a *tool submission* from a bug, auto-closes obvious spam, and posts a maintainer-style comment.
- **#5 Dead-Link Checker** — parses every Markdown link in `readme.md`, HEAD-checks them weekly, and opens a single issue listing any that return ≥400. Directly guards this list's core value: no dead links.
- **#6 Contribution Format Lint** — validates that new entries match `- [RESOURCE](LINK) — DESCRIPTION.` from `CONTRIBUTING.md` and nudges on alphabetical order, so maintainers don't hand-check formatting.
- **#8 Council Convention Validator** — for `farouqagboola94-max/council-of-high-intelligence`: on any PR touching `agents/council-*.md`, checks the required section order (Identity → Grounding Protocol → …) and that Grounding Protocol sits immediately after Identity, per that repo's `CLAUDE.md`. Comments any deviations and reminds to run `./scripts/council-simulation-checklist.sh`.

## Setup

1. **Import** each workflow into n8n.
2. **Credentials:** GitHub PAT (repo scope) for the GitHub node + generic HTTP Header Auth (`Authorization: Bearer <token>`) for raw API calls, OpenAI, Slack OAuth2.
3. **Environment variables:** `GH_REPO` (e.g. `farouqagboola94-max/awesome-workflow-automation`), `GH_OWNER`, `GH_REPO_NAME`.
4. **Webhooks:** add repository webhooks (or a GitHub Actions `workflow` that POSTs) for `issues`, `pull_request`, `workflow_run`, and `create` (tag) events pointing at the matching n8n paths.
5. Test in n8n *Test* mode, then toggle **Active**.

> These run **outside** the repo, so they work regardless of Actions minutes and can post to Slack/Discord that Actions can't reach easily. Several overlap with GitHub Actions — pick whichever fits your stack.
