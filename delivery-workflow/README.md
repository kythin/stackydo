# Delivery Workflow Plugin

An opinionated delivery loop for Claude Code, distilled from a real working monorepo. Covers task lifecycle, weekly triage, sprint open/close ritual with frozen plans, and orchestration agents that delegate implementation rather than doing it themselves.

## Dependencies (read this first)

This plugin uses [stackydo](https://github.com/kythin/stackydo) as the task system. Every agent and command here calls `mcp__plugin_delivery-workflow_stackydo__*` tools directly — the dependency is load-bearing, not optional.

Stackydo is wired in **automatically** when you install the plugin. The plugin ships a `.mcp.json` that runs it via `npx --package @kythin/stackydo stackydo-mcp` — no separate install, no manual MCP config. The first invocation may take a moment while npx fetches the package; subsequent calls are fast.

Tasks live in `./.stackydo/` at the root of whatever project you run Claude Code in. The first command (or `mcp__plugin_delivery-workflow_stackydo__create_task`) initialises that folder.

Sprint open/close additionally assumes a `docs/sprints/` folder convention (with a `_template.md` and an `archive/sprints/` location). The `/sprint-open` command will set this up on first use, but the structure is opinionated — read `skills/delivery-lifecycle/SKILL.md` to know what you're opting into.

## What's in the box

**2 skills, 3 agents, 10 commands.**

### Skill

| Skill | What it does |
|---|---|
| `delivery-lifecycle` | The full lifecycle reference — idea / triage / todo / ready / in_progress / in_review / done, plus sprint-candidate and deferred slots. Auto-triggers when you ask about task status, lifecycle slots, or "how do I promote this". |
| `stackydo-bootstrap` | Detects when stackydo would fall back to the home-dir workspace, confirms with you, and writes a project-local `stackydo.json` + `.stackydo/`. Invoked automatically by the entry-point commands on first stackydo use per session. |

### Agents

| Agent | When |
|---|---|
| `delivery-orchestrator` | Roadmap, sprints, and stackydo bookkeeping. Delegates implementation to specialist agents — does not write code itself. Default entry point for "what's next", "log a bug", "open a sprint for X". |
| `sprint-planner` | Pre-sprint plan-mode work: turns a theme into a frozen plan artefact at `docs/sprints/plans/<slug>.md` with locked design choices, architecture impact, and implementation slices. |
| `claude-config-auditor` | Reviews your `.claude/` orchestration tooling (agents, commands, settings.json) for drift, broken references, weak triggers, voice inconsistency. Review-only — never edits. |

### Commands

| Command | What it does |
|---|---|
| `/board` | Show the current state — in-flight tasks, ready queue, active sprint progress, recent commits. Session-start ritual in one keystroke. |
| `/next` | Decide the next action via a dispatch decision tree (resume in-progress / close shippable sprint / open queued plan / run triage / pick fresh theme). |
| `/log <one-liner>` | Fire-and-forget capture of a bug, idea, or nit into stackydo without breaking the main thread. No investigation, no menu. |
| `/triage` | Walk the weekly triage ritual — triage-tagged bugs, ideas backlog, on-hold items, sprint-promotion candidates. Surfaces each item one at a time. |
| `/sprint-open [slug]` | Open a new sprint. Enforces one-sprint-at-a-time. With no slug, discovers queued frozen plans. |
| `/sprint-close [slug]` | Close + archive the active sprint. Verifies linked tasks shipped, blocks on missing architecture-doc updates, moves the file to archive, fixes inbound links. |
| `/roadmap` | Roadmap-level planning ritual. Walks sprint-candidates one at a time and sequences them into `docs/roadmap.md` (Next up / Queued / Out of view). Optionally spawns `sprint-planner` for promoted items. Sibling to `/triage` — runs at a higher level. |
| `/handoff` | End-of-session ritual. Walks in_progress tasks for a "where am I" note each, then offers to commit + push the working tree with a secret scan + gitignore sanity check first. Sibling to `/board` — `/board` is the resume side, `/handoff` is the park side. |
| `/align-claude-config` | Run the `claude-config-auditor` agent and surface its report. Review-only. |
| `/stackydo-issue <one-liner>` | File a sanitised GitHub issue against the stackydo repo (private `kythin/stackydo-cli` if you have access, else public `kythin/stackydo`). Strips project specifics, previews before posting, always asks for confirmation. |

## Install

```
/plugin marketplace add kythin/kythin-claude-marketplace
/plugin install delivery-workflow
```

## Conventions this plugin assumes

The lifecycle is encoded via **stack + tag**, not via stackydo status — because the built-in statuses don't include `idea` / `triage` / `ready` and extending stackydo's workflow config would be heavier than the convention. Read `skills/delivery-lifecycle/SKILL.md` for the full slot table.

Other assumed conventions:

- Work stacks per major workstream (`admin`, `web`, `infra`, `docs`, etc. — yours, named however you like).
- An `ideas` stack for raw captures, paired with `tag=draft`.
- A `ready` stack as the next-build-phase queue, paired with `tag=ready`.
- One sprint at a time. `docs/sprints/` for active, `docs/archive/sprints/` for shipped/abandoned, `docs/sprints/plans/` for frozen plans.
- `docs/roadmap.md` as a five-section cross-cutting view (Active, Next up, Recently shipped, Queued, Out of view).

You don't need all of this on day one — the lifecycle slots and `/log` work standalone. The sprint machinery is opt-in (start using it when you have multi-task themes).

## Caveats

This is a heavier plugin than skills-only ones in this marketplace — agents + commands + a skill, plus a hard dependency on stackydo. That's intentional: delivery workflow benefits from the whole loop being aware of itself.
