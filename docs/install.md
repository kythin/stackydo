# Install Guide

All install methods provide two binaries: `stackydo` (CLI) and `stackydo-mcp` (MCP server).

## npm (recommended, any platform)

The quickest way to install on any OS with Node.js:

```bash
npm install -g @kythin/stackydo
```

Update:

```bash
npm update -g @kythin/stackydo
```

You can also use npx to run without installing (useful for MCP configs):

```bash
npx -y @kythin/stackydo --version
npx -y @kythin/stackydo-mcp
```

## macOS

**Homebrew**:

```bash
brew tap kythin/homebrew-tap && brew install stackydo
```

**Shell installer** (Apple Silicon and Intel):

```bash
curl --proto '=https' --tlsv1.2 -LsSf https://github.com/kythin/stackydo/releases/latest/download/stackydo-installer.sh | sh
```

Binaries are installed to `~/.cargo/bin/`. The installer will prompt you to add it to your `PATH` if needed.

## Linux

**Homebrew**:

```bash
brew tap kythin/homebrew-tap && brew install stackydo
```

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
# npm
npm update -g @kythin/stackydo

# Homebrew
brew update && brew upgrade stackydo

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
