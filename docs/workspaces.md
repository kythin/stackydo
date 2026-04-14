# Workspaces

Stackydo stores tasks as plain markdown files in a workspace directory. You decide where that directory lives.

## Storage modes

**Central** (default). All tasks in one place under your home directory:

```
~/.stackydo/
  manifest.json
  01HQXYZ1.md
  01HQXYZ2.md
```

**Alongside a project**. Keep tasks near a project without putting them inside the repo:

```
~/Code/my-project/
~/Code/my-project-tasks/       ← stackydo workspace
```

**Inside the project**. Tasks are versioned with the code and shared through git:

```
~/Code/my-project/
  .stackydo-workspace/         ← stackydo workspace
  src/
  ...
```

## Workspace resolution

Stackydo decides which workspace to use based on this priority:

1. **`$STACKYDO_DIR` env var** (highest). Overrides everything. Useful for one-off sessions.
2. **`dir` field in `stackydo.json`**. Resolved relative to the config file's location. Stackydo walks up from the current directory looking for `stackydo.json`, then falls back to `~/.stackydo.json`.
3. **`~/.stackydo/`** (default). Used when nothing else is configured.

## The `stackydo.json` file

Drop a `stackydo.json` in your project root to configure the workspace location and default context for new tasks:

```json
{
  "$schema": "https://raw.githubusercontent.com/kythin/stackydo/main/schemas/stackydo.schema.json",
  "dir": ".stackydo-workspace",
  "context": {
    "project": "my-app",
    "description": "Context captured on new tasks in this project"
  }
}
```

You can also define workflows and per-stack workflow assignments here. See [`config.md`](config.md) for the complete reference and [`workflows.md`](workflows.md) for the workflow customisation guide.

Stackydo walks up from the current directory looking for `stackydo.json`, falling back to `~/.stackydo.json` for global defaults.

Use `stackydo context` to see which config file resolved and what context would be captured.

### Setting up a project workspace

```bash
stackydo init --here --dir .stackydo-workspace
```

This creates a `stackydo.json` file and initialises the workspace directory.

## Multi-workspace

You can have as many workspaces as you like. Stackydo can discover and work across them:

```bash
stackydo list-workspaces        # find all workspaces on the system
stackydo migrate SD1 --to ~/Code/other-project/.stackydo-workspace
```

## Context capture

On task creation, stackydo automatically records:

- Current working directory
- Git branch, remote URL, and HEAD commit (if in a repo)
- Content from the nearest `.stackydo-context` file
- `$STACKYDO_LAST_ID`: the ID of the previous task created in the same shell session

## Environment variables

| Variable | Description |
|----------|-------------|
| `STACKYDO_DIR` | Override the task storage directory (highest priority) |
| `STACKYDO_LAST_ID` | Set automatically by `stackydo create`; chains tasks in a shell session |

## Task file format

Each task is a markdown file named by its ULID:

```markdown
---
id: 01HQXYZ...
short_id: SD1
title: Fix auth bug
status: todo
priority: high
tags: [backend, urgent]
stack: work
index: 0
created: 2025-02-13T10:30:00Z
modified: 2025-02-13T10:30:00Z
context:
  working_dir: /home/user/project
  git_branch: main
  git_commit: a3f7d92
---

The login endpoint returns 500 when the token expires.
```

The workspace also contains a `manifest.json` that tracks tags, stacks, and the short-ID counter. This file is internal state managed automatically — you should not hand-edit it. All user-facing configuration lives in [`stackydo.json`](config.md).
