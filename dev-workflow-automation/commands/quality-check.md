# /wf-dev:quality — Code Quality Report Command

Before running a code quality analysis using the `code-quality-analyzer` sub‑agent, ask the user which path/paths analyze, otherwise use the repo root and then persist a Markdown report under `.claude/dev-sessions/quality-reports/` directory.

> This command does **not** modify code. It executes a read‑only analysis and writes a report file.

---

## What this command does

1. **Report storage**
  - Reports are always written under the global path:  
    `.claude/dev-sessions/quality-reports/`
  - File naming convention:  
    `YYYYMMDD-HHmm-qa-<reportId>.md`  
    Example: `.claude/dev-sessions/quality-reports/20250922-1540-qa-a3c9d1b2.md`
  - `reportId` is an 8-character random lowercase hex string.
  - The command does not depend on or update `.current-session`.

2. **Run the quality analyzer (sub‑agent)**
   - Invoke **code-quality-analyzer** with read‑only tools (linters, TS checks, formatting diffs, complexity scans, dependency audit) and **no file modifications**.
   - Capture the structured output (Executive Summary, Critical Issues, Issue Categories, File-Level Details, Recommendations).

3. **Write a report file**
   - Path: `.claude/dev-sessions/quality-reports/{YYYYMMDD-HHmm}-qa-{reportId}.md`
   - Include the analyzer output verbatim + normalized metadata and status block.
   - Status lifecycle for the report:
     - `PENDING` → set immediately before launching the analyzer
     - `DONE` → set if analyzer completed successfully
     - `ABORTED` → set if the command exits early (e.g. user aborted, repo unreadable)

4. **Return a short summary**
   - Echo the created path and a compact summary (counts by issue category, if available).

---

## Scope & ignore rules (MANDATORY)

- By default the analyzer must **ignore** the following paths/globs:
  - `**/node_modules/**`
  - `**/.git/**`
  - `**/dist/**`, `**/build/**`, `**/.next/**`, `**/.vite/**`
  - `**/coverage/**`, `**/.turbo/**`, `**/.cache/**`
  - `**/tmp/**`, `**/.tmp/**`
  - lockfiles and generated artifacts: `**/package-lock.json`, `**/pnpm-lock.yaml`, `**/yarn.lock`, `**/*.min.*`
- Optional overrides:
  - `--include "<glob1,glob2,...>"` restricts analysis scope
  - `--exclude "<glob1,glob2,...>"` adds ignore rules
  - `--default-excludes=false` disables the default ignore set (discouraged; log a warning)

---

## Report filename

```
.claude/dev-sessions/quality-reports/YYYYMMDD-HHmm-qa-<reportId>.md
```

- `reportId`: 8‑char lower‑hex random token (e.g., `a3c9d1b2`).
- Example: `.claude/dev-sessions/quality-reports/20250922-0211-qa-a3c9d1b2.md`

---

## Report Markdown template (MANDATORY)

```md
# Code Quality Report

**Report ID:** <reportId>  
**Generated At:** <ISO timestamp>  
**Status:** <PENDING | DONE | ABORTED>

## Repository Overview
- Root: <path>
- Branch: <branch-if-known>
- Tools detected: <eslint | tsc | prettier | ...>
- Analyzed scope: <globs/paths>

## Executive Summary
<from sub-agent>

## Critical Issues
<from sub-agent>

## Issue Categories
- **Type Safety:** <summary + examples>
- **Linting:** <summary + examples>
- **Style Consistency:** <summary + examples>
- **Code Complexity:** <summary + examples>
- **Dead Code:** <summary + examples>
- **Dependency Risks:** <summary + examples>

## File-Level Details
<from sub-agent with file paths and line numbers>

## Actionable Recommendations
<from sub-agent>

## Improvement Suggestions
<from sub-agent>

## Metrics
- Total files scanned: <n>
- Total issues: <n>
- Severity breakdown: <Critical/High/Med/Low>
- Top 5 files by issues: <list>

## Appendix
- Tool versions: <eslint vX, typescript vY, prettier vZ, ...>
- Configs detected: <.eslintrc, tsconfig.json, prettier config, ...>
- Notes: <anything notable>
```

**Status semantics:**  
- `PENDING`: report created but the analyzer has not returned (or user aborted before run).  
- `DONE`: analyzer completed and all sections are populated.  
- `ABORTED`: the command intentionally did not run (invalid paths, or explicit user skip).

---

## Output
- Markdown file at the path above.
- Console summary with: issues by category, status, and file path.

---

## Validation checklist (must pass)

- Report file exists under `.claude/dev-sessions/quality-reports/` with the required naming scheme.
- Status is one of `PENDING | DONE | ABORTED` and matches execution outcome.
- Analyzer output sections are present when status is `DONE`.
- No repository files are modified.
- Global session status is unchanged.

---

## Notes

- Keep the analyzer strictly read‑only; never install or change dependencies.
- Prefer existing project tools; do not invent new linters or configs.
- For monorepos, run per‑package scans and aggregate in one report.
