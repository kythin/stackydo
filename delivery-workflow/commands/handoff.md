---
description: End-of-session ritual. Captures "where I am" notes against each in_progress task, files a canonical journal entry for the session, then offers to commit + push the working tree (with a secret scan + gitignore sanity check first). After running, post-handoff observations in the same session get appended to the journal entry as comments so nothing surfaced late is lost. Sibling to /board: /board is the resume side, /handoff is the park side.
---

The user is wrapping up a session and wants the next session (possibly on a different machine) to start mid-thought, not mid-fog. Walk in_progress tasks, capture per-task notes, file a journal entry that captures the session as a whole, then optionally commit + push. Once the journal entry is filed, the rest of the session uses it as a running log — any further observation gets appended there.

## Pre-flight: workspace bootstrap (first stackydo use this session)

Before anything else, check whether the project has a `stackydo.json`. If not (and the user is in a project directory, not their home dir), invoke the `stackydo-bootstrap` skill. Skip if already invoked or declined in this session.

## Step 1: List what's in flight

```
mcp__plugin_delivery-workflow_stackydo__list_tasks status=in_progress
mcp__plugin_delivery-workflow_stackydo__list_tasks status=blocked
```

- **One or more tasks in flight** → walk them in Step 2.
- **Nothing in flight** → skip Step 2 entirely and go straight to Step 3. The session journal entry in Step 6 captures the session even when there's no per-task walk.

## Step 2: Walk each in-flight task, capture a handoff comment

For each task, surface:

- short_id, title, stack
- Last comment (if any) — so the user can see what they wrote last session
- A short blurb of the most recent git activity that touched related files (use `git log --oneline -5` if you can't infer the file scope cheaply; don't over-fetch)

Then prompt for the handoff note via `AskUserQuestion`:

> Question: "Handoff note for `<short_id> — <title>`. Where are you, what's next, anything blocking?"
> Options:
> - **Add note** — opens freeform input for the handoff text
> - **Skip** — leave this task untouched
> - **Mark blocked** — for tasks that aren't actually in flight any more, capture the blocker reason in the note AND set `status=blocked` in the same call

If the user adds a note, write it via `mcp__plugin_delivery-workflow_stackydo__add_comment` against that task. Prefix the comment with `Handoff <YYYY-MM-DD HH:MM>:` so the paper trail is scannable.

If the user said "Mark blocked", do both calls: add the comment AND `update_task <id> status=blocked`. Don't bury the status change inside a single tool call where it can fail silently.

Walk tasks one at a time — don't batch. If the in_progress queue is long (>5), say so up front and offer to truncate after the top 5 (most recently updated).

## Step 3: Working-tree state

Run:

```bash
git status --porcelain
git diff --stat
git diff --cached --stat
git log --oneline @{u}..HEAD 2>/dev/null || echo "(no upstream tracking)"
```

Summarise to the user:

- Modified / untracked files (count + a sampling)
- Staged but uncommitted changes (count)
- Unpushed local commits (count, or "(no upstream)" if no tracking branch)

If everything is clean, say so and skip to Step 6 (the journal entry still gets written — that's the whole point).

## Step 4: Sensitivity check (always runs before commit/push)

Before offering commit/push, scan staged + unstaged changes for things that shouldn't travel. Three categories — surface findings together if any hit:

### 4a. Secret-shaped content

```bash
git diff -p HEAD
git diff -p --cached
```

Grep both outputs for the patterns below (case-insensitive). Don't `cat` whole diffs into the response — only quote the matching line context (file + line + ~50 chars).

- Long base64-ish strings (≥40 chars of `[A-Za-z0-9+/=]` not in a known-noise context like a hash)
- `ghp_`, `gho_`, `ghu_`, `ghs_`, `ghr_`, `sk_`, `sk-`, `pk-`, `api[_-]?key`, `secret`, `password`, `token`, `bearer`
- `-----BEGIN (RSA|DSA|EC|OPENSSH|PGP|ENCRYPTED) PRIVATE KEY-----`
- `AKIA[0-9A-Z]{16}` (AWS access key id)
- `xox[baprs]-` (Slack tokens)
- Anything matching `[\w.-]+@[\w.-]+\.\w+` only if it's in a file that shouldn't have emails (skip if it's a docs/contact file)

### 4b. Files that should be gitignored but aren't

Look at the staged + untracked list for high-risk names that aren't in `.gitignore`:

- `.env`, `.env.*` (unless explicitly `.env.example` / `.env.sample`)
- `*.pem`, `*.key`, `id_rsa`, `id_dsa`, `id_ed25519` (and `.pub` only if paired with private)
- `*service-account*.json`, `*credentials*.json`, `*gcp-key*.json`
- `*.sqlite`, `*.db`, `*.dump`, `*.bak`

For any hit:
1. Surface the file.
2. Offer to add it to `.gitignore` AND `git rm --cached` it (if already staged).
3. Re-run the sensitivity check before proceeding.

### 4c. Foreign workspace data

If `.stackydo/` is staged but `stackydo.json` is NOT staged, that's almost certainly a mistake — staging tasks without the workspace config means the next clone can't read them. Flag and ask before committing.

After the sensitivity pass:

- **Nothing flagged** → "Sensitivity check passed."
- **Findings** → list them. `AskUserQuestion`: **Address now** / **Acknowledge and skip** / **Cancel handoff**. Default towards "Address now" — never silently proceed past secret-shaped content.

## Step 5: Commit and push offer

After the sensitivity check is clean (or the user explicitly acknowledged), `AskUserQuestion`:

> Question: "Working tree state to handle?"
> Options (only show options that are relevant given the working-tree summary):
> - **Commit + push** — stage everything sensible, commit with a generated message, push.
> - **Commit only** — same but no push. Local checkpoint.
> - **Stash** — save uncommitted work as a stash with a "handoff" message; leave commits as-is.
> - **Leave as-is** — no working-tree action.

If **Commit + push** or **Commit only**:
1. Stage what's appropriate. **Don't blanket `git add -A`** — stage each modified-or-untracked file individually after a final sanity glance. If anything looks risky despite passing the secret scan, default to skipping it.
2. Generate a commit message. Default shape: `wip: handoff <YYYY-MM-DD>` plus a short body listing the touched stackydo task short_ids. If there's an active sprint, mention the slug. Show the message before writing — `AskUserQuestion`: **Use as-is** / **Edit** / **Cancel**.
3. Commit with the standard Co-Authored-By trailer.
4. If **Commit + push**: confirm push target. If no upstream, surface "(no tracking branch — push would be `git push -u origin <branch>`)" and confirm. Then push. Don't push to `main` / `master` without an extra "really?" confirmation.

If **Stash**:
- `git stash push -u -m "handoff <YYYY-MM-DD HH:MM>"`. Tell the user the stash ref so they can `stash pop` next session.

If **Leave as-is**:
- No-op. Just acknowledge.

## Step 6: File the session journal entry (ALWAYS — this is the canonical session record)

Now that you know everything that happened (tasks walked, working tree handled, commits made), file ONE journal entry summarising the session:

```
mcp__plugin_delivery-workflow_stackydo__create_task
  stack=journal
  tags=handoff
  status=todo
  title="Handoff YYYY-MM-DD HH:MM — <one-line session theme>"
  body=<see structure below>
```

### Title

`Handoff <YYYY-MM-DD HH:MM> — <theme>`. The theme is a short summary inferred from the work: commit subjects, dominant task short_ids, sprint slug if active. Examples:

- `Handoff 2026-05-27 14:30 — delivery-workflow v0.5.0 (journal stack)`
- `Handoff 2026-05-27 — auth-revamp sprint, SD42 in review`
- `Handoff 2026-05-27 — exploratory, no commits`

Don't include the seconds. Keep the theme under ~60 characters.

### Body

Sections, in order:

- **Session summary** — 1–3 sentences on what happened.
- **Tasks touched** — short_ids that moved (created / status-changed / commented on this session) with one-liners. Include the in-flight tasks you just walked.
- **Commits** — hashes + subjects landed this session. If none, say `(none — exploration only)`.
- **Sprint state** — if there's an active sprint, the slug + brief status. Omit if no sprint.
- **Unactioned observations** — things noticed in this session that didn't warrant their own task or a `/log` call but might matter later. Bullet list. Don't fabricate — if there are none, omit the section.

Capture the short_id returned by `create_task`. Hold it in mind — for the rest of this session, any further observation the agent makes goes here as a comment (see [Post-handoff append rule](#post-handoff-append-rule) below).

## Step 7: Final summary

One concise block back to the user:

```
Handoff complete.

Journal entry: <short_id> — Handoff <date> — <theme>
  Mention anything else this session and I'll append it to <short_id>.

Notes captured: <N> task(s)
  · <short_id> — <one-line of what they said>

Working tree: <one of: committed + pushed / committed locally / stashed / left as-is / clean>
  · <commit hash and message if applicable>
  · <stash ref if applicable>

Next session entry point: run /board.
```

Then stop. No "want me to also…?" — the session is wrapping up by design.

## Post-handoff append rule

After Step 6 has filed the journal entry, the rest of the session operates in **journal-append mode**:

- Any observation the agent makes that isn't actionable enough for `/log` and doesn't warrant a new task → `mcp__plugin_delivery-workflow_stackydo__add_comment` against the journal entry, with a short self-contained note (context + observation + why it might matter). The agent tells the user it did so in a brief one-liner, so the user knows the loose thread was captured.
- Genuine bugs → still go through `/log` (which routes to `triage,bug`). Don't put bug reports in the journal.
- Actionable findings the user needs to decide on now → still surface verbally and ask. Journal-append is for things that *would have been verbal asides* and would otherwise be lost on window close.

This rule is also documented in the `delivery-lifecycle` skill and the `delivery-orchestrator` agent so it applies whether `/handoff` was the entry point or just one of many things this session.

## Constraints

- **The journal entry is non-optional.** Step 6 runs even if everything else is empty (no in_progress walk, no working-tree changes). It's the session record.
- **Never auto-commit without showing the message.** Generated messages are starting points, not final.
- **Never push to `main`/`master` without a deliberate second confirmation.**
- **Never bypass the sensitivity check.** If the user declines to address findings, they must explicitly acknowledge.
- **Don't write handoff content to the active sprint file.** Sprint files capture decisions; journal entries capture sessions.
- **Don't run `git add -A` blindly.** Stage per-file after the sensitivity check.
- **One in_progress task at a time during the walk.**
- **Only one journal entry per `/handoff` invocation.** If the command runs twice in a session, that's two entries — they're cheap, but each should be self-contained.
