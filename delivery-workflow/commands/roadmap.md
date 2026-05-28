---
description: Roadmap-level planning ritual — surfaces sprint-candidate tasks and walks you through sequencing them into docs/roadmap.md sections (Next up / Queued / Out of view). Use after /triage when you have a handful of sprint-candidates that need to be ordered into a forward view. Does NOT touch Active or Recently shipped sections (owned by /sprint-open and /sprint-close).
---

The user wants to plan the upcoming sequence of sprints. Walk the sprint-candidates one at a time, decide where each goes, and commit a refreshed `docs/roadmap.md`.

## Pre-flight: workspace bootstrap (first stackydo use this session)

Before anything else, check whether the project has a `stackydo.json`. If not (and the user is in a project directory, not their home dir), invoke the `stackydo-bootstrap` skill. Skip if already invoked or declined in this session.

## Step 1: Read the current roadmap (or scaffold)

```bash
test -f docs/roadmap.md && echo "EXISTS" || echo "MISSING"
```

If **MISSING**, ask the user once (via `AskUserQuestion`) whether to scaffold `docs/roadmap.md`. If yes, write a minimal skeleton with the five canonical sections:

```markdown
# Roadmap

Cross-cutting forward view. Sprints are the unit of commitment; this is the queue that feeds them.

## Active

_(nothing currently active)_

## Next up

_(none — promote sprint-candidates via `/roadmap`)_

## Recently shipped

_(none yet)_

## Queued

Real ideas worth doing, not committed to a build phase. Lives in stackydo: `list_tasks stack=ideas tag=draft`.

## Out of view

Deliberately rejected directions, with reasoning, so the same conversation doesn't keep reopening.
```

Then proceed. If the user declines the scaffold, exit — there's nothing to plan against.

If **EXISTS**, read it. Identify the headings for the five sections so you know where to splice updates later. If any of the five canonical sections is missing (legacy roadmap with different shape), surface that and ask whether to add the missing one before continuing — don't silently rewrite the user's structure.

## Step 2: List sprint-candidates

```
mcp__plugin_delivery-workflow_stackydo__list_tasks stack=ideas tag=sprint-candidate
```

Group them by their domain tag (the non-lifecycle tag — strip `draft`, `sprint-candidate`, and `deferred`; the remainder is the domain).

If there are zero sprint-candidates, surface that plainly:

> No sprint-candidates in the ideas stack. Nothing to sequence. If you want to seed the roadmap from regular `tag=draft` ideas, run `/triage` first and promote clusters to `sprint-candidate` there.

Then stop. Don't fabricate work.

## Step 3: Read what's already in the roadmap

Parse the current "Next up", "Queued", and "Out of view" sections. For each sprint-candidate from Step 2, check whether it's already listed in one of those sections (match by short_id or by sprint slug if one exists). Three buckets going into Step 4:

- **Unplaced** — sprint-candidates not yet in any section. The primary input to walk.
- **Already placed** — surface them as a "what's already on the roadmap" header before walking, but don't re-walk them unless the user asks.
- **Roadmap entries with no matching task** — entries in "Next up" or "Queued" that point at a `sprint-candidate` task that's been completed, cancelled, or deleted. Flag for cleanup at the end.

## Step 4: Walk the unplaced candidates

For each unplaced sprint-candidate, present:

- Title
- Short_id
- Domain tag
- First 2-3 lines of the body (goal section if structured per the lifecycle skill)

Then `AskUserQuestion`:

> Where does `<title>` go?
> Options:
> - **Next up** — commit to plan this. Higher priority; sequenced near the top.
> - **Queued** — real and wanted, but not soon.
> - **Out of view** — deliberately not doing. Will record the reasoning.
> - **Skip** — leave it in `ideas` for now, decide later.

Handling per option:

### Next up
- Ask one follow-up: "Where in the Next up order?" with options matching the current count: `Top`, `After <first existing>`, `After <second existing>`, ..., or `Bottom`. If Next up is empty, skip the ordering question and just place it.
- Optionally ask: "Spawn `sprint-planner` now to draft a frozen plan stub at `docs/sprints/plans/<slug>.md`?" — Yes / No / Later. If Yes, capture the slug (slugify the title or ask) and queue a `sprint-planner` invocation for after the walk completes.
- The sprint-candidate stays in `stack=ideas` until `/sprint-open` runs against it. We're updating the roadmap *pointer*, not promoting the task.

### Queued
- No follow-up. Add as a bullet to the Queued section.
- The Queued section is just a pointer to `list_tasks stack=ideas tag=draft` per convention. Listing it inline is allowed when you want it on a human-readable map.

### Out of view
- Ask one follow-up: "Why? (One line of reasoning so this doesn't keep reopening.)" — freeform.
- Write the bullet as: `- <title> — <reasoning>`
- Add `tag=deferred` to the task via `update_task <id> tags=...` so it falls out of the default ideas view too.

### Skip
- No-op. Move to the next.

After each placement, update an in-memory draft of the roadmap sections; don't write to disk until Step 6.

## Step 5: Stale-entry cleanup

If Step 3 flagged any roadmap entries pointing at sprint-candidates that no longer exist (completed / cancelled / deleted tasks), surface them at the end of the walk:

> The roadmap references these sprint-candidates that are no longer in the ideas stack:
> - `<title>` (Next up) — task short_id `<id>` is `<status>` / missing.
>
> Remove?

`AskUserQuestion` per entry: **Remove** / **Keep** / **Move to Recently shipped (if done)**.

Don't auto-remove. Surfacing-and-confirming respects the audit trail.

## Step 6: Write and commit

1. Show the user the full updated **Next up**, **Queued**, and **Out of view** sections verbatim before writing. Quote the diff in plain markdown.
2. Confirm via `AskUserQuestion`: **Write and commit** / **Edit** / **Cancel**.
3. On confirm:
   - Edit `docs/roadmap.md` — replace the three sections, leave Active and Recently shipped untouched.
   - For any sprint-candidate placed in "Out of view", run the `update_task <id> tags=draft,sprint-candidate,deferred,<domain>` write.
   - Commit: `docs(roadmap): re-sequence Next up + Queued` (or similar — name it for what actually changed).
   - Push.

## Step 7: Sprint-planner spawns (if any queued from Step 4)

If the user opted to spawn `sprint-planner` for one or more Next-up items, fire those now via the `Agent` tool — one agent per slug, in a single message so they run concurrently if file overlap allows. Each prompt should include:

- The sprint-candidate's short_id and full body (so the planner has the scope artefact).
- The expected plan filename: `docs/sprints/plans/<slug>.md`.
- The reminder that this produces a frozen plan, not a sprint file — `/sprint-open` is the separate step.

Report each plan's path when the agents complete. Don't wait to start them in series — they're independent design work.

## Constraints

- **Don't touch Active or Recently shipped.** Those are owned by `/sprint-open` and `/sprint-close` respectively. If the user wants to manually edit those sections, that's their call; this command stays in its lane.
- **Don't promote sprint-candidates out of `stack=ideas`.** The candidate stays in ideas until `/sprint-open` runs against it. Only Out-of-view items get a tag change (`+deferred`).
- **Don't ask all the placement questions in one batch.** One candidate at a time, `AskUserQuestion` per item.
- **Don't fabricate sequencing.** If the user picks "Top" but there's existing content, ask before reordering — don't assume.
- **Don't write `Recently shipped` from this command** even if the user mentions a sprint that finished. Tell them to use `/sprint-close` for that.
- **If `docs/roadmap.md` was scaffolded fresh this run**, the first commit message should reflect that: `docs(roadmap): scaffold + initial planning` rather than re-sequence wording.
