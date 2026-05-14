---
name: weekly-slack-review
description: Friday-afternoon retro on the owner's Slack interactions for the week. Classifies each interaction (keep manual / templatable / automatable / route / eliminate), surfaces top automation candidates, and delivers a structured summary as a Slack DM to the owner. Use when the user asks for a "weekly Slack review", "Friday retro", "Slack interaction audit", or runs /weekly-slack-review. Requires the Slack MCP.
---

# Weekly Slack review

You are running the `weekly-slack-review` routine. The full spec is at `workflows/weekly-slack-review.md` — read it if you need detail. The condensed runbook follows.

## Preconditions

- Slack MCP must be available (`mcp__*__slack_*` tools). If it is not, stop and tell the user.
- Confirm the owner. Default: the authenticated Slack user. If ambiguous, ask once via `AskUserQuestion`.

## Window

Monday 00:00 → Friday 15:00 in the owner's local timezone, current week. If today is not Friday, run anyway but note the off-cycle execution in the output header.

## Procedure

1. **Resolve owner identity.** Use `slack_search_users` to get the owner's user id and display name.
2. **Find surfaces.** Use `slack_search_channels` for member channels; combine with DMs. Filter to those where the owner sent ≥1 message in the window (peek with `slack_read_channel`).
3. **Collect activity.** For each surface, pull messages in the window. For any thread the owner participated in, expand with `slack_read_thread`.
4. **Classify.** For each distinct exchange, record: counterparty/channel, topic phrase, owner role (`answered-question` · `made-decision` · `unblocked` · `status-update` · `fyi` · `social` · `coordination`), effort (`S` <2m, `M` 2–15m, `L` >15m), and an automation hypothesis (one sentence).
5. **Cluster + tag.** Group by topic + role and tag each cluster: **Keep manual**, **Templatable**, **Automatable**, **Route elsewhere**, **Eliminate**.
6. **Recommend.** For each non-"keep" cluster, propose exactly one concrete mechanism (runbook page, Notion FAQ entry, Linear ticket, Slack workflow, or new `workflows/` routine) and a rough build effort.
7. **Deliver.** Send a single Slack DM to the owner via `slack_send_message`. Use the output template below verbatim. Do not post anywhere else; do not file tickets; do not react or reply to other messages.

## Output template

```
Weekly Slack review — week of <Mon date> → <Fri date>
(<note: off-cycle execution> if applicable)

Activity
- <N> interactions across <M> surfaces · S:<n>  M:<n>  L:<n>
- Top counterparties: <name×count>, <name×count>, <name×count>

By category
| Category    | Count | % effort | Example topic |
|-------------|-------|----------|---------------|
| Keep manual |       |          |               |
| Templatable |       |          |               |
| Automatable |       |          |               |
| Route       |       |          |               |
| Eliminate   |       |          |               |

Top 3 automation candidates
1. <cluster> — mechanism: <…>. Effort: <S/M/L>. Saves ~<N> exchanges/week.
2. …
3. …

Process / routing issues
- <bullet>

Operationalize on Monday
- [ ] <action>
- [ ] <action>
- [ ] <action>
```

## Guardrails

- Slack is **read-only** except for the final self-DM.
- Do **not** create Linear issues, Notion pages, or canvases as part of this routine — propose them in the summary instead.
- If total messages in the window exceed 500, switch to per-channel sampling and state the sampling rule in the output.
- If the owner cannot be resolved, stop and report rather than guessing.

## When done

Reply in chat with: (1) the Slack DM permalink (from `slack_send_message` response), and (2) a one-line tally — total interactions, automation candidates surfaced.
