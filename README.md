# stackydo

**One person's entire workload, in one place.**

Stackydo is a context-aware CLI task manager designed for individual engineers, leads, and makers who juggle work across many projects, teams, and responsibilities. Tasks are plain markdown files with YAML frontmatter — no database, no server, no vendor lock-in.

The core idea: your work doesn't live in one project. You might be debugging a production incident, reviewing a teammate's PR, prepping a conference talk, and planning next sprint — all in the same afternoon. Stackydo uses **stacks** to separate these workstreams while keeping everything in a single, searchable task store.

## Why This Exists

Most task managers are built for teams or for single projects. Stackydo is built for **you** — the individual who needs to:

- Track work across multiple projects, random ideas, ad-hoc tasks, team duties, and personal goals simultaneously
- Create tasks from wherever you are — terminal, editor, scripts, or AI agents — so fast it becomes muscle memory to offload every stray thought
- Search across everything at once ("what was that security thing last week?")
- Let AI tools triage, summarize, and report on your workload via the built-in MCP server
- Own your data as plain files you can grep, version, and back up

## Features

- **Headless CLI** — create, list, search, update, complete, delete from scripts and pipelines
- **MCP server** — gives AI assistants (Claude Desktop, Claude Code, etc.) full access to your task store
- **Stacks** — organize tasks into named workstreams (e.g. "work", "personal", "sprint-12")
- **Short IDs** — human-friendly IDs like SD1, SD2 alongside ULIDs; all commands accept either
- **Automatic context capture** — records git branch/commit, working directory, and project context on task creation
- **Task graph** — subtasks, dependencies (blocked-by, blocks, related-to)
- **AI-friendly storage** — plain markdown+YAML files that any LLM or script can read and write
- **`.stackydo-context` files** — define project-level context that gets attached to new tasks automatically
- **Session chaining** — tracks the last task ID created per shell session via `$STACKYDO_LAST_ID`
- **Configurable storage** — set `dir` in `.stackydo-context` for per-project workspaces, or `$STACKYDO_DIR` for per-session overrides (defaults to `~/.stackydo/`)
- **Multi-workspace** — discover, list, and migrate tasks across workspaces
- **Doctor** — diagnose and auto-fix workspace issues (missing short IDs, orphan refs, corrupt files)

## Install

All install methods provide two binaries: `stackydo` (CLI) and `stackydo-mcp` (MCP server).

### macOS

**Homebrew** (recommended):

```bash
brew tap kythin/homebrew-tap && brew install stackydo
```

**Shell installer** (Apple Silicon and Intel):

```bash
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/kythin/stackydo/releases/latest/download/stackydo-installer.sh | sh
```

Binaries are installed to `~/.cargo/bin/`. Make sure it's on your `PATH` (the installer will prompt you if not).

### Linux

**Homebrew**:

```bash
brew tap kythin/homebrew-tap && brew install stackydo
```

**Shell installer** (x86_64 and aarch64):

```bash
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/kythin/stackydo/releases/latest/download/stackydo-installer.sh | sh
```

**Manual download** — grab the archive for your architecture from the [latest release](https://github.com/kythin/stackydo/releases/latest):

```bash
# x86_64
tar xf stackydo-x86_64-unknown-linux-gnu.tar.xz
sudo mv stackydo stackydo-mcp /usr/local/bin/

# aarch64
tar xf stackydo-aarch64-unknown-linux-gnu.tar.xz
sudo mv stackydo stackydo-mcp /usr/local/bin/
```

### Windows

**PowerShell installer**:

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://github.com/kythin/stackydo/releases/latest/download/stackydo-installer.ps1 | iex"
```

**Manual download** — grab `stackydo-x86_64-pc-windows-msvc.tar.xz` from the [latest release](https://github.com/kythin/stackydo/releases/latest), extract, and add the folder to your `PATH`.

### Update

```bash
# Homebrew
brew update && brew upgrade stackydo

# Shell/PowerShell — use the built-in updater:
stackydo-update

# Or just re-run the installer — it will overwrite the existing version.
```

### Verify

```bash
stackydo --version
stackydo doctor
```

## Quick Start

```bash
# Create a task
stackydo create --title "Fix auth bug" --tags "backend,urgent" --priority high --stack work \
  -- The login endpoint returns 500 when the token expires

# List tasks
stackydo list
stackydo list --status todo --sort priority
stackydo list --stack work
stackydo list --overdue

# Show task detail (prefix matching and short IDs work everywhere)
stackydo show SD1
stackydo show 01HQ

# Update a task
stackydo update SD1 --status in_progress --note "Investigating root cause"

# Complete a task
stackydo complete SD1

# Search
stackydo search "auth"

# Add a comment
stackydo comment SD1 "Turns out it was a timezone issue"

# Check workspace health
stackydo doctor
stackydo doctor --fix
```

## CLI Commands

| Command | Description |
|---------|-------------|
| `create` | Create a new task with title, tags, priority, stack, body, due date, dependencies |
| `list` | List/filter tasks by status, stage, tag, priority, stack, due date; sort, group, paginate |
| `show` | Show a task's full details |
| `update` | Update fields, append body text, sed-style substitution, add timestamped notes |
| `complete` | Mark task(s) as done (single or bulk with `--all`) |
| `delete` | Permanently delete task(s) (single or bulk with `--all`) |
| `search` | Search title and body (case-insensitive) with same filters as `list` |
| `comment` | Add a comment to a task |
| `stats` | Summary statistics: totals, overdue count, breakdowns by status/stack/tag |
| `stacks` | All stacks with per-stack task counts and status breakdowns |
| `context` | Preview what context would be captured for a new task |
| `init` | Initialize a new workspace (optionally with `--here` and `--git`) |
| `doctor` | Diagnose and optionally fix workspace issues |
| `shuffle` | Randomise the order of tasks in a stack+status group |
| `draw` | Draw the top task from one group and move it to another |
| `list-workspaces` | Discover all stackydo workspaces on the system |
| `migrate` | Move or copy tasks between workspaces |
| `import` | Import tasks from stdin (JSON or YAML) |
| `mcp-setup` | Register `stackydo-mcp` with Claude Code via `claude mcp add` |
| `agent-setup` | Generate an AI agent playbook in your project's CLAUDE.md |

## Stacks

A task can belong to one **stack** — a named group like "work", "personal", or "sprint-12". Think of stacks as physical piles of tasks rather than flat database categories.

```bash
# Create a task on a stack
stackydo create --title "Deploy v2" --stack work

# Filter tasks by stack
stackydo list --stack work
stackydo list --stack personal --status todo

# See all stacks with counts
stackydo stacks
```

The manifest tracks known stack names. Tasks without a stack are unstacked and won't appear in stack-filtered results.

## Workflows

Workflows define the statuses available to tasks and how they map to lifecycle stages (backlog, active, archive). Each stack can use a different workflow.

### Built-in Workflows

**Kanban** (default) — traditional task board:

| Stage | Statuses |
|-------|----------|
| Backlog | `todo`, `on_hold` |
| Active | `in_progress`, `blocked`, `in_review` |
| Archive | `done`, `cancelled` |

**Deck** — card game metaphor for randomised/prioritised work:

| Stage | Statuses |
|-------|----------|
| Backlog | `deck` |
| Active | `hand`, `table` |
| Archive | `discard` |

### Per-Stack Assignment

Set a stack's workflow in `manifest.json`:
```json
{
  "stack_workflows": {
    "ideas": "deck",
    "work": "kanban"
  }
}
```

Stacks without a workflow inherit the workspace default (kanban).

### Index Ordering

Tasks within the same stack+status group have an `index` field for positional ordering. Indexes are maintained automatically when tasks are created, moved, or deleted. Use `shuffle` to randomise order, and `draw` to pull the top task from one group to another:

```bash
# Randomise the deck
stackydo shuffle --stack ideas --status deck

# Draw the top card into your hand
stackydo draw --source ideas/deck --target ideas/hand
```

## MCP Server

Stackydo includes an MCP (Model Context Protocol) server that gives AI assistants full access to your task store. The `stackydo-mcp` binary communicates over stdio and works with any MCP-compatible client.

### Claude Code (CLI / VS Code / JetBrains)

The quickest setup — one command:

```bash
stackydo mcp-setup
```

This runs `claude mcp add` to register the server. Options:

```bash
stackydo mcp-setup                    # project scope (default)
stackydo mcp-setup --scope user       # available in all projects
```

**Manual alternative** — add to your project's `.mcp.json`:

```json
{
  "mcpServers": {
    "stackydo": {
      "type": "stdio",
      "command": "stackydo-mcp"
    }
  }
}
```

Restart Claude Code after setup.

### Claude Desktop

Edit the config file:

| Platform | Config path |
|----------|-------------|
| macOS | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |
| Linux | `~/.config/Claude/claude_desktop_config.json` |

Add the `stackydo` entry under `mcpServers`:

```json
{
  "mcpServers": {
    "stackydo": {
      "command": "stackydo-mcp"
    }
  }
}
```

To use a non-default storage directory:

```json
{
  "mcpServers": {
    "stackydo": {
      "command": "stackydo-mcp",
      "env": {
        "STACKYDO_DIR": "/path/to/your/tasks"
      }
    }
  }
}
```

Restart Claude Desktop after editing.

### Cursor

Open **Settings > MCP** (or edit `.cursor/mcp.json` in your project root):

```json
{
  "mcpServers": {
    "stackydo": {
      "command": "stackydo-mcp"
    }
  }
}
```

For global access, add to `~/.cursor/mcp.json` instead. Restart Cursor after setup.

### VS Code (Copilot)

Add to your project's `.vscode/mcp.json` (or user settings):

```json
{
  "servers": {
    "stackydo": {
      "type": "stdio",
      "command": "stackydo-mcp"
    }
  }
}
```

Enable MCP support in VS Code settings: set `chat.mcp.enabled` to `true`. Restart VS Code.

### OpenAI Codex CLI

Add to your project's `.codex/mcp.json`:

```json
{
  "mcpServers": {
    "stackydo": {
      "command": "stackydo-mcp"
    }
  }
}
```

### Windsurf

Open **Settings > MCP** (or edit `~/.codeium/windsurf/mcp_config.json`):

```json
{
  "mcpServers": {
    "stackydo": {
      "command": "stackydo-mcp"
    }
  }
}
```

Restart Windsurf after setup.

### Project-Local Workspace

To scope the MCP server to a project-specific task store:

```bash
stackydo init --here --dir .stackydo-workspace
```

This creates a `.stackydo-context` file that tells `stackydo-mcp` to use the local directory. Any MCP client running from the project root will automatically use the local workspace.

### Available MCP Tools

| Tool | Description |
|------|-------------|
| `list_tasks` | List/filter tasks by status, tag, priority, stack, due date; sort and group |
| `get_task` | Get a single task by ID (prefix matching) |
| `create_task` | Create a task with title, priority, tags, stack, body, due date |
| `update_task` | Update fields, append timestamped notes, body editing |
| `complete_task` | Mark a task as done |
| `delete_task` | Permanently delete a task |
| `search_tasks` | Search title and body (case-insensitive) |
| `add_comment` | Add a comment to a task |
| `get_stats` | Summary statistics: totals, overdue count, breakdowns by status/stack/tag |
| `get_stacks` | All stacks with per-stack task counts and status breakdowns |
| `list_workspaces` | Discover all stackydo workspaces on the system |
| `migrate_tasks` | Move or copy tasks between workspaces |
| `doctor` | Diagnose and optionally fix workspace issues |
| `shuffle` | Randomise task order in a stack+status group |
| `draw` | Draw top task from one group and move to another |

The server also exposes a `stackydo://guide` resource with a full agent guide, and prompt templates for triage, planning, and task extraction.

## AI Agent Setup

Stackydo is designed to be used by AI coding agents (Claude Code, Cursor, etc.) as their own task tracker:

```bash
# Generate a CLAUDE.md playbook for your project
stackydo agent-setup

# Register the MCP server with Claude Code
stackydo mcp-setup
```

`agent-setup` writes instructions into your project's CLAUDE.md so agents know how to create, update, and complete tasks as they work.

## Environment Variables

| Variable | Description |
|----------|-------------|
| `STACKYDO_DIR` | Override the task storage directory (highest priority, overrides `.stackydo-context`; default: `~/.stackydo/`) |
| `STACKYDO_LAST_ID` | Set automatically by `stackydo create`; chains tasks in a shell session |

## Task Storage

Each task is a markdown file at `<STACKYDO_DIR>/<ULID>.md`:

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

A manifest at `<STACKYDO_DIR>/manifest.json` tracks tags, stacks, short ID counter, and settings.

## Context Discovery

On task creation, stackydo automatically captures:

1. Current working directory
2. Git branch, remote URL, and HEAD commit (if in a repo)
3. Content from the nearest `.stackydo-context` file (walks up from CWD, falls back to `~/.stackydo-context`)
4. `$STACKYDO_LAST_ID` — the ID of the previous task created in the same shell session

Use `stackydo context` to preview what would be captured without creating a task.

### Workspace Resolution

The `.stackydo-context` file can set the task store location via a `dir` field. This lets a project root define a shared workspace without requiring every user to set an environment variable.

Resolution priority:
1. `$STACKYDO_DIR` env var (highest — per-session override)
2. `dir` field in the nearest `.stackydo-context` (per-project)
3. `~/.stackydo/` (default)

Example `.stackydo-context`:

```yaml
dir: .stackydo-workspace
project: my-app
stack: dev
description: Project-level context captured on new tasks
```

The `dir` path is resolved relative to the config file's location. Use `stackydo context` to see which source resolved the task store.

To set up a project workspace:

```bash
stackydo init --here --dir .stackydo-workspace
```

## Issues & Feedback

Found a bug or have a feature request? [Open an issue](https://github.com/kythin/stackydo/issues).

## License

MIT
