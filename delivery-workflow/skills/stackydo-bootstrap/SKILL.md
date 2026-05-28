---
name: stackydo-bootstrap
description: >
  Bootstrap a project-local stackydo workspace when none exists. Trigger when
  the user says "set up stackydo", "initialise stackydo", "stackydo init",
  "create stackydo.json", "stackydo isn't configured", "configure stackydo
  for this project", "where does stackydo store tasks", or when an entry-point
  command in this plugin (/board, /log, /next, /triage, /sprint-open,
  /sprint-close) detects that stackydo would fall back to the home-directory
  workspace and wants to offer project-local storage. Writes a minimal
  stackydo.json + creates the .stackydo/ workspace directory after confirming
  with the user.
---

# Stackydo Bootstrap

This skill is invoked by the plugin's entry-point commands when they detect stackydo isn't configured for the current project. Its job is to detect the state, ask once, and bootstrap a project-local workspace if the user wants one.

The stackydo MCP server doesn't expose an `init` command — that's CLI-only — so we replicate the essentials inline.

## Detect

Stackydo walks up from the current working directory looking for `stackydo.json`. If it finds one, that file's `dir` field tells it where to store tasks. If it doesn't find any project-level config, it falls back to `~/stackydo.json` (and ultimately to `~/.stackydo` if that's also absent).

For project-local storage, run this check:

```bash
d="$PWD"; while [ "$d" != "/" ] && [ "$d" != "$HOME" ]; do
  if [ -f "$d/stackydo.json" ]; then echo "FOUND:$d/stackydo.json"; exit 0; fi
  d="$(dirname "$d")"
done
echo "NOT_FOUND"
```

**Three outcomes:**

- `FOUND:<path>` → already configured. Do nothing. Tell the calling command to proceed.
- `NOT_FOUND` and the current directory is `$HOME` itself → user is operating in their home dir, not a project. Probably intentional. Do nothing.
- `NOT_FOUND` and the current directory is a real project (anywhere under `$HOME` but not `$HOME` itself) → offer to bootstrap.

## Bootstrap (only when offering)

### 1. Confirm with the user

Use `AskUserQuestion`:

> Question: "Stackydo isn't configured for this project. Tasks would go to your home-dir workspace. Initialise project-local storage here?"
> Options:
> - **Yes — bootstrap here** (creates `stackydo.json` + `.stackydo/` in this directory)
> - **No — use the home workspace** (don't ask again this session; tasks land in `~/.stackydo/`)

If the user says no, set a session-scoped "don't ask again" expectation by simply not re-running the detect step in this session. The calling command should proceed with home-workspace fallback.

### 2. Gather optional context (only if user said yes)

These are nice-to-have, not required. If a value is hard to determine, skip it — the minimal config is fine.

- **`context.repo`** — `git config --get remote.origin.url` (output may be SSH or HTTPS; either is fine).
- **`context.project`** — from `package.json`'s `name` field if present, else the basename of the current directory.
- **`context.description`** — leave unset unless the user volunteers something. Don't ask another question to fill this.

### 3. Write `./stackydo.json`

Minimal shape (the only required field is `dir`):

```json
{
  "$schema": "https://raw.githubusercontent.com/kythin/stackydo/main/schemas/stackydo.schema.json",
  "dir": ".stackydo",
  "context": {
    "project": "<basename or package.json name>",
    "repo": "<git remote, if available>"
  }
}
```

If you couldn't determine `project` or `repo`, omit the `context` block entirely — don't write empty strings.

**Don't add `workflow` or `stack_workflows`** to the default. The plugin's lifecycle (encoded in the `delivery-lifecycle` skill) deliberately uses the built-in `kanban` workflow and expresses idea/triage/ready as stack + tag. A custom workflow would conflict with the agents and commands.

### 4. Create the workspace directory

```bash
mkdir -p .stackydo
```

The first `create_task` call will populate `.stackydo/manifest.json` automatically.

### 5. Gitignore the lock file

Stackydo creates a `.stackydo/manifest.json.lock` file as machine-local concurrency state. It's a zero-byte sentinel — not meant to travel with the repo. Add it to `.gitignore` so future commits don't accidentally pick it up.

```bash
# Create .gitignore if it doesn't exist, otherwise append.
if [ ! -f .gitignore ] || ! grep -qxF '.stackydo/manifest.json.lock' .gitignore; then
  {
    [ -f .gitignore ] && echo ""
    echo "# Stackydo machine-local lock file — not meant to travel"
    echo ".stackydo/manifest.json.lock"
  } >> .gitignore
fi
```

If the lock file is somehow already tracked (e.g. from a pre-existing repo where stackydo ran before the bootstrap), also run `git rm --cached .stackydo/manifest.json.lock 2>/dev/null` to untrack it. Don't error if it wasn't tracked.

**Don't gitignore `.stackydo/` itself** — task files (`.md`) are meant to travel with the codebase. Only the lock file.

### 6. Report

One short line back to the user:

> Bootstrapped stackydo: `./stackydo.json` + `./.stackydo/`. Tasks for this project will live here from now on. Added `.stackydo/manifest.json.lock` to `.gitignore`.

`.stackydo/` itself is *not* gitignored — task files (`.md`) and `manifest.json` travel with the codebase, which is the whole point of project-local storage. Only the lock file is ignored.

## When the user has a home-dir workspace AND a project workspace

If `~/stackydo.json` exists AND we just bootstrapped a project-local one, the project-local config wins because stackydo's discovery walks up from cwd before falling back to `$HOME`. Tasks created from inside the project go to `.stackydo/`; tasks created from anywhere else go to the home workspace. This is intentional — no action needed.

## Anti-patterns

- **Don't ask twice in one session.** If the user said "no, use home", don't re-prompt on the next command. The calling commands should be aware (see their pre-flight comments).
- **Don't pre-create `.stackydo/manifest.json`.** Stackydo owns that file. Touching it is a recipe for "manifest corrupt" errors.
- **Don't write a non-minimal config by default.** Stick to `$schema` + `dir` + optional `context`. Anything more is a user decision, not a bootstrap default.
- **Don't run `stackydo init` via npx.** The MCP doesn't have init and the CLI's init does more than we need (batch wrapper, gitignore edits we don't want by default). Replicate the essentials inline as above.
- **Don't bootstrap silently.** Always confirm via AskUserQuestion. Some users intentionally use the home-dir workspace for everything.
