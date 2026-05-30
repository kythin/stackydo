---
description: Capture a one-liner bug, idea, or nit into stackydo without breaking the main thread. Files directly via mcp__plugin_stackydo-flow_stackydo__create_task with the right classification — no investigation, no menu, just a sticky note that survives. Use when threading with another agent and don't want to context-switch.
argument-hint: <one-liner: short description of the bug, idea, or nit>
---

The argument is the user's one-liner. If empty, refuse with "Pass a one-liner: /log <short description>".

## Pre-flight: workspace bootstrap (first stackydo use this session)

Before filing, check whether the project has a `stackydo.json`. If not (and the user is in a project directory, not their home dir), invoke the `stackydo-bootstrap` skill once. Skip if already invoked or declined in this session. `/log` is fire-and-forget — if the user declines the bootstrap, proceed with the home-workspace fallback rather than re-asking.

## Classify (single pass, no agents, no investigation)

Look at the one-liner and the active conversation context to pick a slot. Use these rules in order:

1. **Starts with `nit:`** (case-insensitive) → it's a nit. Strip the prefix from the title. Save with: work stack (guess from context — see Stack below), `tags=nit`, `status=todo`, `priority=low`. No `triage` tag, no `bug` tag. Nits are bundled into a later UX/UI pass, not investigated now.
2. **Bug-like phrasing** — words like *broken*, *crash*, *doesn't work*, *wrong*, *failing*, *errors*, *NaN*, *undefined*, *throws*, *500*, *won't*, *can't*. → work stack, `tags=triage,bug`, priority guess by phrasing severity:
   - `critical` — data loss, security, can't ship
   - `high` — guaranteed crash on realistic input (default for bug-like)
   - `medium` — edge case, defensive code missing
   - `low` — cosmetic only
3. **Idea-like phrasing** — words like *would be nice*, *consider*, *what if*, *should we*, *we could*, *maybe*, *future*, *eventually*, *idea*. → `stack=ideas`, `tags=draft`, `priority=low` (judgement-free capture).
4. **Default** (anything else, including neutral observations) → treat as idea: `stack=ideas`, `tags=draft`.

If the user wraps the message in quotes or includes a file path or short_id reference, keep it in the title — it's context.

## Stack guess (for bug + nit)

Guess work stack from keywords in the one-liner (look at both the argument and recent conversation context if it's tight).

**Path beats word.** If the one-liner contains an explicit file path, the path's top-level subproject is the strongest signal. Use the repo's `stackydo.json` `stacks` list (or check `mcp__plugin_stackydo-flow_stackydo__get_stacks`) to know what work stacks exist; map the path's top folder to the closest match.

**No path? Word match against the project's stack vocabulary.** This requires project-specific tuning — what keywords map to which stack depends on what the project's stacks *are*. As an authoring shape for projects that want this, define the mapping in `.claude/log-stack-keywords.json` or document it in `CLAUDE.md`; the command reads that and applies first-match wins (precedence by list order). If no mapping exists, default to the most-used work stack (look at `get_stats`), or to `ideas` for ambiguous bugs and let triage route them.

If still ambiguous after both passes, default to the most-used work stack. Don't ask the user — `/log` is fire-and-forget.

## Search for duplicates

Before filing, `mcp__plugin_stackydo-flow_stackydo__search_tasks` with 2-3 key nouns from the title. If a non-terminal task (`todo` / `in_progress` / `in_review` / `blocked` / `on_hold`) looks like the same thing:

- Add a comment to that task with the new one-liner via `mcp__plugin_stackydo-flow_stackydo__add_comment` instead of creating a duplicate.
- Report "Appended to <short_id> — `<existing title>`" and stop.

If only `done`/`cancelled` matches exist, file a new task — the issue is back.

## File

`mcp__plugin_stackydo-flow_stackydo__create_task` with the title, stack, tags, priority, and status as classified above. Title is the user's one-liner (trimmed, sentence case, no trailing period).

## Output

One line. Three forms:

- New task filed: `` Filed <short_id> — `<title>` (stack=<stack>, tags=<tags>, priority=<priority>). `` (use single backticks, not triple)
- Comment appended to duplicate: `` Appended to <short_id> — `<existing title>`. ``
- Refused (empty arg): `Pass a one-liner: /log <short description>`

Then stop. Don't summarise what you classified, don't ask "want me to also …?", don't suggest next steps. The user is in another thread.

## Constraints

- **No investigation.** `/log` doesn't read code, doesn't reproduce, doesn't open files. The whole point is to be cheap.
- **Don't spawn a sub-agent.** This command files directly.
- **One stackydo write max** (`create_task` OR `add_comment`, never both for the same call).
- Lifecycle slot via stack + tag, never via status (see the `delivery-lifecycle` skill).
- `nit:` prefix → `tag=nit`, stays `todo`, not investigated, bundled into a later UX/UI pass.
