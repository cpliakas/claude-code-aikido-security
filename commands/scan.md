---
name: scan
description: Run Aikido security scan on modified files or a specific target
arguments:
  - name: target
    description: File path to scan, or --secrets-only flag. If omitted, scans all modified files.
    required: false
---

# Aikido Security Scan

Run a security scan using Aikido and report the results.

## Determine Scan Target

1. Parse the argument provided by the user:
   - If `--secrets-only` is passed (with or without a file path), use `mcp__aikido__aikido_secrets_scan` instead of `mcp__aikido__aikido_full_scan`.
   - If a file path is provided, scan that specific file.
   - If no argument is provided, identify all modified files in the current working directory using `git diff --name-only` and `git diff --name-only --cached`. Filter to scannable file types (`.js`, `.ts`, `.tsx`, `.jsx`, `.py`, `.go`, `.java`, `.rb`, `.php`, `.cs`, `.rs`, `.env`, `.yaml`, `.yml`, `.json`, `.toml`, `Dockerfile`, `.tf`, `.hcl`). Exclude `node_modules/`, `.git/`, `dist/`, `build/`, `out/`, `__pycache__/`, `vendor/`, `target/`.

## Run the Scan

2. Execute the appropriate Aikido MCP tool on the target files:
   - Default: `mcp__aikido__aikido_full_scan`
   - With `--secrets-only`: `mcp__aikido__aikido_secrets_scan`

## Report Results

3. Present a structured summary with the following format:

```
## Aikido Scan Results

**Files scanned**: <count>
**Total issues**: <count>

| Severity | SAST | Secrets | Total |
|----------|------|---------|-------|
| Critical | n    | n       | n     |
| High     | n    | n       | n     |
| Medium   | n    | n       | n     |
| Low      | n    | n       | n     |

### Findings

For each finding:
- **<severity>**: <file_path>:<line_number> | <issue_type>
  <one-line description of the issue>
```

4. End with a pass/fail determination:
   - **PASS**: Zero critical or high severity findings.
   - **FAIL**: One or more critical or high severity findings detected. List the blocking issues.

If the Aikido MCP server is unavailable, report that scanning could not be completed and suggest the user check their `AIKIDO_API_KEY` environment variable.
