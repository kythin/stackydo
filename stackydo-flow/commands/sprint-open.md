---
description: Open a new sprint. Enforces the one-sprint-at-a-time invariant. With a slug, opens that sprint. Without a slug, surfaces queued frozen plans + a "Start fresh" path so you don't need to know the slug.
argument-hint: [slug] (kebab-case, e.g. "auth-revamp" or "media-library" — omit to discover)
---

The argument is an optional sprint slug. If passed, open that sprint. If omitted, discover candidates first.

## Pre-flight: workspace bootstrap (first stackydo use this session)

Before anything else, check whether the project has a `stackydo.json`. If not (and the user is in a project directory, not their home dir), invoke the `stackydo-bootstrap` skill. Skip if already invoked or declined in this session.

## Step 0: One-sprint-at-a-time invariant

`grep -lE '^(\*\*)?Status:(\*\*)? +active\b' docs/sprints/*.md 2>/dev/null` (regex tolerates both `Status: active` and `**Status:** active` formats) — filter out `_template.md`. If `docs/sprints/` doesn't exist yet, this is the project's first sprint — skip the invariant check and continue.

- **0 matches** → proceed.
- **1 match, same slug as the requested slug** → refuse: "Sprint `<slug>` is already open at `<path>`. Append decisions to that file; don't reopen."
- **1 match, different slug** → refuse: "Sprint `<active-slug>` is currently active at `<active-path>`. Only one sprint runs at a time — close or abandon it first with `/sprint-close <active-slug>`."
- **>1 match** → refuse and surface them: "Invariant broken — two sprints are active: `<a>`, `<b>`. Close one before opening anything new."

Refusal is hard. The one-sprint-at-a-time invariant is deliberate. Don't offer to override.

## Discovery (only when no slug was passed)

1. **Queued frozen plans** — `ls docs/sprints/plans/*.md 2>/dev/null`. By convention plan filenames are slug-only (`<slug>.md`, no date prefix). A plan is a candidate when no sprint file under `docs/sprints/` ends in `-<slug>.md`. Read the first heading or "**Status:**" / "**Theme:**" line for a one-line summary.
2. **Surface via `AskUserQuestion`** — present the queued plans as options plus a final "Start fresh (no plan)" option. Pick the resolved slug from the user's choice.
3. **No queued plans** → ask the user for a slug as a freeform question. Quote it back in `YYYY-MM-DD-<slug>.md` form so they can confirm.

One question at a time during discovery — don't batch.

## Steps (after slug is resolved + invariant passed)

1. **Today's date.** Use the current date in `YYYY-MM-DD` format (check the system context).
2. **Filename.** `docs/sprints/YYYY-MM-DD-<slug>.md`. Refuse if the file already exists — the user is trying to re-open a sprint, which means they want a successor or to revive it, not a stomp.
3. **Bootstrap if needed.** If `docs/sprints/` doesn't exist yet, create it. If `docs/sprints/_template.md` doesn't exist, author a minimal one alongside the first sprint (goal, in-scope, out-of-scope, acceptance gate, linked tasks, architecture-doc follow-through).
4. **Copy the template.** From `docs/sprints/_template.md`. Fill `Started:` and leave `Closed:` empty, `Status: active`.
5. **Goal + scope.** If the frozen plan already covers goal + in-scope + out-of-scope, lift those and quote back what you'll write for confirmation. Otherwise ask in **three separate turns** via `AskUserQuestion` (or freeform, one at a time) — goal first, then in-scope, then out-of-scope. Don't batch the three asks. Quote back each answer before writing.
6. **Linked stackydo tasks.** Look for tasks the user mentioned by short_id or by topic. If the user has open `idea` or `triage` items in the relevant area, surface them and ask which to attach. For each attached task, add the `sprint-<slug>` tag (no leading date) via `mcp__plugin_stackydo-flow_stackydo__update_task`.
7. **Detail plan?** If discovery resolved this from a queued plan, the plan already exists at `docs/sprints/plans/<slug>.md` — just add a "**Detail plan:** [`plans/<slug>.md`](./plans/<slug>.md)" line near the top of the sprint file. Otherwise ask if a plan-mode session preceded this sprint; if yes, have the user point at the plan, drop it under `docs/sprints/plans/<slug>.md`, prepend a frozen-artefact header noting it's frozen and pointing back at the live sprint, and add the "Detail plan:" reference line to the sprint file.
8. **Roadmap.** Edit `docs/roadmap.md` if it exists:
   - Move the entry from "Next up" or "Queued" (if it lived there) into "Active".
   - If "Active" was `(none — …)`, replace that with the new sprint bullet.
   - One-line summary linking to the sprint file at its current (non-archive) path.
9. **Sprint README.** Edit `docs/sprints/README.md` if it exists — add a bullet to the "Current + recent sprints" list pointing at the new sprint.
10. **Commit and push.** Single conventional commit `docs(sprints): open <slug>`. End with the standard Co-Authored-By trailer.

## Constraints

- Don't fabricate goal / scope content. Ask, then write it back for the user to approve.
- Don't touch `docs/architecture/` or `docs/runbooks/`. Sprint opens don't change living architecture.
- Don't push if the user signals uncertainty about the goal — let it sit as an unstaged draft until they're sure.
- The one-sprint invariant from Step 0 is non-negotiable. If a previous sprint should have been closed and wasn't, the right move is `/sprint-close <other-slug>` first, then re-run `/sprint-open`.
