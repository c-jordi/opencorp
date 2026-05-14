---
name: weekly-slack-review
cadence: weekly
trigger: Friday 16:00 local
owner: <you>
required_mcps: [slack]
optional_mcps: [linear, notion]
output:
  primary: slack_dm_self
  secondary: null
---

# Weekly Slack interaction review

## Purpose

Each Friday, look back at the week's Slack activity and classify your interactions into **keep manual / templatable / automatable / route elsewhere / eliminate**. Produce a self-DM summary with the top automation candidates and concrete next steps, ready for operationalization on Monday.

## Inputs

- **Time window.** Monday 00:00 → Friday 15:00, owner's local timezone.
- **Scope.** DMs and channels where the owner sent at least one message in the window. Threads count as their parent channel.
- **Owner identity.** Resolve via `slack_search_users` for the owner's handle once at start.
- **Optional filters.** Skip channels tagged `#social`, `#random`, or otherwise marked low-signal in a future `workflows/config.md`.

## Steps

1. **Resolve owner.** `slack_search_users` → owner's user id.
2. **Enumerate candidate surfaces.** `slack_search_channels` for channels the owner is a member of; for each, peek with `slack_read_channel` to confirm owner participation in window. Include DMs.
3. **Pull activity.** For each surface, fetch messages in window. Expand threads via `slack_read_thread` when the owner replied inside a thread.
4. **Classify each interaction.** For every distinct exchange the owner participated in, capture:
   - counterparty (or channel)
   - topic (1 short phrase)
   - owner's role: `answered-question` · `made-decision` · `unblocked` · `status-update` · `fyi` · `social` · `coordination`
   - rough effort: `S` (<2 min), `M` (2–15 min), `L` (>15 min)
   - automation hypothesis: would a runbook, FAQ, scheduled bot, or routing change have absorbed this?
5. **Cluster.** Group interactions by topic + role. Tag each cluster:
   - **Keep manual** — high-leverage, judgment-heavy
   - **Templatable** — same answer/structure repeated; needs a snippet or canned response
   - **Automatable** — could be a scheduled routine, Slack workflow, or self-serve doc
   - **Route elsewhere** — wrong person/channel; needs a routing rule
   - **Eliminate** — shouldn't have happened; upstream process fix
6. **Recommend.** For each non-"keep" cluster, propose one concrete next step: a runbook page, a Notion FAQ entry, a Linear ticket, a Slack workflow, or a new entry in this `workflows/` directory. Include rough effort to build.
7. **Deliver.** Send the owner a Slack DM (`slack_send_message` to self) with the structured summary below. Do not post anywhere else.

## Output template

```
Weekly Slack review — week of <Mon date> → <Fri date>

Activity
- <N> interactions across <M> surfaces · <S/M/L breakdown>
- Top counterparties: <name×count>, <name×count>, <name×count>

By category
| Category       | Count | % effort | Example                              |
|----------------|-------|----------|--------------------------------------|
| Keep manual    |   ..  |    ..%   | <topic>                              |
| Templatable    |   ..  |    ..%   | <topic>                              |
| Automatable    |   ..  |    ..%   | <topic>                              |
| Route          |   ..  |    ..%   | <topic>                              |
| Eliminate      |   ..  |    ..%   | <topic>                              |

Top 3 automation candidates
1. <cluster> — proposed mechanism: <runbook / workflow / routine / routing>. Effort: <S/M/L>. Saves ~<N> exchanges/week.
2. ...
3. ...

Process / routing issues
- <bullet>

Operationalize on Monday
- [ ] <action with proposed owner>
- [ ] <action>
- [ ] <action>
```

## Guardrails

- **Read-only on Slack** except for the final self-DM. No reactions, no replies, no posts to shared channels.
- **No issue/ticket creation** in Linear, Notion, or elsewhere. The summary proposes them; the owner files them after review.
- **No personally-identifying analysis** beyond Slack display names already visible to the owner.
- **Bounded scope.** If activity volume exceeds 500 messages in the window, sample by channel rather than processing everything; declare the sampling in the output.
