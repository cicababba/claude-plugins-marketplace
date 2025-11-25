# /quality-check — Code Quality Report

Generate a comprehensive code quality analysis report without modifying any files.

**ABSOLUTE SCOPE & RESTRICTIONS**

- This command is **read-only** — it never modifies source code
- It generates a Markdown report under `.claude/dev-sessions/quality-reports/`
- It does NOT depend on or update any active session
- It does NOT change any session Status

---

## 1. Gather Analysis Scope

> **Skill:** Delegate to `dev-quality-report`

Ask the user which paths to analyze:

- Default: repository root
- Accept specific paths (e.g., `src/components`, `lib/utils`)
- Accept options:
  - `--include "<glob1,glob2,...>"` to restrict scope
  - `--exclude "<glob1,glob2,...>"` to add ignore rules
  - `--default-excludes=false` to disable default ignores (discouraged)

---

## 2. Run Quality Analysis

> **Skill:** Use `dev-quality-report` skill

Invoke `dev-quality-report` with:

- Paths to analyze
- Include/exclude options
- Default excludes preference

The skill will:

- Apply default ignore rules (node_modules, .git, dist, etc.)
- Generate unique report ID and path
- Write initial report with `Status: PENDING`
- Call `code-quality-analyzer` agent (read-only)
- Write final report with analysis and `Status: DONE`

---

## 3. Report Output

The skill generates a report at:

```
.claude/dev-sessions/quality-reports/YYYYMMDD-HHmm-qa-<reportId>.md
```

Report includes:

- **Repository Overview**: Root, branch, detected tools, analyzed scope
- **Executive Summary**: Overall quality assessment
- **Critical Issues**: High-priority problems requiring immediate attention
- **Issue Categories**:
  - Type Safety
  - Linting
  - Style Consistency
  - Code Complexity
  - Dead Code
  - Dependency Risks
- **File-Level Details**: Specific locations with line numbers
- **Actionable Recommendations**: Concrete steps to fix issues
- **Improvement Suggestions**: Broader quality enhancement ideas
- **Metrics**: Files scanned, issue counts, severity breakdown

---

## 4. Return Summary

After report is generated:

- Echo the report path
- Show compact summary:
  - Total files scanned
  - Total issues found
  - Breakdown by severity (Critical/High/Medium/Low)
  - Top problematic files

---

## Default Ignore Rules

Always ignored unless `--default-excludes=false`:

- `**/node_modules/**`
- `**/.git/**`
- `**/dist/**`, `**/build/**`, `**/.next/**`, `**/.vite/**`
- `**/coverage/**`, `**/.turbo/**`, `**/.cache/**`
- `**/tmp/**`, `**/.tmp/**`
- `**/package-lock.json`, `**/pnpm-lock.yaml`, `**/yarn.lock`
- `**/*.min.*`

---

## Report Status Values

- `PENDING`: Report created, analysis not yet complete
- `DONE`: Analysis completed successfully, all sections populated
- `ABORTED`: Analysis intentionally stopped (invalid paths, user abort, error)

---

## Validation Checklist

1. Report file exists under `.claude/dev-sessions/quality-reports/`
2. Report follows naming scheme: `YYYYMMDD-HHmm-qa-<reportId>.md`
3. Status is one of: `PENDING`, `DONE`, `ABORTED`
4. No repository files were modified
5. No session status was changed

---

## Notes

- Keep the analyzer strictly read-only
- Never install or change dependencies
- Prefer existing project tools (ESLint, TSC, Prettier)
- For monorepos, aggregate results in one report
