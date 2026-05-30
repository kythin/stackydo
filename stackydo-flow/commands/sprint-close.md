---
description: Close + archive the active sprint. With a slug, closes that sprint. Without a slug, defaults to the single active sprint (per the one-sprint-at-a-time invariant). Updates status + retro, git-mvs the file (and plan, if any) into archive, fixes inbound links, refreshes the roadmap.
argument-hint: [slug] (optional — defaults to the single active sprint; pass to disambiguate or to abandon a non-active file)
---

The argument is an optional sprint slug (with or without the `YYYY-MM-DD-` prefix).

## Pre-flight: workspace bootstrap (first stackydo use this session)

Before anything else, check whether the project has a `stackydo.json`. If not (and the user is in a project directory, not their home dir), invoke the `stackydo-bootstrap` skill. Skip if already invoked or declined in this session. Note that a project with no `stackydo.json` is unlikely to have an active sprint in the first place, but the check is cheap and keeps the entry-point pattern consistent.

## Resolve the target sprint

- **Slug passed** → resolve via `ls docs/sprints/*<slug>*.md`. If multiple match, ask; if none match, refuse with a useful error.
- **No slug passed** → `grep -l '^Status: active' docs/sprints/*.md 2>/dev/null` (filter out `_template.md`):
  - **1 match** → use it. Print "Closing the active sprint: `<slug>`." and continue.
  - **0 matches** → refuse: "No active sprint to close. If you meant to archive an already-inactive file, pass the slug explicitly."
  - **>1 match** → refuse: "Invariant broken — multiple sprints are active (`<a>`, `<b>`). Pass the slug to disambiguate; fixing the invariant is a separate `/sprint-close` call."

## Before doing anything

1. **Confirm the sprint actually shipped (or was abandoned).** Read the sprint file. Walk the linked stackydo tasks (`mcp__plugin_stackydo-flow_stackydo__list_tasks` with `tag=sprint-<slug>`). If any are `in_progress` or `in_review`, surface them and refuse to close — the user either finishes them, demotes them out of the sprint, or signals they're abandoning.
2. **Verify architecture docs were updated** for the surfaces this sprint touched. Read the sprint file's "Architecture-doc follow-through" section (or equivalent). Spot-check that each cited doc was actually updated since the sprint opened (`git log -- <doc>` since the sprint's `Started:` date). If any aren't, **block the close** and file a `triage,bug` stackydo task under the docs stack for each missing update.
3. **Carry-over items.** Anything in the sprint that didn't ship but is still wanted — promote it to a fresh stackydo task in its work stack (drop the `sprint-<slug>` tag). Don't leave dangling refs in the closed file.

## Closing steps

1. **Sprint file.** Set `Closed: YYYY-MM-DD` (today), `Status: shipped` (or `abandoned`), fill the retro. Confirm the retro draft with the user before writing.
2. **Archive move.** `git mv docs/sprints/<file> docs/archive/sprints/<file>`. If `docs/sprints/plans/<slug>.md` exists, also `git mv` it to `docs/archive/sprints/plans/<slug>.md`. Create `docs/archive/sprints/` and `docs/archive/sprints/plans/` if they don't exist yet (first close).
3. **Fix relative links in the moved files**:
   - `../architecture/…` → `../../architecture/…`
   - `../runbooks/…` → `../../runbooks/…`
   - `./README.md` → `../../sprints/README.md`
   - `./<sibling-sprint-file>.md` (a still-active sibling) → `../../sprints/<sibling-sprint-file>.md`. If the sibling has also been archived since this sprint opened, point at `./<sibling-sprint-file>.md` instead (same archive folder). Check each sibling reference with `ls docs/sprints/<sibling> docs/archive/sprints/<sibling>` to know which form applies.
   - `./plans/<slug>.md` (in sprint file) keeps working (same-folder pair).
4. **Inbound refs**: `grep -rn "<slug>" docs/` to find every reference. Update at minimum:
   - `docs/roadmap.md` if present — move the entry from "Active" / "Recently shipped (pending)" parenthetical into the "Recently shipped" archive-path version. Drop any `(pending …)` caveats only if their preconditions actually cleared.
   - `docs/sprints/README.md` if present — flip the link to the archive path.
   - Sibling sprint files that referenced this one — fix to archive path.
5. **Commit and push.** Single conventional commit `docs(sprints): close + archive <slug>`. End body with the standard Co-Authored-By trailer.
6. **Stackydo bookkeeping.** Optional: add a final comment to each `tag=sprint-<slug>` task pointing at the merged commit / archive path.

## For abandonment (status=abandoned, mid-flight kill)

Same archive ritual, but the retro should focus on *why the premise stopped being true* — not what shipped. Walk every linked task: keep `done` ones, demote ones that are still wanted (`ready` → `todo` in work stack, drop `ready` + `sprint-<slug>` tags), cancel ones that are now pointless. **No `in_progress` task may carry the `sprint-<slug>` tag after abandonment.**

## Constraints

- Don't lie in the retro. If a sprint shipped with known gaps, list them in "Carry-over" with the new stackydo IDs you filed.
- Don't push if the architecture-docs gate fails. Block, file follow-ups, escalate to the user.
