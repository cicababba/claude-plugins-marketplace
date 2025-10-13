End the current development session with comprehensive documentation:

## 1. Load Active Session

- Check `.claude/dev-sessions/.current-session` for the active session
- If `.current-session` is empty or missing:
  - Read the status of all sessions in `.claude/dev-sessions/` (If no sessions found, inform user there's nothing to end)
  - Filter sessions to show only those with status `PLANNING` or `IN_PROGRESS`
  - Prompt the user to select one from the filtered list

## 2. Update Session Status

- If all steps COMPLETED: Status = COMPLETED
- If any steps FAILED: Status = FAILED
- If steps remain NOT_STARTED: Status = PARTIALLY_COMPLETED
- Add "End Time" to Session Overview
- Calculate and update "Duration"

## 3. Generate Summary Based on Session Status

### If session status is PLANNING:

- Update "Last Updated" timestamp in Session Overview
- Calculate and update "Duration"
- Update "Status" in READY
- IMPORTANT Do not update anything else
- Suggest /wf-dev:run

### If session status is IN_PROGRESS:

- Ask: "Commit completed steps now? (Y/N)"
- If Y: run git-commit-manager (pass `plan.md` + `session.md`)
- If N: Go to the next question
- Ask: "Do you want to generate the doc of what is done until now? (Y/N)"
- If N: Go to the next steps
- If Y: run docs-generator (pass `plan.md` + `session.md` and note: uncommitted changes) with these specifics:

#### Implementation Summary

- Update the Todo Progress section with final status
- Key accomplishments and features implemented
- All completed steps with their outcomes
- Problems encountered and solutions applied
- Breaking changes or important findings

#### Technical Details

- Dependencies added/removed
- Configuration changes made
- Database migrations or schema changes (if any)
- API changes or new endpoints (if any)

#### Session Closure

- What wasn't completed and why
- Lessons learned during implementation
- Tips for future developers
- Recommendations for next steps

### If session status is COMPLETED

- Ask: "Commit completed changes? (Y/N)"
  - If Y: pass `plan.md` and `session.md` to the **git-commit-manager** agent.
- Ask: "Generate final documentation? (Y/N)"
  - If Y: pass `plan.md` and `session.md` to the **docs-generator** agent.  
    - The generated documentation must be saved in the path found in the file `.claude/dev-sessions/.current-session`.
- Ask: "Run a code quality check for this dev session? (Y/N)"
  - If Y: run `wf-quality-check` passing as argument the list of modified file paths in the session.
  - If N: continue as usual.


## 4. Close Session

- Clear contents of `.claude/dev-sessions/.current-session` (don't remove file)
- Inform user the session has been documented
