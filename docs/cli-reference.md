# CLI Reference

## Commands

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
| `bulk-status` | Move all tasks matching a status (and optional filters) to a new status |
| `shuffle` | Randomise the order of tasks in a stack+status group |
| `draw` | Draw the top task from one group and move it to another |
| `list-workspaces` | Discover all stackydo workspaces on the system |
| `migrate` | Move or copy tasks between workspaces |
| `import` | Import tasks from stdin (JSON or YAML) |
| `mcp-setup` | Register `stackydo-mcp` with Claude Code via `claude mcp add` |
| `agent-setup` | Generate an AI agent playbook in your project's CLAUDE.md |

Run any command with `--help` for full usage details.

## Stacks

A task belongs to one **stack**: a named group like "work", "personal", or "sprint-12". Think of stacks as separate piles of sticky notes on your desk.

```bash
stackydo create --title "Deploy v2" --stack work
stackydo list --stack work
stackydo list --stack personal --status todo
stackydo stacks                  # see all stacks with counts
```

Tasks without a stack are unstacked and won't appear in stack-filtered results.

## Workflows

Workflows define the statuses available to tasks and how they map to lifecycle stages (`backlog`, `active`, `archive`). Stackydo ships with two built-in workflows — `kanban` (default) and `deck` — and you can define custom statuses and per-stack assignments in [`stackydo.json`](config.md).

For the complete guide — built-in workflows, custom workflows, per-stack assignment, index ordering, shuffle/draw, bulk-status, and cross-stack moves — see [**Workflows**](workflows.md).

Quick reference for the built-in `kanban` workflow:

| Stage     | Statuses                                  |
|-----------|-------------------------------------------|
| `backlog` | `todo`, `on_hold`                         |
| `active`  | `in_progress`, `blocked`, `in_review`     |
| `archive` | `done`, `cancelled`                       |
