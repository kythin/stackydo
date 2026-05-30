---
description: Audit and align the .claude/ orchestration tooling (agents, commands, settings). Delegates to the claude-config-auditor agent for the actual review; reports findings + files follow-up stackydo tasks for Must-fix items. Review-only — does not edit any .claude/ file.
argument-hint: [scope: agents | commands | settings | all]  (default: all)
---

The optional argument scopes the audit. If omitted, audit everything.

## Steps

1. **Inventory.** `ls .claude/agents/ .claude/commands/` and `test -f .claude/settings.json`. Report counts so the user knows the scope:
   - "Auditing N agents, M commands, settings.json (present|absent)."
   - If the user passed a scope arg, restrict accordingly.

2. **Delegate to the auditor.** Spawn the `claude-config-auditor` agent via the `Agent` tool. Hand it:
   - The exact scope (agents-only / commands-only / settings-only / all).
   - The reminder that this is a review-only pass — no edits, no new agents/commands.
   - The expectation that it files stackydo tasks (under the project's docs/tooling stack with `tags=triage,bug, priority=medium`) for any Must-fix findings and returns their short_ids.

3. **Surface the report verbatim** to the user with light framing:
   - Three buckets: Must fix / Should fix / Notes.
   - Headline synthesis sentence.
   - List of stackydo short_ids the auditor filed.

4. **Don't auto-fix.** Even if the report contains tiny obvious fixes, leave them for an explicit follow-up. The auditor's job is to surface; the orchestrator (or you, manually) decides what to fix and when.

## Constraints

- Don't run the auditor on `settings.local.json` or transcripts — those are local scratch and out of scope.
- If `.claude/agents/` or `.claude/commands/` is empty, say so plainly and stop — nothing to audit.
- If the auditor reports no Must-fix items, say so plainly. Don't manufacture findings to justify the invocation.
