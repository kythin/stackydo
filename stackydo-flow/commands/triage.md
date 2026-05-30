---
description: Walk the weekly triage ritual from the delivery-lifecycle skill — agent-spotted bugs awaiting prioritisation, then the ideas backlog, then on-hold items, then idea clusters dense enough to graduate to a sprint plan. Lets stale captures get acted on instead of rotting.
---

Walk four queues in order, surface each, and help the user act on items. Don't make decisions for them — present each item with enough context to decide and suggest the obvious move; the user redirects.

Use `AskUserQuestion` per item, one at a time. Don't batch the queue into a single prose dump.

## Pre-flight: workspace bootstrap (first stackydo use this session)

Before walking the queues, check whether the project has a `stackydo.json`. If not (and the user is in a project directory, not their home dir), invoke the `stackydo-bootstrap` skill. Skip if already invoked or declined in this session.

## Queue 1: Triage-tagged bugs (agent-spotted, real, awaiting prioritisation)

`mcp__plugin_stackydo-flow_stackydo__list_tasks` with `tag=triage`.

For each: title, priority (current best-guess), one-line summary of the claim. The call is:

- **Promote** → set priority, drop the `triage` tag → it becomes a normal `todo`.
- **Escalate** → set `priority=critical` and start `in_progress` now.
- **Downgrade** → move to `stack=ideas`, add `tag=draft`, drop `triage` (turns out it wasn't actually broken / not worth doing).

## Queue 2: Ideas backlog (raw captures)

`mcp__plugin_stackydo-flow_stackydo__list_tasks` with `stack=ideas` `tag=draft` — and **exclude `deferred`** (those are intentionally parked far-future ideas, surfacing them rots the queue). If the tool doesn't support exclusion in one call, filter the result.

For each: title, age, any comments. The call is:

- **Refine** → move to a work stack, drop `draft` tag, add type/domain tags.
- **Park** → set `status=on_hold` if it's real but not now.
- **Defer** → add `tag=deferred` if it's far-future with no near-horizon trigger.
- **Kill** → `cancelled` with a one-line reason in `add_comment`.

## Queue 3: On-hold items (parked, may need reviving)

`mcp__plugin_stackydo-flow_stackydo__list_tasks` with `status=on_hold`.

For each: title, what was on hold for, how long it's been there. The call is:

- **Revive** → set `status=todo`, drop the hold reason.
- **Park longer** → leave it, add a comment explaining why.
- **Kill** → `cancelled` with reason.

## Queue 4: Sprint-promotion candidates (idea clusters dense enough to graduate)

Take the Queue 2 result (`stack=ideas tag=draft` minus `deferred`) **after** the user has worked through it — promotions in Queue 2 may have thinned the dataset. Group what remains by topic tag.

Any topic tag with **≥3 ideas** is a sprint-promotion candidate. Surface each cluster with its tag, count, and the cluster's titles (one line each). The call is:

- **Promote to plan** → drop a stub at `docs/sprints/plans/<slug>.md` (or append to an existing stub) listing the cluster's task short_ids. Don't open the sprint — `/sprint-open` is a deliberate, separate act. The stub goes into the queued-plans pile that `/next` reads.
- **Park** → leave the cluster, but optionally tag one or two members `tag=ready` if the user wants them pulled into the next session without sprint-scoping.
- **Leave** → no-op, the cluster stays as raw ideas.

If no topic tag has ≥3 ideas after Queue 2, skip Queue 4 with a one-liner ("No idea cluster is dense enough to graduate.").

## Output discipline

After each queue, pause for the user. Don't process all four in one monologue. Don't dump full task bodies — title + one-line context is enough. Anything they want refined, they'll tell you.

At the end, summarise: how many items moved through each queue, what stack-counts changed, and whether any sprint plan stubs were dropped under `docs/sprints/plans/`. Stop.
