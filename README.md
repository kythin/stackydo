# stackydo

**The shared task list for you and your AI agents.**

Every task is a plain markdown file with structured context, organised into named **stacks**, under a workflow that is semi-opinionated but flexible: usable as-is as a general development lifecycle, or tuned to how you actually work. Because tasks are just files, an agent can read your whole workload, pick up a task, update it, and hand it back through the same store you work from.

It started as a stack of sticky notes. Across 20 years of building software, from solo freelance work to small product teams to large multi-department engineering orgs, that pile on the desk outlasted every todo app, project board, and productivity system I tried. It survived because it did one thing none of the others could: it only ever showed me one note. The stack could grow as deep as it liked, but a glance at the desk offered just the top of it. Complete tunnel vision of one achievable goal.

That goal can sit at any altitude: "Automated UI testing for the website" might span a dozen Jira tickets, or the next note down might be a ten-minute fix. The stack does not care about the level, only the focus. Git issues, epics, roadmaps and strategy tooling still own the breadth and the long view. The stack holds the one thing you are pointed at right now. Scribble down a stray thought and jam it under the top note to handle next. Triage and reprioritise the pile when it needs it, then drop back into tunnel vision.

I've rebuilt that idea many times since, on paper and on screen. stackydo is the version that stuck: one searchable store of plain files, no app to log into, no database, no lock-in.

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

A `stackydo.json` file at the project root tells stackydo which mode to use, and is also where you customise workflows, per-stack workflow assignments, and project context. Or set `$STACKYDO_DIR` to point anywhere you like. See [Workspaces](docs/workspaces.md) and [Configuration](docs/config.md) for the full details.

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
# npm (any platform)
npm install -g @kythin/stackydo

# Homebrew (macOS / Linux)
brew tap kythin/homebrew-tap && brew install stackydo
```

This installs two binaries: `stackydo` (CLI) and `stackydo-mcp` (MCP server).

For shell/PowerShell installers, manual downloads, and other options, see the [Install Guide](docs/install.md).

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

Or use npx in any MCP config without installing first:

```json
{ "command": "npx", "args": ["-y", "--package", "@kythin/stackydo", "stackydo-mcp"] }
```

For Claude Desktop, Cursor, VS Code, Codex, Windsurf, and manual config, see the [MCP Setup Guide](docs/mcp-setup.md).

## Working with AI

Early use taught me that stackydo works best with AI agents in the loop. A few patterns that have held up:

**Pick the workspace to fit the work.** Keep a global store (`~/.stackydo/`) for cross-cutting tasks: admin, reminders, the things that belong to no single repo. For code, use a project workspace stored inside or alongside the repo, so tasks travel with the project. The same setup suits a monorepo, or a folder sitting above several clones that make up one system. See [Workspaces](docs/workspaces.md) for the placement options.

**The files make sense with or without stackydo.** Tasks and their context are structured markdown, so any LLM can read a workspace and follow the same conventions even where stackydo is not installed. Where it is installed, the MCP tools do that same job far more cheaply in tokens, which adds up over a long session.

**One task spine, many tools.** It does not try to be your only task system. Claude, Codex, and Gemini all ship capable task tracking of their own. What it adds is a shared spine. Switch tools without losing your place, or point several at the same workspace. If you run a single tool with agent-team support, that is usually the better setup: an orchestrator delegating work while sub-agents record their own progress in real time, with no long context handoffs in between.

**WSL, made less painful.** If you develop on Windows through WSL, run the stackydo MCP server both inside WSL and on the Windows host. Your desktop app and your WSL environment then share one set of tasks and one context, instead of two disconnected worlds.

For agent setup, MCP tools, and ready-made playbooks, see the [AI Agents guide](docs/ai-agents.md).

## Documentation

| Guide | What's in it |
|-------|-------------|
| [Install](docs/install.md) | Platform-specific install, manual downloads, updating |
| [MCP Setup](docs/mcp-setup.md) | Connect stackydo to Claude, Cursor, VS Code, Codex, Windsurf |
| [CLI Reference](docs/cli-reference.md) | All commands, stacks, quick reference |
| [Configuration](docs/config.md) | The `stackydo.json` file: every field, examples, schema |
| [Workflows](docs/workflows.md) | Stages, built-in workflows, custom workflows, per-stack assignment, index ordering |
| [Workspaces](docs/workspaces.md) | Storage modes, workspace discovery, multi-workspace, task format |
| [AI Agents](docs/ai-agents.md) | Agent setup, MCP tools, playbooks for coding agents |

## Issues & Feedback

Found a bug or have a feature request? [Open an issue](https://github.com/kythin/stackydo/issues).

## License

MIT
