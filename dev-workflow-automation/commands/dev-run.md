Execute the next available steps from the active development session:

## 1. Load Active Session

- Read `.claude/dev-sessions/.current-session` to get the active session path
- If `.current-session` is empty or missing:
  - Read the status of all sessions in `.claude/dev-sessions/`
  - Filter sessions to show only those with status `READY` or `PLANNING`
  - Prompt the user to select one from the filtered list
  - Update `.claude/dev-sessions/.current-session` with the user's selection
- Load `session.md` from the selected session path and access the Todo Progress section
- Show current session status and available steps to execute

## 2. Validate Session State

- Verify all session files exist and are readable
- Check plan.md has generated prompts
- Validate no circular dependencies

## 3. Update Session Overview

- Update "Last Updated"
- Update "Status" to IN_PROGRESS
- Never change global session status to COMPLETED

## 4. Execute Available Steps

- Execute steps using pre-generated prompts from `plan.md`

## 5. Update Progress & Continue

- After each step completion:
  - IMPORTANT: Update step and batch statues in session.md, not only the step but also the Execution Batches
  - Check for newly unblocked steps
  - If parallel batch complete, move to next batch
  - If no more executable steps:
    - DO NOT change global session status
    - Show summary of completed steps
    - Keep status as IN_PROGRESS

## Error Handling

- No active session: Prompt user to run /init first
- Session files missing: Attempt recovery or suggest re-initialization
- Step execution fails:
  - Mark as FAILED with reason
  - Check if failure blocks other steps
  - Suggest remediation
- Dependency conflict: Report which steps are blocked
