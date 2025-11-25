# Development Workflow Automation

> A comprehensive Claude Code plugin that transforms your development process with structured planning, execution tracking, automated documentation, and code quality checks.

[![Version](https://img.shields.io/badge/version-1.4.0-blue.svg)](https://github.com/cicababba/claude-plugins-marketplace) [![License](https://img.shields.io/badge/license-MIT-green.svg)](../LICENSE)

---

## 📦 Installation

```bash
# Add the marketplace (if not already added)
/plugin marketplace add cicababba/claude-plugins-marketplace

# Install this plugin
/plugin install dev-workflow-automation
```

---

## ✨ Key Features

- **Smart Session Management** - Structured workflow from analysis to deployment
- **Interactive Specification Gathering** - AI-guided requirement collection with one critical question at a time
- **Automated Planning** - Break down complex tasks into manageable, executable steps
- **Progress Tracking** - Real-time status updates and execution batch monitoring
- **Code Quality Checks** - Built-in linting, testing, and complexity analysis
- **Git Integration** - Automated commits with proper conventional commit messages
- **Documentation Generation** - Auto-generate comprehensive project documentation
- **Multi-Stack Support** - React, Angular, Node.js, Java, and more
- **Error Recovery** - Smart failure handling and remediation suggestions
- **Parallel Execution** - Run independent tasks concurrently for faster delivery

---

## 🎯 Quick Start

```bash
# 1. Start a new development session
/init

# 2. Execute your implementation plan
/run

# 3. Finalize and close the session
/end

# 4. Run code quality checks (anytime)
/quality-check
```

---

## 🛠️ What's Included

### 4 Slash Commands

| Command          | Description                                                      |
| ---------------- | ---------------------------------------------------------------- |
| `/init`          | Initialize structured development sessions with context analysis |
| `/run`           | Execute implementation plans step-by-step with progress tracking |
| `/end`           | Finalize sessions with commits, docs, and quality checks         |
| `/quality-check` | Generate standalone code quality reports                         |

### 9 Reusable Skills

Skills are model-invoked capabilities that encapsulate reusable logic. Commands delegate to skills for a cleaner, more maintainable architecture.

| Skill                       | Description                                                      |
| --------------------------- | ---------------------------------------------------------------- |
| `session-manager`           | Session lifecycle management (create, load, update, close)       |
| `artifact-fs-manager`       | Unified file handler for dev artifacts (read/write/check)        |
| `dev-context-analysis`      | Code path analysis for patterns, dependencies, and tests         |
| `dev-specification-qa`      | Requirements gathering via one-question-at-a-time Q&A            |
| `dev-plan-orchestrator`     | Wrapper for development-planner agent invocation                 |
| `dev-execution-orchestrator`| Step execution, sub-agent dispatch, and progress tracking        |
| `dev-session-closure`       | Session finalization with optional commit/docs/quality triggers  |
| `dev-quality-report`        | Quality report generation wrapper for code-quality-analyzer      |
| `git-and-docs`              | Coordinate git commits and documentation generation              |

### 11 Specialized Sub-Agents

- **Development Planner** - Transforms specifications into actionable plans
- **React Frontend Developer** - React UI, TypeScript, hooks, state management
- **Angular Frontend Developer** - Angular apps, RxJS, Material, routing
- **Node.js Backend Developer** - Node.js APIs, controllers, services, jobs
- **Java Backend Developer** - Spring/Hibernate, concurrency, scalable architectures
- **React Test Writer** - React component/integration tests, RTL, e2e
- **Backend Test Writer** - Unit, integration, and contract/API tests
- **Database Architect** - Schema design, migrations, performance, indexing
- **Git Commit Manager** - Conventional commits with proper formatting
- **Documentation Generator** - Auto-generate comprehensive project docs
- **Code Quality Analyzer** - Linting, complexity, dependency audits

---

## 🏗️ Architecture

This plugin uses a layered architecture separating user interaction, orchestration logic, and execution:

```
┌─────────────────────────────────────────────────────────────┐
│                    SLASH COMMANDS                           │
│  /init    /run    /end    /quality-check                    │
│  (User-invoked entry points - thin orchestrators)           │
└─────────────────────────┬───────────────────────────────────┘
                          │ delegate to
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                       SKILLS                                │
│  session-manager | artifact-fs-manager | dev-context-analysis│
│  dev-specification-qa | dev-plan-orchestrator               │
│  dev-execution-orchestrator | dev-session-closure           │
│  dev-quality-report | git-and-docs                          │
│  (Model-invoked reusable logic - single responsibility)     │
└─────────────────────────┬───────────────────────────────────┘
                          │ invoke when needed
                          ▼
┌─────────────────────────────────────────────────────────────┐
│                      SUB-AGENTS                             │
│  development-planner | react-frontend-dev | nodejs-backend  │
│  git-commit-manager | docs-generator | code-quality-analyzer│
│  (Explicitly dispatched - heavy lifting & code generation)  │
└─────────────────────────────────────────────────────────────┘
```

**Key Principles:**

- **Commands** are thin orchestrators that delegate to skills
- **Skills** encapsulate reusable logic with single responsibility
- **Agents** perform heavy lifting (code generation, analysis, commits)
- Each layer only communicates with adjacent layers

---

## 💡 Use Cases

- **Feature Development** - From specification gathering to deployment with full traceability
- **Bug Fixes** - Structured analysis, reproduction, resolution, and verification
- **Code Refactoring** - Planned improvements with before/after comparisons
- **Technical Debt** - Systematic cleanup with quality metrics and tracking
- **Team Collaboration** - Consistent workflow and documentation across developers
- **Code Reviews** - Automated quality checks before PR submission
- **Onboarding** - Learn best practices through guided workflows

---

## 📖 Detailed Command Reference

### ✅ 1. `/init` — Start a New Development Session

**Purpose:** Prepare a clean, structured environment before any coding happens.

**What it does:**

- Identifies the relevant context by scanning specific file paths (via sub‑agents)
- Extracts existing patterns, dependencies, and related tests
- Interactively gathers the complete specification with one critical question at a time (no assumptions, one question per turn)
- Creates the session folder with:
  - `context-analysis.md` (structured findings by path)
  - `specification.md` (complete requirements and constraints)
  - `session.md` (metadata, status, and progress areas)
- Tracks the active session in `.claude/dev-sessions/.current-session`
- **Important:** this phase produces **no code**. It only creates analysis and specs. All generated docs are **in English**

**Failure handling highlights:**

- If paths are invalid or unreadable, it asks you to verify and retries
- If the slug collides, it appends `-2`, `-3`, etc.
- If new specs arrive while status is `PLANNING` or `READY`, it loops back to the Q&A step to refine them

---

### ✅ 2. `/run` — Execute the Implementation Plan

**Purpose:** Move from planning to controlled execution.

**What it does:**

- Loads the active session from `.current-session` (or lets you pick one with status `READY`/`PLANNING`)
- Verifies all session files exist and that `plan.md` contains runnable prompts (no circular dependencies)
- Updates overview fields (`Last Updated`) and sets session status to `IN_PROGRESS`
- Executes steps using the pre‑generated prompts from `plan.md`
- After each step:
  - Updates the **Todo Progress** and **Execution Batches** in `session.md`
  - Marks the step as `DONE` or `FAILED` and records reasons
  - Unblocks dependent steps and continues
- If no further steps are executable, it prints a summary and **keeps** the global status as `IN_PROGRESS` (never auto‑completes)

**Error handling:**

- Missing session or files → suggests re‑init or attempts recovery
- Failed step → marks as `FAILED`, assesses impact, and suggests remediation

---

### ✅ 3. `/end` — Finalize and Close the Session

**Purpose:** Properly wrap up the session with commits, docs, and final status.

**What it does:**

1. Loads the active session (or lets you pick one)
2. Computes final status:
   - `COMPLETED` if all steps finished
   - `PARTIALLY_COMPLETED` if some steps remained `NOT_STARTED`
   - `FAILED` if any step failed
3. Interactive closure:
   - Ask to commit changes (via **git-commit-manager**, typically passing `plan.md` + `session.md`)
   - Ask to generate final docs (via **docs-generator**; saves next to the session path)
   - Optionally run `/quality-check` on the modified paths
4. Records **End Time** and **Duration** in the session overview
5. Clears the contents of `.claude/dev-sessions/.current-session` to mark the session as closed

**Special cases:**

- If session was only `PLANNING`, it simply updates timestamps and sets status to `READY` with no other edits

---

### ✅ 4. `/quality-check` — Standalone Code Quality Report

**Purpose:** Perform a **read‑only** codebase analysis at any time, inside or outside a session.

**What it does:**

- Asks which folders/files to analyze (default: repo root)
- Invokes the **code-quality-analyzer** sub‑agent using existing project tools (linters, TS checks, complexity, dependency audit), **without modifying files**
- Writes a Markdown report to:
  ```
  .claude/dev-sessions/quality-reports/YYYYMMDD-HHmm-qa-<reportId>.md
  ```
  where `<reportId>` is an 8‑char lowercase hex token
- Populates: Executive Summary, Critical Issues, Issue Categories, File‑Level Details, Actionable Recommendations, Metrics
- Report status lifecycle: `PENDING` → `DONE` (or `ABORTED` if skipped)

**Scope defaults (ignored by default):** `node_modules`, `.git`, `dist`, `build`, `.next`, `.vite`, `coverage`, `.turbo`, `.cache`, `tmp`, lockfiles, and generated artifacts. Overrides are available with `--include` / `--exclude` flags.

---

## 🔗 How These Commands Work Together

| Phase              | Command          | Goal                                                  |
| ------------------ | ---------------- | ----------------------------------------------------- |
| 1                  | `/init`          | Understand context and define requirements            |
| 2                  | `/run`           | Implement the plan step‑by‑step with tracked progress |
| 3                  | `/end`           | Commit, document, and formally close the session      |
| Optional (anytime) | `/quality-check` | Audit code health with a read‑only report             |

**Typical lifecycle:**

1. Start a new feature → `/init`
2. Execute tasks iteratively → `/run` (repeat as needed)
3. Finalize and document → `/end`
4. (Optional) Audit code health → `/quality-check`

---

## ✅ Why This Workflow Is Effective

- **Prevents coding with unclear requirements** - No ambiguity, complete specs before code
- **Enforces disciplined execution** - Explicit status and batch tracking
- **Produces durable documentation** - Every session fully documented
- **Integrates commits, docs, and quality** - End-to-end automation
- **Keeps efforts auditable** - Full traceability from spec to deployment
- **Repeatable and collaboration-friendly** - Consistent process across team members
- **Reduces context switching** - Structured workflow keeps you focused
- **Catches issues early** - Quality checks integrated throughout

---

## 📂 Project Structure

### Plugin Structure

```
dev-workflow-automation/
├── commands/                                 # User-invoked slash commands
│   ├── init.md                              # /init command
│   ├── run.md                               # /run command
│   ├── end.md                               # /end command
│   └── quality-check.md                     # /quality-check command
├── skills/                                   # Model-invoked reusable logic
│   ├── session-manager/SKILL.md
│   ├── artifact-fs-manager/SKILL.md
│   ├── dev-context-analysis/SKILL.md
│   ├── dev-specification-qa/SKILL.md
│   ├── dev-plan-orchestrator/SKILL.md
│   ├── dev-execution-orchestrator/SKILL.md
│   ├── dev-session-closure/SKILL.md
│   ├── dev-quality-report/SKILL.md
│   └── git-and-docs/SKILL.md
├── agents/                                   # Sub-agents for heavy lifting
│   ├── development-planner.md
│   ├── react-frontend-dev.md
│   └── ... (11 agents total)
└── .claude-plugin/plugin.json               # Plugin metadata
```

### Session Structure

Each session creates the following structure:

```
.claude/dev-sessions/
├── .current-session                          # Points to active session
├── YYYY-MM-DD-HHMM-feature-name/            # Session directory
│   ├── context-analysis.md                   # Code analysis findings
│   ├── specification.md                      # Complete requirements
│   ├── session.md                            # Progress and metadata
│   ├── plan.md                               # Step-by-step execution plan
│   └── architecture-plan.yaml                # (Optional) Complex projects
└── quality-reports/
    └── YYYYMMDD-HHmm-qa-<reportId>.md       # Quality analysis reports
```

---

## 🎓 Best Practices

1. **Be thorough during `/init`** - Answer all questions carefully; good specs = good results
2. **Review the plan before `/run`** - Check `plan.md` to understand what will happen
3. **Run `/quality-check` early** - Catch issues before they compound
4. **Use incremental sessions** - Break large features into smaller sessions
5. **Commit frequently** - Use `/end` to save progress even if not fully complete
6. **Review generated docs** - Documentation is auto-generated but you should verify it
7. **Keep sessions focused** - One feature/fix per session for clarity

---

## 📝 Notes

- All generated documentation is in **English**
- `/quality-check` never writes to source files. Reports live under `.claude/dev-sessions/quality-reports/`
- Sessions are stored under `.claude/dev-sessions/YYYY-MM-DD-HHMM-<slug>/` with `.current-session` pointing to the active one
- The plugin respects `.gitignore` and won't analyze ignored files by default
- Agent selection is automatic based on your tech stack

---

## 🐛 Troubleshooting

**Session not found:**

```bash
# Check if .current-session exists
cat .claude/dev-sessions/.current-session

# List all sessions
ls -la .claude/dev-sessions/
```

**Plan execution fails:**

```bash
# Review the plan for circular dependencies
cat .claude/dev-sessions/$(cat .claude/dev-sessions/.current-session)/plan.md

# Restart from /init if needed
```

**Quality check takes too long:**

```bash
# Run on specific directories only
/quality-check --path src/components
```

---

## 👥 Authors

Ideated and developed in collaboration with:

- **Matteo Adduci** - [@cicababba](https://github.com/cicababba)
- **Giuseppe Vincenzi** - [@giuvincenzi](https://github.com/giuvincenzi)

---

## 📄 License

MIT License - see [LICENSE](../LICENSE) for details.

---

## 🔗 Resources

- [Main Marketplace](../README.md)
- [Contributing Guidelines](../CONTRIBUTING.md)
- [Claude Code Documentation](https://docs.claude.com/en/docs/claude-code/overview)
- [Report Issues](https://github.com/cicababba/claude-plugins-marketplace/issues)

---

**Built with ❤️ for the Claude Code community**
