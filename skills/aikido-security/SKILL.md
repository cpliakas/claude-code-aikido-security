---
name: aikido-security
description: Automatically scan code for security vulnerabilities and secrets using Aikido SAST and secrets detection. Triggers after code generation, before commits, and on security review requests.
---

# Aikido Security Scanning

You have access to three Aikido security scanning tools via MCP. Use them to catch vulnerabilities and leaked secrets in code you generate or modify.

## When to Scan

### Automatic Triggers

- **After generating or significantly modifying code files**: When you create new files or make substantial changes (not just formatting or comments) to files with scannable extensions, run a scan before presenting the final result to the user.
- **Before git commits**: When the user asks you to commit, stage changes, or you are about to create a commit, scan all staged/modified files first.
- **On explicit request**: When the user asks for a security review, audit, or vulnerability check.

### Scannable File Types

Scan files with these extensions: `.js`, `.ts`, `.tsx`, `.jsx`, `.py`, `.go`, `.java`, `.rb`, `.php`, `.cs`, `.rs`, `.swift`, `.kt`, `.scala`, `.env`, `.yaml`, `.yml`, `.json`, `.toml`, `.ini`, `.cfg`, `Dockerfile`, `.tf`, `.hcl`, `.cfn`, `.template`, `.sam`.

### Excluded Paths

Never scan: `node_modules/`, `.git/`, `dist/`, `build/`, `out/`, `.next/`, `__pycache__/`, `vendor/`, `target/`, binary files, test fixture data files, or generated lock files.

## Which Tool to Use

- **`mcp__aikido__aikido_full_scan`**: Use this as the default. It runs both SAST and secrets detection in one pass. Use it after code generation, before commits, and for general security reviews.
- **`mcp__aikido__aikido_sast_scan`**: Use when you only need to check for logic vulnerabilities, injection flaws, or insecure patterns, and secrets detection is not relevant (e.g., pure algorithm code with no config or credential handling).
- **`mcp__aikido__aikido_secrets_scan`**: Use when files are likely to contain credentials, tokens, API keys, or sensitive configuration values. Good for `.env` files, config files, or when the user is working with authentication/secrets management code.

## How to Handle Findings

### Auto-Remediate First

When Aikido reports findings, attempt to fix them yourself before surfacing them to the user:

1. Read each finding's description, file path, and line number.
2. If the fix is straightforward (e.g., parameterizing a SQL query, removing a hardcoded secret, adding input validation), apply the fix directly.
3. After fixing, re-scan to confirm the issue is resolved.
4. Only present findings to the user that you cannot confidently fix. Explain what the issue is, why you could not auto-fix it, and suggest a remediation approach.

### Presenting Findings

When surfacing findings to the user, present them inline with the relevant code context rather than as a separate report dump. For each finding, include:

- The file path and line number
- The issue type and severity
- A brief explanation of the risk
- Your suggested fix or why auto-fix was not possible

### Severity Interpretation

- **Critical/High**: These must be addressed before committing. Block the commit workflow and explain why.
- **Medium**: Flag these to the user but do not block the workflow.
- **Low**: Mention briefly; do not interrupt the workflow.

## Graceful Degradation

Scanning is non-blocking. If the Aikido MCP server is unavailable, fails to start, or returns errors:

- Do not halt the user's workflow.
- Proceed with the task normally.
- Briefly note that security scanning was unavailable so the user is aware.
- Do not retry repeatedly or troubleshoot the MCP connection unless the user asks.
