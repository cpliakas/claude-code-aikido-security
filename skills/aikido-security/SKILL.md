---
name: aikido-security
description: Automatically scan code for security vulnerabilities and secrets using Aikido SAST and secrets detection. Triggers after code generation, before commits, and on security review requests.
---

# Aikido Security Scanning

You have access to three Aikido security scanning tools via MCP. Use them to catch vulnerabilities and leaked secrets in code you generate or modify.

## When to Scan

### Automatic Triggers

- **After generating, adding, or modifying first-party code files**: Always run a scan on generated, added, and modified first-party code unless the user's prompt explicitly says not to. Provide the full file content to the scanner for analysis.
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

### Remediation Loop

When Aikido reports findings, use the remediation guidance provided by Aikido to fix the issues. Then re-scan the code to confirm the fixes are effective and no new issues were introduced. Continue this cycle until the code passes with zero remaining or newly introduced security issues.

1. Read each finding's description, file path, line number, and Aikido's remediation guidance.
2. Apply the recommended fix.
3. Re-scan the fixed code to confirm resolution.
4. If new issues appear, repeat the cycle.
5. Only present findings to the user if you cannot resolve them after multiple attempts. Explain what the issue is, what you tried, and suggest next steps.

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
- Inform the user that the Aikido MCP server is not available and direct them to https://help.aikido.dev/ide-plugins/aikido-mcp for setup instructions.
- Do not retry repeatedly or troubleshoot the MCP connection unless the user asks.
