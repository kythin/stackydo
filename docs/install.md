# Install Guide

All install methods provide two binaries: `stackydo` (CLI) and `stackydo-mcp` (MCP server).

## npx (recommended, any platform with Node.js)

Run on demand without installing — npx fetches the binaries on first use and caches them:

```bash
npx -y @kythin/stackydo --version
npx -y --package @kythin/stackydo stackydo-mcp
```

This is the simplest path for MCP configs (point your client at `npx` with the args above — see [MCP Setup](mcp-setup.md)) and for one-off CLI use.

## npm (global install)

If you'll be running `stackydo` interactively many times a day, a global install avoids npx's startup overhead:

```bash
npm install -g @kythin/stackydo
```

Update:

```bash
npm update -g @kythin/stackydo
```

## macOS

**Shell installer** (Apple Silicon and Intel):

```bash
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/kythin/stackydo/releases/latest/download/stackydo-installer.sh | sh
```

Binaries are installed to `~/.cargo/bin/`. The installer will prompt you to add it to your `PATH` if needed.

## Linux

**Shell installer** (x86_64 and aarch64):

```bash
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/kythin/stackydo/releases/latest/download/stackydo-installer.sh | sh
```

**Manual download** from the [latest release](https://github.com/kythin/stackydo/releases/latest):

```bash
# x86_64
tar xf stackydo-x86_64-unknown-linux-gnu.tar.xz
sudo mv stackydo stackydo-mcp /usr/local/bin/

# aarch64
tar xf stackydo-aarch64-unknown-linux-gnu.tar.xz
sudo mv stackydo stackydo-mcp /usr/local/bin/
```

## Windows

**PowerShell installer**:

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://github.com/kythin/stackydo/releases/latest/download/stackydo-installer.ps1 | iex"
```

**Manual download**: grab `stackydo-x86_64-pc-windows-msvc.tar.xz` from the [latest release](https://github.com/kythin/stackydo/releases/latest), extract it, and add the folder to your `PATH`.

## Update

```bash
# npx — force a fresh download (otherwise uses the cached version)
npx -y @kythin/stackydo@latest --version

# npm
npm update -g @kythin/stackydo

# Shell/PowerShell installer
stackydo-update

# Or re-run any installer. It will overwrite the existing version.
```

## Verify

```bash
stackydo --version
stackydo doctor
```

`doctor` checks that your workspace is healthy and reports any issues.
