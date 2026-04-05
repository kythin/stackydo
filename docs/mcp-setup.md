# MCP Setup Guide

Stackydo includes an MCP (Model Context Protocol) server that gives AI assistants full access to your task store. The `stackydo-mcp` binary communicates over stdio and works with any MCP-compatible client.

## Claude Code (CLI / VS Code / JetBrains)

One command:

```bash
stackydo mcp-setup
```

This runs `claude mcp add` to register the server. Options:

```bash
stackydo mcp-setup                    # project scope (default)
stackydo mcp-setup --scope user       # available in all projects
```

**Manual alternative**: add to your project's `.mcp.json`:

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

## Claude Desktop

Edit the config file for your platform:

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

To point at a specific task store:

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

## Cursor

Open **Settings > MCP**, or edit `.cursor/mcp.json` in your project root:

```json
{
  "mcpServers": {
    "stackydo": {
      "command": "stackydo-mcp"
    }
  }
}
```

For global access, use `~/.cursor/mcp.json` instead. Restart Cursor after setup.

## VS Code (Copilot)

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

Enable MCP in VS Code settings: set `chat.mcp.enabled` to `true`. Restart VS Code.

## OpenAI Codex CLI

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

## Windsurf

Open **Settings > MCP**, or edit `~/.codeium/windsurf/mcp_config.json`:

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

## Project-local workspace

To scope the MCP server to a project-specific task store:

```bash
stackydo init --here --dir .stackydo-workspace
```

This creates a `.stackydo-context` file that tells `stackydo-mcp` to use the local directory. Any MCP client running from the project root will automatically pick it up.

See [Workspaces](workspaces.md) for more on how workspace resolution works.
