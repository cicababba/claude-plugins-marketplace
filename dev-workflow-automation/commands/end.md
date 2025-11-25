# /end — Finalize Development Session

End the current development session with comprehensive documentation and proper closure.

**ABSOLUTE SCOPE & RESTRICTIONS**

- This command finalizes and closes a development session
- It computes final Status based on step completion
- It optionally triggers commits, docs, and quality checks
- It clears `.current-session` to mark session as closed

---

## 1. Load Active Session

> **Skill:** Use `session-manager` to load the session

- Check `.claude/dev-sessions/.current-session`
- If empty or missing:
  - Filter sessions by Status: `PLANNING` or `IN_PROGRESS`
  - If no sessions found: inform user there's nothing to end
  - Present filtered list to user for selection

---

## 2. Handle Based on Session Status

### If Status is PLANNING

> **Skill:** Use `session-manager` to update timestamps

- Update `Last Updated` timestamp
- Calculate and update `Duration`
- Set Status to `READY`
- **Do NOT** run commits, docs, or quality checks
- Suggest `/run` to start execution
- Clear `.current-session`

### If Status is IN_PROGRESS, READY, or PARTIALLY_COMPLETED

> **Skill:** Use `dev-session-closure` for full closure

**Step 2a: Ask User Preferences**

- Ask: "Commit completed changes? (Y/N)"
- Ask: "Generate documentation? (Y/N)"
- Ask: "Run code quality check? (Y/N)"

**Step 2b: Invoke Closure**

Invoke `dev-session-closure` with:

- Session path
- User preferences:
  - `triggerCommit`: User's commit choice
  - `triggerDocs`: User's docs choice
  - `triggerQuality`: User's quality choice
  - `modifiedFiles`: List of files touched in session (from step completion tickets)

The skill will:

- Compute final Status:
  - All steps COMPLETED → `COMPLETED`
  - Any steps FAILED → `FAILED`
  - Some NOT_STARTED → `PARTIALLY_COMPLETED`
- Update `session.md`:
  - Final Status
  - End Time
  - Duration
  - Last Updated
- If commit requested: Use `git-and-docs` skill
- If docs requested: Use `git-and-docs` skill
- If quality requested: Use `dev-quality-report` skill
- Clear `.current-session`

---

## 3. Documentation Sections (when generated)

When docs are generated via `git-and-docs`, include:

### Implementation Summary

- Key accomplishments and features implemented
- All completed steps with their outcomes
- Problems encountered and solutions applied
- Breaking changes or important findings

### Technical Details

- Dependencies added/removed
- Configuration changes made
- Database migrations or schema changes (if any)
- API changes or new endpoints (if any)

### Session Closure

- What wasn't completed and why
- Lessons learned during implementation
- Tips for future developers
- Recommendations for next steps

---

## 4. Confirm Closure

- Inform user the session has been documented
- Report final Status
- Report Duration
- List any generated artifacts:
  - Commit hash (if committed)
  - Documentation path (if generated)
  - Quality report path (if generated)

---

## Error Handling

- **No sessions to end**: Inform user, nothing to close
- **Session already COMPLETED**: Report that session is already closed
- **Commit fails**: Report error, continue with other closures
- **Docs generation fails**: Report error, continue with closure
- **Quality check fails**: Report error, continue with closure
