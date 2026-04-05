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

Workflows define the statuses available to tasks and how they map to lifecycle stages (backlog, active, archive). Each stack can use a different workflow.

### Kanban (default)

| Stage | Statuses |
|-------|----------|
| Backlog | `todo`, `on_hold` |
| Active | `in_progress`, `blocked`, `in_review` |
| Archive | `done`, `cancelled` |

### Deck

A card game metaphor for randomised or prioritised work.

| Stage | Statuses |
|-------|----------|
| Backlog | `deck` |
| Active | `hand`, `table` |
| Archive | `discard` |

### Per-stack assignment

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

### Index ordering

Tasks within the same stack+status group have an `index` field for positional ordering. Indexes are maintained automatically. Use `shuffle` to randomise order, and `draw` to pull the top task from one group to another:

```bash
stackydo shuffle --stack ideas --status deck
stackydo draw --source ideas/deck --target ideas/hand
```
