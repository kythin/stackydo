# Workflows

Stackydo separates **stages** (a fixed system concept) from **statuses** (user-defined). A workflow is the mapping from your statuses to stages.

This split lets you customise the status names you work with (e.g. `icebox` / `wip` / `shipped`) while letting stackydo keep doing useful things behind the scenes — hiding archived tasks, filtering active work, computing overdue, etc.

## Stages

Every task belongs to exactly one of three stages, derived from its status via the active workflow:

| Stage     | What it means                                                                 |
|-----------|--------------------------------------------------------------------------------|
| `backlog` | Not actively being worked on. The default for newly created tasks.             |
| `active`  | In flight. Eligible for overdue calculation.                                   |
| `archive` | Terminal. Hidden from `list` and `search` by default; surface with `--stage archive` or `--status <archived-status>`. |

Stages are not configurable. They are how stackydo categorises work for filtering, hiding, and reporting.

## Built-in workflows

Stackydo ships with two built-in workflows.

### `kanban` (default)

The traditional kanban flow.

| Stage     | Statuses                                  |
|-----------|-------------------------------------------|
| `backlog` | `todo`, `on_hold`                         |
| `active`  | `in_progress`, `blocked`, `in_review`     |
| `archive` | `done`, `cancelled`                       |

Aliases: `doing` → `in_progress`, `inprogress` → `in_progress`, `canceled` → `cancelled`.

Index sort factors (used by `stackydo reindex` and similar): `due`, `priority`, `created`.

### `deck`

A card-game metaphor: draw from the deck into your hand, play to the table, discard when done. Useful for stacks where order matters more than dates and you want to play through items rather than schedule them.

| Stage     | Statuses           |
|-----------|--------------------|
| `backlog` | `deck`             |
| `active`  | `hand`, `table`    |
| `archive` | `discard`          |

Index sort factors: `created` only.

## Configuring workflows in `stackydo.json`

All workflow configuration lives in [`stackydo.json`](config.md). There are two knobs:

1. **`workflow`** — override the workspace default workflow.
2. **`stack_workflows`** — assign specific stacks to a built-in workflow.

When stackydo needs to validate a status or compute a stage for a task, it resolves the workflow like this:

1. If the task's stack is named in `stack_workflows`, use that built-in workflow.
2. Otherwise, if `workflow` is set in `stackydo.json`, use that.
3. Otherwise, use the built-in `kanban` workflow.

### Per-stack assignment

The most common customisation: pick a built-in workflow for one stack, leave everything else on kanban.

```json
{
  "$schema": "https://raw.githubusercontent.com/kythin/stackydo/main/schemas/stackydo.schema.json",
  "dir": ".stackydo",
  "stack_workflows": {
    "ideas": "deck"
  }
}
```

Now tasks in the `ideas` stack accept only `deck`/`hand`/`table`/`discard`, while every other stack continues to use `todo`/`in_progress`/etc.

Currently supported workflow names: `kanban`, `deck`. Matching is case-insensitive.

### Custom default workflow

Define a brand new status set as the workspace default by setting the `workflow` field. Each status must map to one of the three fixed stages.

```json
{
  "$schema": "https://raw.githubusercontent.com/kythin/stackydo/main/schemas/stackydo.schema.json",
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

**Aliases** are resolved on input. With the config above, `stackydo update SD1 --status wip` stores the task as `doing`. Aliases are useful for accepting common typos or shorthand without polluting the canonical status set.

A few rules:

- Status keys are case-insensitive on input but stored exactly as you write them.
- Every status must map to `backlog`, `active`, or `archive`.
- New tasks default to a `backlog` status — `todo` if you have one, otherwise the alphabetically first backlog status.
- The literal status `deleted` is reserved. Use the `delete` command to remove tasks; you cannot transition a task into `deleted`.

### Combining `workflow` and `stack_workflows`

You can do both. The custom `workflow` becomes the default; `stack_workflows` overrides it for specific stacks.

```json
{
  "workflow": {
    "statuses": {
      "icebox": "backlog",
      "doing": "active",
      "shipped": "archive"
    }
  },
  "stack_workflows": {
    "playtest": "deck"
  }
}
```

Tasks in the `playtest` stack use the `deck` workflow; everything else uses your custom three-status flow.

## Cross-stack moves

Moving a task from a stack on one workflow to a stack on a different workflow is allowed, but it requires you to also change the status. Stackydo will not silently leave the task with a status that is invalid in its new stack:

```bash
# 'ideas' uses deck, 'work' uses kanban
stackydo update SD1 --stack work
# error: Moving to stack 'work' (kanban workflow) requires specifying a status. Valid: todo, on_hold, in_progress, blocked, in_review, done, cancelled

stackydo update SD1 --stack work --status in_progress
# ok
```

## Index ordering

Tasks within the same `(stack, status)` group have an `index` field for positional ordering — `0` is the top of the pile. Stackydo maintains the indexes automatically:

- New tasks are appended at the end of their group.
- When a task changes status or stack, the gap it left is closed and it is appended to the new group.
- When a task is deleted, its group is re-contiguified.

You can also influence ordering directly:

```bash
stackydo shuffle --stack ideas --status deck    # randomise the order
stackydo draw --source ideas/deck --target ideas/hand  # pop the top of one group, append to another
```

The `draw` command is named after the deck workflow but works with any workflow's statuses. With kanban you might use `stackydo draw --source work/todo --target work/in_progress` to pull the next thing off the top of your todo pile.

### Sort factors

When stackydo needs to fully reindex a group (e.g. after a bulk operation), it sorts by the active workflow's **index sort factors** before assigning new index values:

| Workflow | Sort factors                       |
|----------|-------------------------------------|
| `kanban` | `due`, then `priority`, then `created` |
| `deck`   | `created` only                      |

Custom workflows defined inline in `stackydo.json` use kanban's sort factors. If you need different factors for a custom workflow, that's currently a feature gap — open an issue.

## Bulk status transitions

Move many tasks between statuses in one command:

```bash
# Move all 'todo' tasks in the work stack to 'in_progress'
stackydo bulk-status --from todo --to in_progress --stack work

# Filter by tag
stackydo bulk-status --from in_progress --to done --tag sprint-3

# Preview without applying
stackydo bulk-status --from todo --to in_progress --stack work --dry-run

# Skip confirmation
stackydo bulk-status --from todo --to in_progress --stack work --yes
```

Available filters: `--stack`, `--tag`, `--priority`. The target status is validated against each affected stack's workflow, so a bulk operation that spans stacks with different workflows will refuse to apply a status that some of the matching stacks don't recognise.

## See also

- [`config.md`](config.md) — the full `stackydo.json` reference
- [`cli-reference.md`](cli-reference.md) — every command at a glance
- [`schemas/stackydo.schema.json`](../schemas/stackydo.schema.json) — JSON Schema for `stackydo.json`
