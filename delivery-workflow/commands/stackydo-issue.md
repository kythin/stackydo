---
description: File a GitHub issue against the stackydo project (bug, feature request, or expectation gap). Targets kythin/stackydo-cli if the user has access, falls back to the public kythin/stackydo. Sanitises content first — no secrets, no project-specific details — and always shows a preview before posting.
argument-hint: <one-liner: short description of the bug, feature, or expectation gap>
---

The argument is the user's one-liner describing the issue. If empty, refuse with "Pass a one-liner: /stackydo-issue <short description>".

## Step 1: Resolve the target repo

Try the private repo first. The user normally has access to `kythin/stackydo-cli`; the public `kythin/stackydo` is the fallback.

```bash
if gh repo view kythin/stackydo-cli --json name >/dev/null 2>&1; then
  echo "TARGET=kythin/stackydo-cli"
else
  echo "TARGET=kythin/stackydo"
fi
```

State which repo will be used so the user can intervene if it picked wrong.

## Step 2: Gather diagnostic context

Cheap things worth including:

- **Stackydo version**: `npx --package @kythin/stackydo stackydo --version 2>&1 | head -3`. May be slow on a cold npx cache — if it takes more than 5 seconds, skip and note "version: unknown (npx cold)".
- **Plugin identity**: this is the `delivery-workflow` plugin running stackydo via `npx --package @kythin/stackydo stackydo-mcp`. Include that one-liner.
- **OS/shell**: `uname -sr` and `$SHELL`. Don't fingerprint deeper.

Do **not** include:

- The current project's path, name, or git remote.
- The contents of `stackydo.json`, `.stackydo/`, or any task files.
- Environment variables or `STACKYDO_*` values that the user has set.
- Recent stackydo MCP tool outputs from this session (may contain task content).

## Step 3: Sanitise the user's description

Read the user's one-liner (and any expansion they ask you to add). Scrub:

- **Absolute paths** → generalise (`/Users/chris/Documents/Projects/foo` → `<project-root>`).
- **Project names**, repo URLs, customer names → generalise (`my-customer-portal` → `<project>`).
- **Task content or IDs from this session** → strip; the maintainer (the user) doesn't need to see their own tasks.
- **Anything that looks like a token, key, or password** (long base64-ish strings, `sk_`, `ghp_`, etc.) → refuse to include; flag to the user.
- **Stack names that match the project's vocabulary** (`admin`, `webar`, etc.) → keep only if they're load-bearing for the bug; otherwise generalise to `<stack>`.

If the description references a specific stackydo behaviour, keep the actual behaviour description verbatim — that's the value. It's the **surrounding context** (where, what project) that gets stripped.

## Step 4: Build the issue body

Use this shape:

```markdown
## What happened (or: what I expected)

<sanitised user description>

## Repro steps

<numbered steps if applicable, else "Not consistently reproducible — see description.">

## Environment

- stackydo: `<version>`
- Invoked via: `delivery-workflow` Claude Code plugin (`npx --package @kythin/stackydo stackydo-mcp`)
- OS: `<uname output>`

<!-- Filed via /stackydo-issue from the delivery-workflow plugin. -->
```

If the user's one-liner clearly describes a feature request or expectation gap rather than a bug, retitle the first section to **What I expected** or **Suggestion** as appropriate. Don't force a bug-shaped issue when it isn't one.

## Step 5: Preview and confirm

Before any `gh issue create`, show the user:

1. The resolved **title** (sentence case, derived from the one-liner; trim trailing punctuation).
2. The resolved **target repo**.
3. The full **issue body** as it will be posted, verbatim.

Then `AskUserQuestion`:

- **Post** → run `gh issue create --repo <target> --title "<title>" --body-file <tmpfile>`. Report the issue URL.
- **Edit** → ask the user what to change, regenerate, re-preview.
- **Cancel** → discard, no posting.

Never skip the preview, even if the sanitisation pass produced a clean result. Public-facing posts are irreversible enough that the cost of a confirmation tap is well worth it.

## Step 6: Report

One line back to the user:

> Filed: `<issue-url>` (`<target-repo>`)

If the user cancelled, just:

> Cancelled — nothing posted.

## Constraints

- **No silent posting.** The preview gate is mandatory.
- **Never include secrets, even after the sanitisation pass.** If something passed Step 3 but you have any uncertainty, flag it in the preview and let the user redact.
- **One issue per invocation.** Don't fan out into multiple repos or related issues.
- **Don't include labels.** The user (maintainer) triages on their side.
- **Don't link to private project context.** Even if `kythin/stackydo-cli` is private, the maintainer reading the issue should be able to understand it without access to the project that triggered the report.
