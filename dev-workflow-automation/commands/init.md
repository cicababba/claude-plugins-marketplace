# /init — Initialize Development Session

Initialize a structured development session for any development task (new feature, bug fix, refactoring, performance optimization, etc.).

**ABSOLUTE SCOPE & RESTRICTIONS**

- This command ONLY initializes a development session.
- It MUST ONLY:
  - gather context,
  - gather specifications via Q&A,
  - generate the `context-analysis.md` and `specification.md` files,
  - update `.claude/dev-sessions/.current-session`.
- It MUST NEVER:
  - perform any planning,
  - generate a development plan,
  - create any section or content titled "Plan", "Planning", "Implementation Plan", "Roadmap", or similar.
- Planning is done ONLY by the separate `development-planner` agent via `dev-plan-orchestrator` skill, AFTER this command has completed.
- **IMPORTANT**: This command must never produce code snippets or implementations, only analysis and specifications.
- **IMPORTANT**: All generated files (context-analysis.md, specification.md) must ALWAYS be written in ENGLISH, regardless of the language used in the conversation.

---

## 1. Create Session

> **Skill:** Use `session-manager` to create a new session

- Ask user for task description
- Generate session slug from description
- Create session directory: `.claude/dev-sessions/YYYY-MM-DD-HHMM-${slug}/`
- Initialize `session.md` with Session Overview (Status: PLANNING)
- Update `.claude/dev-sessions/.current-session` with session path

---

## 2. Context Analysis

> **Skill:** Use `dev-context-analysis` to analyze relevant code paths

- Ask the user which files or folders to analyze for context
- Give examples (e.g., `/src/auth`, `/src/models/user.ts`)
- When user provides paths, invoke `dev-context-analysis` skill with:
  - Session path
  - List of paths to analyze
- The skill will:
  - Analyze each path for patterns, dependencies, tests, implementation details
  - Generate structured `context-analysis.md`
  - Save via `artifact-fs-manager`
- Present a **Quick Summary** to the user

**IMPORTANT:** In this step you are ONLY analyzing. Do NOT start planning, do NOT propose implementation steps.

---

## 3. Specification Gathering

> **Skill:** Use `dev-specification-qa` to gather complete requirements

- Invoke `dev-specification-qa` skill with:
  - Session path
  - Context analysis content
  - Task description
- The skill will:
  - Ask exactly ONE targeted question at a time
  - Prioritize highest-impact unknowns
  - Allow up to 4 follow-ups for unclear answers
  - Build structured specification
  - Ask about test requirements
  - Present Quick Summary for confirmation
  - Save `specification.md` via `artifact-fs-manager` after confirmation

**CRITICAL RULE:** Even after summary confirmation and test preference is known, NO planning happens here.

---

## 4. Confirm Session Files

- Verify via `artifact-fs-manager`:
  - `context-analysis.md` exists and contains structured analysis
  - `specification.md` exists and contains full specification
- Verify via `session-manager`:
  - `.current-session` points to session directory
  - Session Status is `PLANNING`

---

## 5. Next Steps

- Inform user that files are created
- Invite them to review the artifacts
- Ask if they want to start the `dev-plan-orchestrator` skill to create the development plan
- **But NEVER plan here**

---

## Error Handling

- No coding in this phase
- If context analysis fails: ask user to verify paths (via `dev-context-analysis` error handling)
- If file creation fails: fallback directory (via `artifact-fs-manager` error handling)
- On slug collision: append -2, -3, ... (via `session-manager`)
- If user declines tests: include this decision in specification

---

## Validation Checklist

1. `context-analysis.md` contains structured analysis (via `artifact-fs-manager`)
2. `specification.md` contains full spec (via `artifact-fs-manager`)
3. `.current-session` points to session (via `session-manager`)
4. No code generated
5. No planning performed
6. Planning only happens later via `dev-plan-orchestrator`
