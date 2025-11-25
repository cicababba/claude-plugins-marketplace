---
name: session-manager
description: Single source of truth for dev session lifecycle. Use when creating, loading, updating, or closing development sessions under .claude/dev-sessions/. Handles session directory creation, .current-session tracking, and session.md metadata updates (Status, timestamps, Duration).
---

# Session Manager

This skill is the single source of truth for development session lifecycle management.

## When to Use

- When creating a new development session
- When loading an existing session (by path or by filtering Status)
- When updating session metadata (Status, Last Updated, End Time, Duration)
- When closing a session and clearing `.current-session`
- At the start of `/init`, `/run`, or `/end` commands

## Responsibilities

- Create session directory: `.claude/dev-sessions/YYYY-MM-DD-HHMM-${slug}/`
- Initialize/update `session.md` headers (Session Overview section only)
- Read and update `.claude/dev-sessions/.current-session`
- Load existing sessions (with optional Status filtering)
- Update in `session.md`:
  - `Status`: PLANNING, READY, IN_PROGRESS, COMPLETED, PARTIALLY_COMPLETED, FAILED
  - `Start Time`: When session is created
  - `Last Updated`: On any modification
  - `End Time`: When session is closed
  - `Duration`: Calculated from Start Time to End Time
- Close session and clear `.current-session` content (file remains)

## MUST NOT

- Generate context analysis, specification, plan, documentation, or code
- Modify any source code files
- Read or write `plan.md` content (only track its existence)
- Execute development steps
- Trigger sub-agents

## Process

### Creating a New Session

1. Generate slug from task description (lowercase, hyphens, max 50 chars)
2. Generate timestamp: `YYYY-MM-DD-HHMM`
3. Create directory: `.claude/dev-sessions/{timestamp}-{slug}/`
4. Initialize `session.md` with Session Overview:

   ```markdown
   ## Session Overview

   **Start Time:** [YYYY-MM-DD HH:MM AM/PM] **Last Updated:** [YYYY-MM-DD HH:MM AM/PM] **Status:** PLANNING **Progress:** 0/0 steps completed **Task:** [task description] **Duration:** **End Time:**
   ```

5. Write session path to `.claude/dev-sessions/.current-session`

### Loading an Existing Session

1. Read `.claude/dev-sessions/.current-session`
2. If file contains a valid path:
   - Verify the session directory exists
   - Return the session path
3. If file is empty or path invalid:
   - Scan `.claude/dev-sessions/` for session directories
   - For each directory, read `session.md` and extract Status
   - Filter by requested status (e.g., READY, PLANNING, IN_PROGRESS)
   - Present filtered list to user for selection
   - Update `.current-session` with selected path
   - Return the session path

### Updating Session Metadata

1. Read current `session.md` content
2. Update requested fields in Session Overview:
   - `Status`: New status value
   - `Last Updated`: Current timestamp
   - `Progress`: X/Y steps completed (if provided)
   - `End Time`: Current timestamp (if closing)
   - `Duration`: Calculate from Start Time to End Time
3. Write updated `session.md`

### Closing a Session

1. Update `session.md` with:
   - Final `Status`
   - `End Time`: Current timestamp
   - `Duration`: Calculated value
   - `Last Updated`: Current timestamp
2. Clear contents of `.claude/dev-sessions/.current-session` (keep file)

## Output

Returns session information:

```
{
  "sessionPath": ".claude/dev-sessions/YYYY-MM-DD-HHMM-slug",
  "status": "PLANNING|READY|IN_PROGRESS|COMPLETED|PARTIALLY_COMPLETED|FAILED",
  "startTime": "YYYY-MM-DD HH:MM AM/PM",
  "lastUpdated": "YYYY-MM-DD HH:MM AM/PM",
  "endTime": "YYYY-MM-DD HH:MM AM/PM" | null,
  "duration": "Xh Ym" | null,
  "task": "task description"
}
```

## Error Handling

- **Directory creation fails**: Report permission error, suggest alternative location
- **Session not found**: List available sessions or suggest `/init`
- **Invalid session structure**: Report missing files, suggest re-initialization
- **Slug collision**: Append `-2`, `-3`, etc. to make unique
- **No sessions match filter**: Report no sessions found with requested status
