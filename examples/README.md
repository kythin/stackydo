# Examples

Importable and configuration examples you can drop into any stackydo workspace.

## `deck-of-cards.json`

A full 52-card deck as importable tasks, all in the `cards` stack with status `deck`. Useful for trying out the `deck` workflow — draw cards into your hand, play them to the table, discard when done.

### Setup

Because the `cards` stack needs to use the `deck` workflow (its statuses are `deck` / `hand` / `table` / `discard`, not the default `todo` / `in_progress` / `done`), you need a `stackydo.json` with a per-stack workflow assignment before importing.

A ready-made config is included next to the example: [`deck-of-cards.stackydo.json`](deck-of-cards.stackydo.json). Copy it into your project root as `stackydo.json`:

```bash
# In a fresh directory
curl -O https://raw.githubusercontent.com/kythin/stackydo/main/examples/deck-of-cards.stackydo.json
curl -O https://raw.githubusercontent.com/kythin/stackydo/main/examples/deck-of-cards.json
mv deck-of-cards.stackydo.json stackydo.json

stackydo init -y
stackydo import < deck-of-cards.json
```

Or if you already have a `stackydo.json`, add this to it:

```json
{
  "stack_workflows": {
    "cards": "deck"
  }
}
```

Then import:

```bash
stackydo import < deck-of-cards.json
```

### Play

```bash
# Shuffle the deck
stackydo shuffle --stack cards --status deck

# Draw the top card into your hand
stackydo draw --source cards/deck --target cards/hand

# Play it to the table
stackydo draw --source cards/hand --target cards/table

# Discard when done
stackydo draw --source cards/table --target cards/discard

# See your current state
stackydo list --stack cards --status hand
stackydo list --stack cards --status table
```

See [`docs/workflows.md`](../docs/workflows.md) for the full workflow reference, and [`docs/config.md`](../docs/config.md) for the `stackydo.json` reference.
