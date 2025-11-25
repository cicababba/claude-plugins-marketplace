---
name: dev-execution-orchestrator
description: Execute steps from plan.md and session.md. Use during /run to dispatch steps to sub-agents, update progress, and manage execution batches. Never sets Status to COMPLETED.
---

# Dev Execution Orchestrator

This skill orchestrates the execution of development steps defined in the plan.

## When to Use

- During `/run` command execution
- When executing one or more planned steps
- When updating step progress and batch status

## Input

- `sessionPath`: Path to the current session (or read from `.current-session`)
- `mode`: Execution mode:
  - `next-step`: Execute single next available step
  - `full-batch`: Execute all steps in next available batch
  - `specific-step`: Execute a specific step by ID (if dependencies satisfied)
- `stepId` (optional): Specific step ID for `specific-step` mode

## Responsibilities

- Load session via `session-manager`
- Read `plan.md` and `session.md` via `artifact-fs-manager`
- Verify plan integrity:
  - Plan exists and contains Generated Prompts
  - Plan matches Session "All Steps Overview"
  - No circular dependencies between steps
- Determine executable steps:
  - Status = `NOT_STARTED`
  - All dependencies have Status = `COMPLETED`
- Dispatch execution to appropriate sub-agents using pre-generated prompts:
  - `react-frontend-dev`
  - `angular-frontend-dev`
  - `nodejs-backend-dev`
  - `java-backend-developer`
  - `react-test-writer`
  - `backend-test-writer`
  - `database-architect`
- Update after each step:
  - Step status in `session.md`
  - Batch status in Execution Batches
  - Progress counter (X/Y steps completed)
- Keep global Status = `IN_PROGRESS`

## MUST

- Set session Status to `IN_PROGRESS` at start of execution
- Update step status to `IN_PROGRESS` before dispatching
- Update step status to `COMPLETED` or `FAILED` after completion
- Update progress after each step
- NEVER set global session Status to `COMPLETED`

## MUST NOT

- Regenerate or alter the plan structure
- Modify source code directly (sub-agents do this)
- Create new steps not in the plan
- Skip dependency validation
- Auto-complete the session

## Process

### Step 1: Load Session and Artifacts

1. Use `session-manager` to load current session
2. Use `artifact-fs-manager` to read `plan.md`
3. Use `artifact-fs-manager` to read `session.md`

### Step 2: Validate Plan Integrity

1. Check `plan.md` has `# Generated Prompts` section
2. Parse steps from `# Step-by-Step Implementation`
3. Match against `session.md` "All Steps Overview"
4. Build dependency graph
5. Detect circular dependencies using DFS
6. Report any validation errors

### Step 3: Determine Executable Steps

Based on execution mode:

**next-step mode:**

1. Find first step where:
   - Status = `NOT_STARTED`
   - All dependencies have Status = `COMPLETED`
2. Return single step or report no executable steps

**full-batch mode:**

1. Identify current batch from Execution Batches
2. Find all steps in batch where:
   - Status = `NOT_STARTED`
   - All dependencies satisfied
3. Return list of executable steps

**specific-step mode:**

1. Find step by ID
2. Verify Status = `NOT_STARTED`
3. Verify all dependencies satisfied
4. Return step or report why not executable

### Step 4: Update Session Status

1. Use `session-manager` to set Status = `IN_PROGRESS` (if not already)
2. Update `Last Updated`

### Step 5: Execute Steps

For each executable step:

1. **Mark as IN_PROGRESS**:

   - Update step status in `session.md`

2. **Get execution prompt**:

   - Find matching prompt in `# Generated Prompts` section
   - Extract: Context, Task, Requirements, Dependencies, Validation, Project Context

3. **Dispatch to sub-agent**:

   - Identify agent from step's `Subagent ID`
   - For `main-agent-sequential`: Execute directly
   - For `subagent-parallel`: Call appropriate agent

4. **Process result**:

   - Parse JSON completion ticket from agent
   - Verify `status` field

5. **Update step status**:

   - If success: Set Status = `COMPLETED`
   - If failure: Set Status = `FAILED`, record reason

6. **Update progress**:

   - Increment completed count
   - Update `Progress: X/Y steps completed`

7. **Update batch status**:
   - Check if all steps in batch completed
   - Update batch status in Execution Batches

### Step 6: Check for More Work

1. After batch completes, check for next batch
2. If no more executable steps:
   - Show summary of completed steps
   - Keep Status as `IN_PROGRESS`
   - Report what steps remain blocked

## Output

Returns:

```
{
  "success": true,
  "sessionPath": ".claude/dev-sessions/...",
  "status": "IN_PROGRESS",
  "executed": [
    {
      "stepId": "step-001",
      "agent": "react-frontend-dev",
      "status": "COMPLETED",
      "filesTouched": ["src/components/Auth.tsx"]
    }
  ],
  "progress": "3/5 steps completed",
  "nextExecutable": ["step-004"],
  "blocked": [
    {
      "stepId": "step-005",
      "reason": "Waiting for step-003"
    }
  ]
}
```

## Error Handling

- **Plan not found**: Suggest running `dev-plan-orchestrator`
- **Circular dependencies**: Report cycle, do not execute
- **Step execution fails**: Mark as FAILED, assess impact on dependents
- **Sub-agent unavailable**: Report error, keep step as NOT_STARTED
- **No executable steps**: Report current state, list blocked steps with reasons
