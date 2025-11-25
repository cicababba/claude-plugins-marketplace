---
name: dev-session-closure
description: Close development sessions with proper status computation, timestamps, and optional triggers for commits, docs, and quality checks. Use during /end to finalize sessions and clear .current-session.
---

# Dev Session Closure

This skill handles the proper closure of development sessions.

## When to Use

- During `/end` command
- When finalizing a development session
- When computing final session status
- When clearing `.current-session`

## Input

- `sessionPath`: Path to the current session (or read from `.current-session`)
- `options`:
  - `triggerCommit`: boolean - Whether to trigger git commits
  - `triggerDocs`: boolean - Whether to trigger documentation generation
  - `triggerQuality`: boolean - Whether to trigger quality check
  - `modifiedFiles`: string[] - List of files modified in session (for quality check)

## Responsibilities

- Load session via `session-manager`
- Read `session.md` (and `plan.md` if needed) via `artifact-fs-manager`
- Compute final Status based on step completion:
  - All steps `COMPLETED` → `COMPLETED`
  - Some steps `FAILED` → `FAILED`
  - Some `COMPLETED` and some `NOT_STARTED` → `PARTIALLY_COMPLETED`
  - No plan exists (planning phase only) → `READY`
- Update `session.md`:
  - `Status`: Final computed status
  - `End Time`: Current timestamp
  - `Duration`: Calculated from Start Time
  - `Last Updated`: Current timestamp
- Optionally trigger (based on flags):
  - Git commits via `git-and-docs` skill
  - Documentation generation via `git-and-docs` skill
  - Quality report via `dev-quality-report` skill
- Clear `.current-session` content via `session-manager`

## MUST NOT

- Modify `plan.md` content
- Execute development steps
- Re-run planning
- Modify source code

## Process

### Step 1: Load Session

1. Use `session-manager` to load current session
2. Verify session exists and is in a closeable state

### Step 2: Read Session State

1. Use `artifact-fs-manager` to read `session.md`
2. Check if `plan.md` exists
3. Extract current Status
4. Extract all step statuses from "All Steps Overview"

### Step 3: Handle Based on Current Status

**If Status is PLANNING:**

1. No steps to evaluate
2. Set Status to `READY`
3. Update timestamps only
4. Do NOT trigger commits, docs, or quality check
5. Suggest `/run` to execute

**If Status is IN_PROGRESS, READY, or other:**

1. Evaluate step completion (Step 4)
2. Ask user for closure options (Step 5)
3. Execute closures (Step 6)

### Step 4: Compute Final Status

Parse "All Steps Overview" and count:

- `completedCount`: Steps with Status = `COMPLETED`
- `failedCount`: Steps with Status = `FAILED`
- `notStartedCount`: Steps with Status = `NOT_STARTED`
- `totalCount`: Total steps

Determine final status:

```
if (failedCount > 0):
  finalStatus = "FAILED"
elif (notStartedCount > 0 and completedCount > 0):
  finalStatus = "PARTIALLY_COMPLETED"
elif (completedCount == totalCount and totalCount > 0):
  finalStatus = "COMPLETED"
elif (totalCount == 0):
  finalStatus = "READY"
else:
  finalStatus = "PARTIALLY_COMPLETED"
```

### Step 5: Process Optional Triggers

If `triggerCommit` is true:

1. Use `git-and-docs` skill with commit request
2. Pass `plan.md` and `session.md` paths
3. Record commit result

If `triggerDocs` is true:

1. Use `git-and-docs` skill with docs request
2. Specify output path = session directory
3. Record docs path

If `triggerQuality` is true:

1. Use `dev-quality-report` skill
2. Pass `modifiedFiles` as paths to analyze
3. Record report path

### Step 6: Update Session Metadata

1. Use `session-manager` to update:
   - `Status`: Final computed status
   - `End Time`: Current timestamp
   - `Duration`: Calculate from Start Time to now
   - `Last Updated`: Current timestamp

### Step 7: Close Session

1. Use `session-manager` to clear `.current-session`
2. Confirm session is documented and closed

## Output

Returns:

```
{
  "success": true,
  "sessionPath": ".claude/dev-sessions/...",
  "finalStatus": "COMPLETED" | "PARTIALLY_COMPLETED" | "FAILED" | "READY",
  "summary": {
    "completed": 4,
    "failed": 0,
    "notStarted": 1,
    "total": 5
  },
  "duration": "2h 15m",
  "triggers": {
    "commit": {
      "executed": true,
      "result": "Committed with message: feat(auth): ..."
    },
    "docs": {
      "executed": true,
      "path": "{sessionPath}/documentation.md"
    },
    "quality": {
      "executed": false,
      "path": null
    }
  },
  "sessionClosed": true
}
```

## Error Handling

- **Session not found**: Report error, nothing to close
- **Invalid session state**: Report current state, ask for clarification
- **Commit fails**: Report error, continue with other closures
- **Docs generation fails**: Report error, continue with closure
- **Quality check fails**: Report error, continue with closure
- **Status update fails**: Report error, session may be partially closed
