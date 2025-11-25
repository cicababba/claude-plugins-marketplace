---
name: dev-plan-orchestrator
description: Thin wrapper around development-planner agent. Use after context-analysis.md and specification.md exist to generate plan.md and session.md. Orchestrates planning without implementing planning logic itself.
---

# Dev Plan Orchestrator

This skill orchestrates the planning phase by coordinating with the `development-planner` agent.

## When to Use

- After `/init` completes with `context-analysis.md` and `specification.md`
- When user explicitly requests planning
- When session status needs to transition from PLANNING to READY

## Input

- `sessionPath`: Path to the current session directory (or read from `.current-session`)

## Responsibilities

- Load current session via `session-manager`
- Verify `context-analysis.md` and `specification.md` exist via `artifact-fs-manager`
- Read content of both artifacts
- Call `development-planner` agent with artifact contents
- Persist returned artifacts via `artifact-fs-manager`:
  - `plan.md` (always)
  - `session.md` with All Steps Overview and Execution Batches (always)
  - `architecture-plan.yaml` (only for Mode B complex projects)
- Update session Status: `PLANNING` → `READY`

## MUST NOT

- Implement planning logic itself (all planning stays in `development-planner`)
- Change the semantics or constraints of `development-planner`
- Generate prompts or step breakdowns directly
- Modify source code
- Skip calling `development-planner` agent

## Process

### Step 1: Load Session

1. Use `session-manager` to load or verify current session
2. Confirm session exists and has Status that allows planning

### Step 2: Verify Prerequisites

1. Use `artifact-fs-manager` to check `context-analysis.md` exists
2. Use `artifact-fs-manager` to check `specification.md` exists
3. If either missing: Report error, suggest running `/init`

### Step 3: Read Artifacts

1. Read `context-analysis.md` content
2. Read `specification.md` content
3. Prepare input payload for `development-planner`

### Step 4: Invoke development-planner

1. Call `development-planner` agent with:
   - Session path
   - Context analysis content
   - Specification content
2. Wait for agent to complete
3. Receive:
   - `plan.md` content
   - `session.md` content (with steps and batches)
   - `architecture-plan.yaml` content (optional, Mode B only)

### Step 5: Persist Artifacts

1. Use `artifact-fs-manager` to write `plan.md`
2. Use `artifact-fs-manager` to write `session.md`
3. If Mode B: Use `artifact-fs-manager` to write `architecture-plan.yaml`

### Step 6: Update Session Status

1. Use `session-manager` to update Status to `READY`
2. Update `Last Updated` timestamp

## Output

Returns:

```
{
  "success": true,
  "sessionPath": ".claude/dev-sessions/...",
  "mode": "A" | "B",
  "status": "READY",
  "artifacts": {
    "plan": "{sessionPath}/plan.md",
    "session": "{sessionPath}/session.md",
    "architecture": "{sessionPath}/architecture-plan.yaml" | null
  },
  "stepsCount": 5,
  "batchesCount": 2
}
```

## Error Handling

- **Prerequisites missing**: List missing artifacts, suggest `/init`
- **development-planner fails**: Report error, keep Status as PLANNING
- **Invalid plan structure**: Report validation errors from planner
- **Write fails**: Report which artifact couldn't be saved
