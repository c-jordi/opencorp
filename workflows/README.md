# Workflows

Scheduled and on-demand routines for running the company. Each routine is a markdown spec in this directory; the runnable version lives as a slash command in `.claude/skills/<name>/SKILL.md`.

## How a workflow is structured

Every spec has:

- **Frontmatter** — `name`, `cadence`, `trigger`, `owner`, `required_mcps`, `optional_mcps`, `output`.
- **Purpose** — one paragraph: what decision or artifact this produces.
- **Inputs** — window, scope, identifiers to resolve.
- **Steps** — numbered, tool-by-tool. Read-only by default.
- **Output template** — exact shape of the artifact.
- **Guardrails** — what the routine must not do (send messages, mutate state, etc.).

## Invocation (v1)

Manual slash commands only. Each spec has a matching skill so it is invokable as `/<name>`. Scheduling (GitHub Actions cron firing Claude Code on the web sessions, or `loop` within a session) is layered in once the prompts are stable.

## Catalogue

| Status | Cadence | Routine | Primary MCPs | Trigger |
|---|---|---|---|---|
| ✅ | Weekly | [weekly-slack-review](./weekly-slack-review.md) — classify Slack interactions, surface automation candidates | Slack | Fri 16:00 |
| 🟡 | Daily | morning-briefing — calendar + urgent threads + top Linear items | Calendar, Gmail, Slack, Linear | Weekday 08:30 |
| 🟡 | Daily | eod-digest — what shipped / what's blocked | Linear, GitHub | Weekday 18:00 |
| 🟡 | Weekly | week-kickoff — meetings, deliverables, focus blocks | Calendar, Linear, Notion | Mon AM |
| 🟡 | Weekly | inbox-sweep — unactioned email threads → tasks or drafts | Gmail, Linear | Fri PM |
| 🟡 | Weekly | pipeline-review — stale HubSpot deals, next steps | HubSpot, Granola | Fri PM |
| 🟡 | Weekly | pr-debt-sweep — open PRs, stale reviews | GitHub, Linear | Fri PM |
| 🟡 | Bi-weekly | cycle-close — Linear cycle outcomes + writeup | Linear, Notion | End of cycle |
| 🟡 | Monthly | customer-health — Granola transcripts × deal stage | Granola, HubSpot, Notion | 1st of month |
| 🟡 | Monthly | doc-audit — stale Drive + Notion hygiene | Drive, Notion | 1st of month |
| 🟡 | Quarterly | okr-review — OKR + hiring pipeline | Notion, HubSpot, Linear | Quarter end |

Legend: ✅ implemented · 🟡 planned

## Conventions

- **Read-only first.** v1 routines never send messages, file issues, or mutate CRM state without explicit confirmation in the same session.
- **Self-DM for delivery.** Default output is a Slack DM to the owner. Heavier artifacts (multi-section reviews) may be paired with a Notion page or canvas — declared explicitly in the spec's `output` field.
- **Time window.** All "weekly" routines use Mon 00:00 → Fri 15:00 local unless otherwise specified.
- **Identifier resolution.** Resolve the owner's Slack/Gmail/Linear ids once at the start of the routine; cache nothing across sessions.
