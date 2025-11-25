---
name: dev-quality-report
description: Generate persistent code quality reports using code-quality-analyzer. Use when running /quality-check or when quality analysis is requested during session closure. Creates reports under .claude/dev-sessions/quality-reports/.
---

# Dev Quality Report

This skill generates persistent code quality reports.

## When to Use

- During `/quality-check` command
- When quality check is requested during `/end`
- When analyzing code quality for specific paths
- Independent of any active session

## Input

- `paths`: List of paths to analyze (default: repo root)
- `options`:
  - `include`: Glob patterns to include
  - `exclude`: Glob patterns to exclude
  - `defaultExcludes`: boolean (default: true) - Apply default ignore rules

## Responsibilities

- Ask user which paths to analyze (if not provided)
- Apply default ignore rules (unless disabled)
- Generate unique `reportId` (8-char hex)
- Generate report path: `.claude/dev-sessions/quality-reports/YYYYMMDD-HHmm-qa-{reportId}.md`
- Write initial report with `Status: PENDING`
- Call `code-quality-analyzer` agent in read-only mode
- Rewrite report with full analysis and `Status: DONE` or `ABORTED`
- Return report path and summary

## Default Ignore Rules

Always ignored unless `defaultExcludes: false`:

- `**/node_modules/**`
- `**/.git/**`
- `**/dist/**`, `**/build/**`, `**/.next/**`, `**/.vite/**`
- `**/coverage/**`, `**/.turbo/**`, `**/.cache/**`
- `**/tmp/**`, `**/.tmp/**`
- `**/package-lock.json`, `**/pnpm-lock.yaml`, `**/yarn.lock`
- `**/*.min.*`

## MUST

- Use `artifact-fs-manager` for all file operations
- Never modify repository source files
- Never change any session Status
- Always set report Status to PENDING before analysis
- Always set report Status to DONE or ABORTED after

## MUST NOT

- Modify any source code files
- Install or change dependencies
- Change session status
- Run commands that modify the codebase

## Process

### Step 1: Determine Paths

If paths not provided:

1. Ask user: "Which paths should I analyze? (default: repo root)"
2. Accept user input or use default

### Step 2: Apply Ignore Rules

1. Start with default ignore patterns (if `defaultExcludes: true`)
2. Add any additional `exclude` patterns
3. Apply `include` patterns to narrow scope

### Step 3: Generate Report Identity

1. Generate timestamp: `YYYYMMDD-HHmm`
2. Generate reportId: 8-char lowercase hex (e.g., `a3c9d1b2`)
3. Construct path: `.claude/dev-sessions/quality-reports/{timestamp}-qa-{reportId}.md`

### Step 4: Write Initial Report

Use `artifact-fs-manager` to write:

```markdown
# Code Quality Report

**Report ID:** {reportId} **Generated At:** {ISO timestamp} **Status:** PENDING

## Repository Overview

- Root: {analyzed root}
- Branch: {current branch}
- Tools detected: {detecting...}
- Analyzed scope: {paths}

## Executive Summary

[Analysis in progress...]

## Critical Issues

[Analysis in progress...]

## Issue Categories

[Analysis in progress...]

## File-Level Details

[Analysis in progress...]

## Actionable Recommendations

[Analysis in progress...]

## Improvement Suggestions

[Analysis in progress...]

## Metrics

[Analysis in progress...]

## Appendix

[Analysis in progress...]
```

### Step 5: Invoke code-quality-analyzer

1. Call `code-quality-analyzer` agent with:
   - Paths to analyze
   - Ignore patterns
   - Read-only constraint
2. Receive structured analysis output

### Step 6: Write Final Report

Update report with full content:

```markdown
# Code Quality Report

**Report ID:** {reportId} **Generated At:** {ISO timestamp} **Status:** DONE

## Repository Overview

- Root: {path}
- Branch: {branch}
- Tools detected: {eslint, tsc, prettier, ...}
- Analyzed scope: {globs/paths}

## Executive Summary

{from code-quality-analyzer}

## Critical Issues

{from code-quality-analyzer}

## Issue Categories

- **Type Safety:** {summary + examples}
- **Linting:** {summary + examples}
- **Style Consistency:** {summary + examples}
- **Code Complexity:** {summary + examples}
- **Dead Code:** {summary + examples}
- **Dependency Risks:** {summary + examples}

## File-Level Details

{from code-quality-analyzer with file paths and line numbers}

## Actionable Recommendations

{from code-quality-analyzer}

## Improvement Suggestions

{from code-quality-analyzer}

## Metrics

- Total files scanned: {n}
- Total issues: {n}
- Severity breakdown: Critical/{n}, High/{n}, Medium/{n}, Low/{n}
- Top 5 files by issues: {list}

## Appendix

- Tool versions: {eslint vX, typescript vY, prettier vZ}
- Configs detected: {.eslintrc, tsconfig.json, ...}
- Notes: {any notable findings}
```

If analysis fails or is aborted:

- Set `Status: ABORTED`
- Include reason in Executive Summary

## Output

Returns:

```
{
  "success": true,
  "reportId": "a3c9d1b2",
  "reportPath": ".claude/dev-sessions/quality-reports/20250125-1430-qa-a3c9d1b2.md",
  "status": "DONE" | "ABORTED",
  "summary": {
    "totalFiles": 42,
    "totalIssues": 15,
    "critical": 2,
    "high": 5,
    "medium": 6,
    "low": 2
  }
}
```

## Error Handling

- **Invalid paths**: Report which paths are invalid
- **Permission denied**: Report unreadable paths
- **Analyzer fails**: Set Status to ABORTED, include error
- **No files to analyze**: Report empty scope, set ABORTED
- **Timeout**: Set ABORTED, include partial results if any
