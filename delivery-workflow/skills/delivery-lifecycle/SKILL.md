---
name: delivery-lifecycle
description: >
  The full task lifecycle for the delivery-workflow plugin — idea, triage, todo,
  ready, in_progress, in_review, done, plus sprint-candidate, deferred, on_hold,
  blocked, cancelled. Trigger when the user asks "what status should this be",
  "how do I promote this", "move this to ready", "what's a sprint-candidate",
  "lifecycle slot", "promote idea", "graduate to sprint", "what stack should
  this go in", "where do bugs go", or any time the user is unsure how to capture,
  refine, or move a task through the loop. Also trigger when the user asks
  about stack vs tag, why status=ready doesn't work, or the rationale for
  splitting the backlog into idea / sprint-candidate / todo / ready.
---

# Delivery Lifecycle

This skill encodes the full lifecycle used by the delivery-workflow plugin's agents and commands. It assumes [stackydo](https://github.com/kythin/stackydo) as the task system.

## Why stack + tag, not status

Stackydo's built-in statuses are `todo, in_progress, in_review, blocked, on_hold, done, cancelled`. They cover the **active-state machine** (am I working on it right now? am I waiting?) but not the **commitment lifecycle** (is this a raw idea, a real bug, a ready-to-pull task, or a sprint candidate?).

Rather than extending stackydo's workflow config, this lifecycle uses **stack + tag** to carry the commitment slot, and leaves status to its job. `update_task status=ready` will be rejected — that's by design.

If a project later needs `idea` / `triage` / `ready` to be enforced rather than conventional, extend `stackydo.json` with `workflow.statuses` and migrate. Until then: stack + tag is the source of truth for slot, status is the source of truth for active state.

## The lifecycle

```
       ┌──────────────────── backlog ────────────────────┐   ┌──── active ────┐   ┌── archive ──┐
       │                                                 │   │                │   │             │
  →  idea  →  triage  →  todo  →  ready  →           in_progress  →  in_review  →  done
                                      ↘                   ↑                          ↑
                                        on_hold ──────────┘                          │
                                                                                cancelled
```

| Slot | Status | Stack | Tag | Meaning |
|---|---|---|---|---|
| `idea` | `todo` | `ideas` | `draft` | Raw capture. Cheap, judgement-free. May never happen. |
| `sprint-candidate` | `todo` | `ideas` | `draft, sprint-candidate` | Scoped idea that should graduate into a **sprint plan**, not a single task. Has research, citations, and decided scope. Promotion path is `/sprint-open`, not a move to `ready`. |
| `triage` | `todo` | (work stack) | `triage` | Confirmed real, awaiting prioritisation. |
| `todo` | `todo` | (work stack) | — | Refined, prioritised, not yet committed to a build phase. |
| `ready` | `todo` | `ready` | `ready` | **Selected for the next build phase. This is the queue you pull from.** |
| `on_hold` | `on_hold` | (any) | — | Parked with intent to revive in the foreseeable future. Was real, currently not urgent. |
| `deferred` | `todo` | `ideas` | `draft, deferred` | Far-future. Real idea, intentionally parked indefinitely until a concrete trigger surfaces. Differs from `on_hold` in that no revival is expected on any near horizon. |
| `in_progress` | `in_progress` | (work stack) | — | Actively being worked on. |
| `blocked` | `blocked` | (work stack) | — | In flight, waiting on something external. Use a `note` to point at the blocker. |
| `in_review` | `in_review` | (work stack) | — | PR up, awaiting review/CI. |
| `done` | `done` | (work stack) | — | Shipped. Terminal. |
| `cancelled` | `cancelled` | (any) | — | Won't do. Terminal. Keep for the audit trail. |

**Rule**: **stack + tag carry the lifecycle slot**, status carries the active-state machine only. `list_tasks stack=ready` is your sprint board; `list_tasks stack=ideas` is the parking lot.

## The three backlog slots, the rationale

Most teams collapse "ideas" and "backlog" into one bucket and the bucket rots. The split exists because **the cost of capturing an idea must be near-zero**, but **the cost of committing to a build phase must be deliberate**.

- `idea` (stack=`ideas`, tag=`draft`) is a parking lot. No judgement, no scope, no acceptance criteria.
- `sprint-candidate` (stack=`ideas`, tags=`draft,sprint-candidate`) is a scoped idea that's been worked up enough to know it should become a *sprint* (not a single task). Body carries scope decisions, file:line citations to prior art, integration touchpoints, and open questions for the sprint planner. Promotion path is **`/sprint-open` → `docs/sprints/<slug>.md`**, not "move to a work stack as a single task".
- `todo` (stack=work-area, no `draft`/`ready`/`triage` tag) is the refined backlog. Real work, scoped, prioritised, not yet promised.
- `ready` (stack=`ready`, tag=`ready`) is the promise. When it's `ready`, the next dev session can grab it without re-thinking.
- `deferred` (stack=`ideas`, tags=`draft,deferred`) is the indefinite parking lot — real ideas consciously parked. Filter out of the default ideas view so the active idea queue stays scannable.

Promotion paths:
- **Single-task ideas**: move out of the `ideas` stack into the relevant work stack and drop the `draft` tag. When selected for the next build phase, move into the `ready` stack and add the `ready` tag.
- **Sprint-candidates**: do *not* move to a work stack. Run `/sprint-open`, which creates `docs/sprints/<slug>.md` from the candidate's scope artefact. The candidate task stays in `ideas` until the sprint closes, at which point it's completed as "shipped via sprint <slug>".

Status stays `todo` the whole way through either path.

## Stack convention

One stack per task. Stacks separate workstreams; tags handle cross-cutting concerns.

Pick stack names that match how your work actually splits. Common patterns:

| Stack (example) | Scope (example) |
|---|---|
| `admin` | Backoffice / dashboard / portal app |
| `web` | Public site / runtime / customer-facing app |
| `infra` | CI/CD, deploy, secrets, monorepo plumbing |
| `docs` | Architecture, runbooks, project-level docs |
| `ideas` | Raw captures (paired with tag `draft`) |
| `ready` | Committed for next build phase (paired with tag `ready`) |
| `journal` | Session handoff records — one entry per `/handoff` invocation, paired with `tag=handoff`. Post-handoff observations land here as comments. |

If a task spans two work stacks, pick the side where the bulk of the work happens and tag the other (`tag:cross-stack`).

The `ideas`, `ready`, and `journal` stacks are structural — don't reuse those names for workstreams.

## Tag conventions

Lowercase, hyphenated, short.

- **Lifecycle**: `draft`, `triage`, `ready`, `deferred`, `sprint-candidate`
- **Type**: `bug`, `feature`, `refactor`, `tech-debt`, `security`, `perf`, `a11y`, `ux`, `data-model`
- **Domain** (yours — name them for your product surfaces)
- **Phase grouping** (optional): `phase:<label>` — e.g. `phase:2026-q3`, `phase:next`. Slice the `ready` stack into upcoming batches.
- **Source**: `pr-feedback`, `incident`, `customer-request`
- **Nit**: `nit` — UX/UI polish bundled into a later pass, not investigated now.

`list_tasks stack=ready tag=phase:next` = the next build queue.

## Standard rituals

### Start of every dev session

```
get_stats
list_tasks status=in_progress
list_tasks stack=ready limit=5
```

Three calls, ten seconds. You see what's in flight, what's next, and whether you're behind.

### Entry points (where tasks come from)

Three doors, three confidence levels.

**A. Claude spots a thought / TODO / refactor → `idea` slot.** Capture as `stack=ideas, tags=draft`, `status=todo`. Keep working.
```
create_task title="Refactor X transaction boundaries" stack=ideas tags=draft,tech-debt
```

**B. Claude spots a real bug → `triage` slot.** Definitely broken (guaranteed crash, wrong calculation, security issue). Capture as the relevant work stack with `tags=triage,bug`, `status=todo`, best-guess priority. Return to the original task without investigating.
```
create_task title="<concrete-symptom>" stack=<workstack> tags=triage,bug priority=high
```
The asymmetry vs `ideas`: ideas-stack items are guesses; `tag=triage` items are claims. `list_tasks tag=triage` is the bug-finding queue.

**Don't investigate.** Capture, return, move on.

**A2. Sprint-scale idea, scoped through Q&A → `sprint-candidate` slot.** When the user surfaces a chunky theme (typically a major new capability, a new domain, a runtime-wide change, a UX overhaul) and you've walked them through enough scoping questions to lock down the shape — and the answer is "this is too big for a single task" — capture as `stack=ideas, tags=draft,sprint-candidate,<domain>`. The body must include:
- **Goal** (1–2 sentences)
- **Scope decisions** (the bullets the user committed to during scoping — these are *decided*, not open)
- **References** (file:line citations from prior art / reference codebases — what makes the eventual sprint cheap to plan)
- **Integration touchpoints** (directories the sprint will touch)
- **Open questions for the sprint planner** (the things that weren't worth deciding without code in hand)

```
create_task title="Sprint candidate: <theme>" stack=ideas tags=draft,sprint-candidate,<domain>
```

Promotion: when ready to plan, run `/sprint-open` against the candidate. The sprint file inherits the scope artefact; the candidate task stays in `ideas` until the sprint closes.

**Priority guess heuristic:**
- `critical` — data loss, security, production outage, can't-ship blocker
- `high` — guaranteed crash on realistic input, wrong output that could mislead a user
- `medium` — incorrect edge-case behaviour, defensive code missing
- `low` — cosmetic, theoretical, only-in-tests

If unsure between two priorities, pick the higher one. Triage will downgrade; missed bugs don't get re-noticed.

**C. User reports a bug → `todo` slot, investigate immediately.** Already triaged by the act of being reported. Lands directly in its work stack at `priority=high`, `tag=bug`, no `triage` tag, then investigation begins immediately.

### After a debugging session

Append findings with `update_task <id> note="..."`. Notes timestamp into frontmatter and become a paper trail.

### Triage (weekly)

Walk three queues in order:

1. **`list_tasks tag=triage`** — Claude-spotted bugs awaiting prioritisation. Set priority + acceptance criteria, drop the `triage` tag → it's now `todo`. Or escalate (`critical` + `in_progress`). Or downgrade (move to `ideas` stack with `draft` tag) if not worth doing.
2. **`list_tasks stack=ideas`** — refine the real ones (move stack, drop `draft`); park maybes as `on_hold`; defer far-future as `tag=deferred`; kill the rest as `cancelled`.
3. **`list_tasks status=on_hold`** — revive or kill.

Anything in the work stacks without `draft` or `triage` tags should be something you'd actually pick up if you had a free hour.

### Planning (per build phase)

Pull from the work stacks, **move into the `ready` stack** with `tag=ready` and the phase label. Cap the queue — if it's longer than you can plausibly ship, you're lying to yourself.
```
update_task <id> stack=ready tags=ready,phase:next
```

### Picking up work

```
list_tasks stack=ready
```

Pick the top item, **move it back into its work stack** (so the `ready` stack stays a clean queue), drop the `ready` tag, set `status=in_progress`.

```
update_task <id> stack=<workstack> status=in_progress tags=<domain>,<type>
```

Don't pick from work stacks directly — if it wasn't in `ready`, the planning gate didn't happen.

### Opening a PR

`update_task <id> status=in_review note="<PR url>"`.

### After merge

`complete_task <id>`. Done. Archive stage is hidden from default `list_tasks`.

## Journal entries (post-handoff append)

The `journal` stack is a session-record surface — one entry per `/handoff` invocation, paired with `tag=handoff`. Journal entries are *records*, not work; they don't promote, they don't get planned against, they don't enter `ready`.

### Why a dedicated stack

Mixing session records into `ideas` rots the triage queue (you have to exclude `tag=handoff` from `/triage` walks) and obscures the journal as a discoverable surface. The journal is its own thing: a running log of where each session left off.

### Post-handoff append behaviour

After `/handoff` has filed a journal entry in the current session, the rest of the session operates in **journal-append mode**:

- Observations that aren't actionable enough for `/log` and don't warrant a new task → `add_comment` against the latest journal entry (`list_tasks stack=journal limit=1 sort=created reverse=true`). The agent tells the user it captured the observation in a brief one-liner, so the user knows the loose thread didn't go in the bin.
- Each comment must be self-contained: short context + the observation + why it might matter. The user may resume from a fresh session and need to understand the comment in isolation.

**Examples of append targets** (post-handoff, in-session):

- "Noticed `X` while doing `Y` — worth investigating later but not now."
- "The `Z` workflow surfaced a wart: <description>. Could be its own task, didn't seem urgent enough."
- Anything that would normally be a verbal aside and would otherwise be lost on window close.

**Not append targets** (still handle normally):

- Actionable findings the user needs to decide on right now → surface verbally, ask.
- Real bugs → still use `/log` to file properly with `triage,bug`.
- Status updates on ongoing work → those go on the relevant in_progress task, not the journal.

### Cross-session behaviour

The append rule applies **within the session that ran `/handoff`**. A new session resuming a project sees the previous journal entry via `/board` ("Last handoff: <short_id> — <title> (<N>h ago)") and may reference it for context, but starts fresh — observations during the resume session route to `/log` or verbal-surface until that session's own `/handoff` creates a new journal entry.

## Anti-patterns

- **Trying to set `status=ready` / `status=idea` / `status=triage`.** Stackydo will reject them. Use stack + tag.
- **Picking from a work stack directly without going through `ready`.** Planning gate stops happening, `ready` becomes meaningless.
- **Letting `in_progress` rot.** If untouched for a week, it's `on_hold` or `blocked`.
- **Tags as workstreams.** That's what stacks are for. Tags answer "what kind of work," not "which area."
- **Long-lived project context in tasks.** That belongs in `CLAUDE.md`, `docs/architecture/`, or `docs/sprints/`. Tasks are for *work*, not *knowledge*.
- **Inventing custom stacks ad hoc.** Pick the set once and use a tag to slice cross-cutting concerns. Adding a new stack should feel like a deliberate decision, not a casual one.

## Commands cheat-sheet

```bash
# Situational awareness
get_stats
list_tasks status=in_progress
list_tasks stack=ready
list_tasks tag=triage
list_tasks overdue=true

# Capture (agent-spotted thought)
create_task title="..." stack=ideas tags=draft,tech-debt

# Capture (agent-spotted real bug — defer investigation)
create_task title="..." stack=<workstack> tags=triage,bug priority=high

# Capture (user-reported bug)
create_task title="..." stack=<workstack> tags=bug priority=high

# Refine an idea into the real backlog (move stack, drop draft tag)
update_task <id> stack=<workstack> tags=<domain>,<type>

# Commit to next build phase
update_task <id> stack=ready tags=ready,phase:next

# Start work (pull out of ready stack into work stack)
update_task <id> stack=<workstack> status=in_progress tags=<domain>,<type>

# Open PR
update_task <id> status=in_review note="<PR url>"

# Done
complete_task <id>

# Search before creating to avoid duplicates
search_tasks "<keywords>"
```

Prefix-matching works on task short_ids — the short ID is enough.

## Workspace configuration

`stackydo.json` at the repo root is intentionally minimal — no custom statuses, no aliases. The lifecycle slots above are encoded in **convention**, not in stackydo's workflow config.

If the project later needs the `idea` / `triage` / `ready` statuses to be enforceable (rather than convention), extend `stackydo.json` with `workflow.statuses` and migrate this skill to a status-based lifecycle. Until then: **stack + tag is the source of truth for lifecycle slot**, status is the source of truth for active state.

`.stackydo/manifest.json` is internal state owned by stackydo. Don't edit it directly.
