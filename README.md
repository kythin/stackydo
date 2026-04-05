# stackydo

The most reliable task system I ever used was a stack of sticky notes on my desk. No app, no sync, no setup. Write it down, put it on the pile, work from the top. It survived years of trying every todo app, project board, and productivity system out there. Nothing stuck like the sticky notes did.

Stackydo is that system, digitised. Tasks are plain markdown files. You organise them into named **stacks**, the same way you'd have separate piles on your desk for different projects or areas of your life. Everything lives in a single searchable store, and because it's all just files, any tool that can read text can work with your tasks.

## Where your tasks live

You choose. Stackydo doesn't force a single model.

**Central** (default). Everything in one place, searchable across all your work:

```
~/.stackydo/
```

**Alongside a project**. Scoped to a project but not inside the repo. Great for personal task tracking on a shared codebase:

```
~/Code/my-project/
~/Code/my-project-tasks/    ← stackydo workspace here
```

**Inside the project**. Versioned with the code. The team can share tasks through git:

```
~/Code/my-project/
  .stackydo-workspace/      ← stackydo workspace here
```

A `.stackydo-context` file at the project root tells stackydo which mode to use. Or set `$STACKYDO_DIR` to point anywhere you like. See [Workspaces](docs/workspaces.md) for the full details.

## What it does

- **Stacks** to separate workstreams (work, personal, sprint-12, ideas)
- **Short IDs** like SD1, SD2 that are easy to type and remember
- **Automatic context capture** records git branch, commit, and working directory on every task
- **Subtasks and dependencies** (blocked-by, blocks, related-to)
- **MCP server** so AI assistants (Claude, Cursor, Copilot, Codex) can read and manage your tasks
- **Plain markdown+YAML files** you can grep, version, and back up
- Runs on **macOS, Linux, and Windows**

## Install

```bash
# macOS / Linux (Homebrew)
brew tap kythin/homebrew-tap && brew install stackydo

# macOS / Linux (shell installer)
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/kythin/stackydo/releases/latest/download/stackydo-installer.sh | sh

# Windows (PowerShell)
powershell -ExecutionPolicy ByPass -c "irm https://github.com/kythin/stackydo/releases/latest/download/stackydo-installer.ps1 | iex"
```

This installs two binaries: `stackydo` (CLI) and `stackydo-mcp` (MCP server).

For manual downloads, platform-specific notes, and update instructions, see the [Install Guide](docs/install.md).

## Quick start

```bash
stackydo create --title "Fix auth bug" --stack work --priority high \
  -- The login endpoint returns 500 when the token expires

stackydo list --stack work
stackydo show SD1
stackydo update SD1 --status in_progress --note "Investigating root cause"
stackydo complete SD1
stackydo search "auth"
```

## MCP server

Give your AI assistant full access to your task store. Setup is one command for Claude Code:

```bash
stackydo mcp-setup
```

For Claude Desktop, Cursor, VS Code, Codex, Windsurf, and manual config, see the [MCP Setup Guide](docs/mcp-setup.md).

## Documentation

| Guide | What's in it |
|-------|-------------|
| [Install](docs/install.md) | Platform-specific install, manual downloads, updating |
| [MCP Setup](docs/mcp-setup.md) | Connect stackydo to Claude, Cursor, VS Code, Codex, Windsurf |
| [CLI Reference](docs/cli-reference.md) | All commands, stacks, workflows, deck mode |
| [Workspaces](docs/workspaces.md) | Storage modes, `.stackydo-context`, multi-workspace, task format |
| [AI Agents](docs/ai-agents.md) | Agent setup, MCP tools, playbooks for coding agents |

## Issues & Feedback

Found a bug or have a feature request? [Open an issue](https://github.com/kythin/stackydo/issues).

## License

MIT
