# /run — Execute Development Plan

Execute the next available steps from the active development session.

**ABSOLUTE SCOPE & RESTRICTIONS**

- This command executes planned steps from `plan.md`
- It MUST update progress in `session.md`
- It MUST keep Status as `IN_PROGRESS`
- It MUST NEVER set Status to `COMPLETED` (this is done by `/end`)

---

## 1. Load Active Session

> **Skill:** Use `session-manager` to load the current session

- Read `.claude/dev-sessions/.current-session`
- If empty or missing:
  - Filter sessions by Status: `READY` or `PLANNING`
  - Present filtered list to user
  - Let user select one
  - Update `.current-session` with selection
- Load session and show current status

---

## 2. Validate Session State

> **Skill:** Use `artifact-fs-manager` to verify artifacts exist

- Check that all required files exist and are readable:
  - `context-analysis.md`
  - `specification.md`
  - `plan.md`
  - `session.md`
- If `plan.md` is missing:
  - Suggest running `dev-plan-orchestrator` skill first
  - Exit command

---

## 3. Execute Steps

> **Skill:** Use `dev-execution-orchestrator` to run available steps

- Invoke `dev-execution-orchestrator` with:
  - Session path
  - Execution mode: `full-batch` (or `next-step` if user prefers)
- The skill will:
  - Validate plan integrity (no circular dependencies)
  - Set session Status to `IN_PROGRESS`
  - Determine executable steps (dependencies satisfied)
  - Dispatch to appropriate sub-agents using pre-generated prompts
  - Update step status after each completion
  - Update batch status in Execution Batches
  - Update Progress counter

---

## 4. Report Progress

After execution completes:

- Show summary of executed steps
- Show current progress (X/Y steps completed)
- List any blocked steps and why
- List next executable steps (if any)
- Keep Status as `IN_PROGRESS`

**IMPORTANT:** Do NOT change global session status to `COMPLETED`

---

## Error Handling

- **No active session**: Suggest running `/init` first
- **Session files missing**: Suggest re-initialization or recovery (via skill error handling)
- **Step execution fails**:
  - Mark step as `FAILED` with reason
  - Check if failure blocks other steps
  - Suggest remediation
  - Continue with other executable steps if possible
- **Dependency conflict**: Report which steps are blocked and why
- **No executable steps**: Report current state, keep Status as `IN_PROGRESS`
