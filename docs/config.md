# Configuration: `stackydo.json`

Stackydo has one user-facing configuration file: **`stackydo.json`**. Everything you might want to customise — the workspace location, the stack filter, project context, workflows, per-stack workflow assignments — lives here.

The workspace's `manifest.json` is **internal state** managed automatically by stackydo. You should not hand-edit it.

## Where stackydo looks for it

When you run a stackydo command, it searches in this order:

1. **Walk up from the current working directory** looking for a `stackydo.json` file. The first one found wins.
2. **Fall back to `~/.stackydo.json`** (your home directory).
3. **No file** — built-in defaults are used.

This means you can put a project-specific `stackydo.json` next to a project's `.git`, and a global `~/.stackydo.json` for everywhere else.

You can see which file resolved with:

```bash
stackydo context
```

## Schema and editor support

The full JSON Schema is published at [`schemas/stackydo.schema.json`](../schemas/stackydo.schema.json). To get autocompletion and validation in editors that support JSON Schema (VS Code, JetBrains, Neovim with `coc-json`, etc.), include the `$schema` field at the top of your file:

```json
{
  "$schema": "https://raw.githubusercontent.com/kythin/stackydo/main/schemas/stackydo.schema.json",
  "dir": ".stackydo"
}
```

You can also print the schema locally:

```bash
stackydo schema config
```

## Fields

All fields are optional. A bare `{}` is a valid `stackydo.json`.

### `dir`

Path to the task store directory, relative to the `stackydo.json` file. Overrides the `STACKYDO_DIR` environment variable.

```json
{ "dir": ".stackydo" }
```

The directory is created on first task save if it does not exist.

### `stack_filter`

Glob pattern (`*` wildcards) limiting which stacks appear in `list`, `stats`, `stacks`, and `search` output. When absent, all stacks are shown.

```json
{ "stack_filter": "project-myapp_*" }
```

This does **not** restrict write access — for true isolation, use separate workspaces.

### `context`

Project context metadata captured into each task's `context` field at creation time. Useful for giving AI agents background about the project a task belongs to.

```json
{
  "context": {
    "project": "My App",
    "repo": "https://github.com/org/my-app",
    "description": "Customer-facing web app. Tasks here usually relate to bug fixes or feature work in the main repo."
  }
}
```

| Field         | Type   | Description                                                    |
|---------------|--------|----------------------------------------------------------------|
| `project`     | string | Human-readable project name                                    |
| `repo`        | string | Repository URL or identifier                                   |
| `description` | string | Free-text notes captured into task context at creation time    |

### `workflow`

Workspace default workflow. Replaces the built-in `kanban` workflow as the default for any stack that is not assigned a workflow in `stack_workflows`. Use this to define custom statuses or aliases.

See [Workflows](workflows.md) for the complete reference. A short example:

```json
{
  "workflow": {
    "statuses": {
      "icebox": "backlog",
      "next": "backlog",
      "doing": "active",
      "review": "active",
      "shipped": "archive",
      "dropped": "archive"
    },
    "aliases": {
      "wip": "doing",
      "done": "shipped"
    }
  }
}
```

When omitted, the built-in `kanban` workflow is used.

### `stack_workflows`

Per-stack workflow assignment. Maps stack name to the name of a built-in workflow. Stacks not listed here use the workspace default (`workflow` if set, otherwise `kanban`).

```json
{
  "stack_workflows": {
    "ideas": "deck",
    "playtest": "deck"
  }
}
```

Currently supported workflow names: **`kanban`** and **`deck`**. Matching is case-insensitive. See [Workflows](workflows.md) for what each workflow defines.

## Complete examples

### Minimal project config

```json
{
  "$schema": "https://raw.githubusercontent.com/kythin/stackydo/main/schemas/stackydo.schema.json",
  "dir": ".stackydo"
}
```

### Project with context and one custom-workflow stack

```json
{
  "$schema": "https://raw.githubusercontent.com/kythin/stackydo/main/schemas/stackydo.schema.json",
  "dir": ".stackydo",
  "context": {
    "project": "stackydo-cli",
    "repo": "github.com/kythin/stackydo"
  },
  "stack_workflows": {
    "ideas": "deck"
  }
}
```

### Workspace with a fully custom default workflow

```json
{
  "$schema": "https://raw.githubusercontent.com/kythin/stackydo/main/schemas/stackydo.schema.json",
  "workflow": {
    "statuses": {
      "icebox": "backlog",
      "next": "backlog",
      "doing": "active",
      "review": "active",
      "shipped": "archive"
    },
    "aliases": {
      "wip": "doing"
    }
  }
}
```

## What lives in `manifest.json` instead

`manifest.json` is the workspace's internal state file. It tracks:

- Schema version
- The set of tag and stack names that have been used (auto-registered as you create tasks)
- The next short-ID counter (`SD1`, `SD2`, …)

That's it. If you find yourself wanting to edit `manifest.json`, you almost certainly want to edit `stackydo.json` instead.
