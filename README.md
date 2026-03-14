# Aikido Security Plugin for Claude Code

Integrates [Aikido's](https://www.aikido.dev/) SAST, secrets detection, and full security scanning directly into the Claude Code workflow. Code generated or modified by Claude is automatically scanned for vulnerabilities and leaked secrets before it reaches your repository, shifting security left to the point of code generation.

## Prerequisites

- **Node.js** (v18+): required for `npx` to run the Aikido MCP server
- **Aikido account**: sign up at [aikido.dev](https://www.aikido.dev/) if you don't have one
- **Personal Access Token (PAT)**: generated from the Aikido dashboard

## Installation

### 1. Generate an Aikido Personal Access Token

This is a one-time step:

1. Log into the [Aikido dashboard](https://app.aikido.dev/)
2. Navigate to **Settings → Integrations → IDE → MCP**
3. Generate a new Personal Access Token
4. Copy the token (you won't be able to see it again)

### 2. Set the Environment Variable

Add the token to your shell profile:

```sh
echo 'export AIKIDO_API_KEY="your_token_here"' >> ~/.zshrc
source ~/.zshrc
```

Replace `~/.zshrc` with `~/.bashrc` if you use bash. Open a new terminal session if `source` doesn't pick it up.

### 3. Add the Marketplace and Install

```
/plugin marketplace add cpliakas/claude-code-aikido-security
/plugin install aikido-security
```

When prompted, select the **User** scope to install the plugin globally.

### 4. Verify

Start a new Claude Code session and confirm the plugin loaded:

- The Aikido MCP server should appear in the active MCP connections
- The `aikido-security` skill should be listed in active skills
- Run `/aikido-security:scan` on any file to confirm scanning works

## Usage

### Automatic Scanning

The `aikido-security` skill operates automatically. No manual invocation is needed for normal workflows:

- **After code generation**: When Claude creates or significantly modifies code files, it scans them before presenting the result.
- **Before commits**: When you ask Claude to commit, it scans staged files first and blocks the commit if critical or high severity issues are found.
- **On request**: Ask Claude for a "security review" or "vulnerability check" and it will scan the relevant files.

Findings are auto-remediated when possible. Claude fixes straightforward issues (SQL injection, hardcoded secrets, missing input validation) and re-scans to confirm. Only issues it cannot confidently fix are surfaced to you.

### `/aikido-security:scan` Command

Run targeted scans manually:

```
/aikido-security:scan                        # Scan all modified files
/aikido-security:scan src/auth/login.ts      # Scan a specific file
/aikido-security:scan --secrets-only         # Secrets detection only
```

Output includes a severity breakdown (critical/high/medium/low), finding type (SAST vs secrets), file locations, and a pass/fail determination. A scan passes if there are zero critical or high findings.

## Available Tools

The plugin registers three Aikido MCP tools:

- **`aikido_full_scan`**: Runs both SAST and secrets detection in a single pass. This is the default tool used by the skill and the `/aikido-security:scan` command. Use it for comprehensive security checks.

- **`aikido_sast_scan`**: Static application security testing only. Detects injection vulnerabilities, insecure patterns, logic flaws, and other code-level security issues. Use when you know the code doesn't handle credentials or configuration values.

- **`aikido_secrets_scan`**: Secrets and credential detection only. Identifies hardcoded API keys, tokens, passwords, and other sensitive values. Use for `.env` files, configuration files, or authentication-related code.

## Troubleshooting

**MCP not found / fails to start**
The `AIKIDO_API_KEY` environment variable is not set in the shell Claude Code was launched from. Verify with `echo $AIKIDO_API_KEY` in your terminal. If empty, re-export it and source your shell profile.

**`npx` not found**
Node.js is not installed or not on your PATH. Install it from [nodejs.org](https://nodejs.org/) or via your package manager.

**Authentication errors**
Your PAT is expired or incorrect. Generate a new one from the Aikido dashboard at **Settings → Integrations → IDE → MCP**.

**Scans returning no results unexpectedly**
The file type may not be supported by Aikido, or the file path matches a default exclusion rule (`node_modules/`, `dist/`, `build/`, etc.). Check that the file extension is in the supported list.

**Claude Code launched from a GUI without shell environment**
Environment variables set in `~/.zshrc` or `~/.bashrc` are not available when Claude Code is launched from a desktop shortcut or IDE integration that doesn't source the shell profile. Always launch Claude Code from a terminal session.

## Security

The `AIKIDO_API_KEY` Personal Access Token should never be committed to source control or hardcoded in plugin files. The `${AIKIDO_API_KEY}` environment variable reference in `.mcp.json` is intentional; it resolves at runtime from your shell environment. Add `AIKIDO_API_KEY` to your `.gitignore` and secrets manager as appropriate.

## Global vs Per-Project Installation

This plugin is designed for **global installation** (`~/.claude/`) since the same Aikido workspace and PAT typically applies across all projects on your machine.

For teams that want to enforce scanning for all contributors, commit a reference to this marketplace in the repo's `.claude/settings.json`. Each contributor still needs their own `AIKIDO_API_KEY` set locally, but the plugin configuration and scanning behavior will be consistent across the team.
