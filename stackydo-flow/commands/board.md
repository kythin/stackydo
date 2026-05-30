---
description: Show the current stackydo board (in-flight work + next-up queue + active-sprint progress + recent commits). The session-start ritual in one keystroke. For "what should I do next?", use /next instead.
---

## Pre-flight: workspace bootstrap (first stackydo use this session)

Before any stackydo calls, check whether the project has a `stackydo.json`. If not (and the user is in a project directory, not their home dir), invoke the `stackydo-bootstrap` skill. The skill will detect the state, confirm once with the user, and either bootstrap project-local storage or proceed with the home-workspace fallback. Skip this step if the user already declined in this session.

## Action

Read the current state in parallel, then summarise.

Run in parallel:

1. `mcp__plugin_stackydo-flow_stackydo__get_stats` — total task counts and breakdowns.
2. `mcp__plugin_stackydo-flow_stackydo__list_tasks` with `status=in_progress` — what's mid-flight.
3. `mcp__plugin_stackydo-flow_stackydo__list_tasks` with `stack=ready` and `limit=5` — what's next.
4. `mcp__plugin_stackydo-flow_stackydo__list_tasks` with `stack=journal`, `limit=1`, `sort=created`, `reverse=true` — the last handoff (so resume sessions land on context).
5. `grep -l '^Status: active' docs/sprints/*.md 2>/dev/null` (filter out `_template.md`) — find the active sprint file (expect 0 or 1, per one-sprint-at-a-time). If `docs/sprints/` doesn't exist, skip this and treat as "no active sprint".
6. `git log --oneline -5` — what just landed.

Then, **only if an active sprint was found**, run `mcp__plugin_stackydo-flow_stackydo__list_tasks tag=sprint-<slug>` and bucket by status.

## Output shape

```
In flight: <N tasks — short_id + title each line, or "(none)">
Next up (ready): <up to 5, short_id + title>
Active sprint: <slug — N todo · M in_progress · K done [· (close-ready) if no open work remains: todo + in_progress + in_review + blocked + on_hold = 0]>, or "(none active)"
Last handoff: <short_id> — <title> (<N>h ago)    ← only render if a journal entry exists AND it's from a previous calendar day. Same-day entries are noise during the session that wrote them.
Recent commits: <5 oneliners>
```

End with a one-sentence **observation** about state — not an offer of work. Examples:

- "The active sprint has no remaining open tasks."
- "Ready queue is empty; nothing committed for the next session."
- "Three tasks in flight — heaviest concurrent load in a while."

Don't write "want me to also do X, Y?" or "shall I run /sprint-close?". The observation is for the user's awareness; `/next` is the dispatcher.

## Constraints

- If `>1` sprint matches `Status: active`, the invariant is broken — surface that as "Active sprint: ⚠ invariant broken — `<a>`, `<b>` both active. Run `/sprint-close <slug>`."
- If `tag=sprint-<slug>` returns no tasks, print "Active sprint: `<slug>` — no tasks tagged yet" (legitimate state right after `/sprint-open`).
- Keep it tight. No preamble, no padding. The user is orienting, not reading.
