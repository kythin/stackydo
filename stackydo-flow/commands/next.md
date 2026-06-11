---
description: Decide what to do next. Reads loop state (active sprint, in-progress tasks, ready queue, triage backlog, queued frozen plans) and points at the right next action — pick up a task, close a shippable sprint, open a queued plan, or run triage. Use when you don't want to inventory the state yourself.
---

Read the whole delivery-loop state in parallel, then dispatch via the decision tree below. The output is one paragraph of context and one of: a quoted next command, a "just resume" pointer at an in-progress task, or an `AskUserQuestion` when the right next thing is genuinely ambiguous. Never a freeform prose menu.

## Pre-flight: workspace bootstrap (first stackydo use this session)

Before reading state, check whether the project has a `stackydo.json`. If not (and the user is in a project directory, not their home dir), invoke the `stackydo-bootstrap` skill. Skip if already invoked or declined in this session.

## State assessment

Run these in parallel (skip anything that depends on a folder that doesn't exist):

1. **Active sprint?** `ls docs/sprints/*.md 2>/dev/null | grep -v _template` and `grep -lE '^(\*\*)?Status:(\*\*)? +active\b' docs/sprints/*.md 2>/dev/null` (regex tolerates both `Status: active` and `**Status:** active` formats). Expect 0 or 1 (one-sprint-at-a-time invariant). If >1, the invariant is broken — surface that as the answer.
2. **In-progress tasks?** `mcp__plugin_stackydo-flow_stackydo__list_tasks status=in_progress`.
3. **Active sprint task buckets?** If an active sprint exists, `mcp__plugin_stackydo-flow_stackydo__list_tasks tag=sprint-<slug>` and bucket by status.
4. **Ready queue?** `mcp__plugin_stackydo-flow_stackydo__list_tasks stack=ready limit=3`.
5. **Triage backlog size?** Two calls: `mcp__plugin_stackydo-flow_stackydo__list_tasks tag=triage` and `mcp__plugin_stackydo-flow_stackydo__list_tasks stack=ideas tag=draft`. Subtract any with `tag=deferred` from the second.
6. **Queued sprint plans?** `ls docs/sprints/plans/*.md 2>/dev/null` minus any whose `<slug>.md` (or `*-<slug>.md`) exists in `docs/sprints/`. Those are plans authored but not yet promoted to a sprint.

## Dispatch decision tree

Top match wins. Stop at the first branch that fires.

1. **Invariant broken** (>1 active sprint) → "Two sprints are active: `<slug-a>`, `<slug-b>`. Close one before doing anything else. Run `/sprint-close <slug>`."
2. **A task is `in_progress`** → "Resume: <short_id> — `<title>` (stack=`<stack>`). Pick that up before starting anything new." No `/`-command suggestion — just resume.
3. **Active sprint + open tagged tasks** → "Active sprint: `<slug>` (N todo · M in_progress · K done). Pick up: <short_id> — `<title>`." Choose the task by: top of `ready` stack with the sprint tag, else top `todo` with the sprint tag (highest priority). If none qualify but in-progress exists under the sprint tag, branch 2 already won.
4. **Active sprint, no open work under the sprint tag** (no `todo`, `in_progress`, `in_review`, `blocked`, or `on_hold` tasks tagged `sprint-<slug>`) → "Sprint `<slug>` looks shippable — no open work left under `tag=sprint-<slug>`. Run `/sprint-close <slug>`."
5. **No active sprint, queued plan exists** → If exactly one plan is queued: "Queued sprint plan: `<slug>`. Run `/sprint-open <slug>` to start it." If multiple plans are queued: use `AskUserQuestion` with one option per plan (label = slug, description = first line of the plan file) plus a "None — pick later" option. Don't prose-list them.
6. **No active sprint, no queued plan, triage backlog heavy** (≥10 untriaged across `tag=triage` + filtered `ideas/draft`) → "Triage backlog is heavy (N triage · M ideas). Run `/triage`."
7. **No active sprint, no queued plan, light triage** → "No active work. Nothing in the queued-plans folder, no `in_progress` tasks, triage backlog is light (N items). Time to pick a fresh theme — browse `stackydo list stack=ideas` or open a sprint deliberately."

## Output discipline

- One paragraph of state, one quoted next command.
- Don't dump full task lists or sprint frontmatter — branch summary is enough.
- Don't offer "want me to also do X?" — the user has the next command and will type it.
- If the branch picked is ambiguous (e.g. two equally-top tasks in branch 3), pick one and say so in one short clause; don't ask.

## Constraints

- Read-only. `/next` doesn't mutate stackydo or sprint files. The user runs the suggested command if they want.
- The one-sprint-at-a-time invariant is real — if branch 1 fires, that's the answer.
- If `docs/sprints/plans/` doesn't exist, branch 5 is impossible — skip it without erroring.
