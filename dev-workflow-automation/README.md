# Development Workflow Commands

This repository follows a structured and automated development workflow designed to improve clarity, consistency, and code quality across all tasks. Instead of jumping straight into coding, each development session goes through a guided process of analysis, planning, execution, and proper closure. This reduces ambiguity, prevents unfinished work, and ensures every feature or fix is fully documented. The workflow is powered by specialized Claude sub‑agents, each responsible for a specific phase of the lifecycle. The commands below explain how this system works step by step.

---

## ✅ 1. `/dev-init` — Start a New Development Session

**Purpose:** Prepare a clean, structured environment before any coding happens.

**What it does:**
- Identifies the relevant context by scanning specific file paths (via sub‑agents).
- Extracts existing patterns, dependencies, and related tests.
- Interactively gathers the complete specification with one critical question at a time (no assumptions, one question per turn).
- Creates the session folder with:
  - `context-analysis.md` (structured findings by path)
  - `specification.md` (complete requirements and constraints)
  - `session.md` (metadata, status, and progress areas)
- Tracks the active session in `.claude/dev-sessions/.current-session`.
- **Important:** this phase produces **no code**. It only creates analysis and specs. All generated docs are **in English**.

**Failure handling highlights:**
- If paths are invalid or unreadable, it asks you to verify and retries.
- If the slug collides, it appends `-2`, `-3`, etc.
- If new specs arrive while status is `PLANNING` or `READY`, it loops back to the Q&A step to refine them.

---

## ✅ 2. `/dev-run` — Execute the Implementation Plan

**Purpose:** Move from planning to controlled execution.

**What it does:**
- Loads the active session from `.current-session` (or lets you pick one with status `READY`/`PLANNING`).
- Verifies all session files exist and that `plan.md` contains runnable prompts (no circular dependencies).
- Updates overview fields (`Last Updated`) and sets session status to `IN_PROGRESS`.
- Executes steps using the pre‑generated prompts from `plan.md`.
- After each step:
  - Updates the **Todo Progress** and **Execution Batches** in `session.md`.
  - Marks the step as `DONE` or `FAILED` and records reasons.
  - Unblocks dependent steps and continues.
- If no further steps are executable, it prints a summary and **keeps** the global status as `IN_PROGRESS` (never auto‑completes).

**Error handling:**
- Missing session or files → suggests re‑init or attempts recovery.
- Failed step → marks as `FAILED`, assesses impact, and suggests remediation.

---

## ✅ 3. `/dev-end` — Finalize and Close the Session

**Purpose:** Properly wrap up the session with commits, docs, and final status.

**What it does:**
1. Loads the active session (or lets you pick one).
2. Computes final status:
   - `COMPLETED` if all steps finished
   - `PARTIALLY_COMPLETED` if some steps remained `NOT_STARTED`
   - `FAILED` if any step failed
3. Interactive closure:
   - Ask to commit changes (via **git-commit-manager**, typically passing `plan.md` + `session.md`).
   - Ask to generate final docs (via **docs-generator**; saves next to the session path).
   - Optionally run `/quality-check` on the modified paths.
4. Records **End Time** and **Duration** in the session overview.
5. Clears the contents of `.claude/dev-sessions/.current-session` to mark the session as closed.

**Special cases:**
- If session was only `PLANNING`, it simply updates timestamps and sets status to `READY` with no other edits.

---

## ✅ 4. `/quality-check` — Standalone Code Quality Report

**Purpose:** Perform a **read‑only** codebase analysis at any time, inside or outside a session.

**What it does:**
- Asks which folders/files to analyze (default: repo root).
- Invokes the **code-quality-analyzer** sub‑agent using existing project tools (linters, TS checks, complexity, dependency audit), **without modifying files**.
- Writes a Markdown report to:
  ```
  .claude/dev-sessions/quality-reports/YYYYMMDD-HHmm-qa-<reportId>.md
  ```
  where `<reportId>` is an 8‑char lowercase hex token.
- Populates: Executive Summary, Critical Issues, Issue Categories, File‑Level Details, Actionable Recommendations, Metrics.
- Report status lifecycle: `PENDING` → `DONE` (or `ABORTED` if skipped).

**Scope defaults (ignored by default):**
`node_modules`, `.git`, `dist`, `build`, `.next`, `.vite`, `coverage`, `.turbo`, `.cache`, `tmp`, lockfiles, and generated artifacts.
Overrides are available with `--include` / `--exclude` flags.

---

## 🔗 How These Commands Work Together

| Phase | Command | Goal |
|------|---------|------|
| 1 | `/dev-init` | Understand context and define requirements |
| 2 | `/dev-run` | Implement the plan step‑by‑step with tracked progress |
| 3 | `/dev-end` | Commit, document, and formally close the session |
| Optional (anytime) | `/quality-check` | Audit code health with a read‑only report |

**Typical lifecycle:**
1. Start a new feature → `/dev-init`
2. Execute tasks iteratively → `/dev-run` (repeat as needed)
3. Finalize and document → `/dev-end`
4. (Optional) Audit code health → `/quality-check`

---

## ✅ Why This Workflow Is Effective

- Prevents coding with unclear or shifting requirements
- Enforces disciplined execution with explicit status and batch tracking
- Produces durable documentation for every session
- Integrates commits, docs, and code‑quality checks
- Keeps efforts auditable, repeatable, and collaboration‑friendly

---

### Notes
- All generated documentation is in **English**.
- `/quality-check` never writes to source files. Reports live under `.claude/dev-sessions/quality-reports/`.
- Sessions are stored under `.claude/dev-sessions/YYYY-MM-DD-HHMM-<slug>/` with `.current-session` pointing to the active one.

---

## 👥 Authors

Ideated and developed in collaboration with:
- **Matteo Adduci** - [@cicababba](https://github.com/cicababba)
- **Giuseppe Vincenzi** - [@giuvincenzi](https://github.com/giuvincenzi)
