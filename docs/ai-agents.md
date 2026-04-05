# AI Agents

Stackydo is designed to work with AI coding agents. The MCP server gives agents full read/write access to your task store, and the `agent-setup` command generates project-specific playbooks.

## Setup

```bash
# Register the MCP server with Claude Code
stackydo mcp-setup

# Generate an agent playbook for your project
stackydo agent-setup
```

`agent-setup` writes instructions into your project's CLAUDE.md so agents know how to create, update, and complete tasks as they work. If MCP is configured, the playbook uses MCP tools; otherwise it falls back to CLI commands.

For MCP configuration with other tools (Cursor, VS Code, Codex, Windsurf), see the [MCP Setup Guide](mcp-setup.md).

## Available MCP tools

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

## How agents use it

A typical agent workflow:

```
1. Check existing tasks          →  list_tasks / search_tasks
2. Create a task for the work    →  create_task
3. Log progress with notes       →  update_task --note "..."
4. Mark done when finished       →  complete_task
```

Agents can also use stackydo to:

- Break complex work into subtasks with dependencies
- Triage and prioritise a backlog
- Summarise progress across stacks with `get_stats`
- Move tasks between workspaces when work shifts between projects
