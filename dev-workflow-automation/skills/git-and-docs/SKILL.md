---
name: git-and-docs
description: Coordinate git commits and documentation generation. Use when commits or docs are requested during session closure. Wraps git-commit-manager and docs-generator agents without changing session status.
---

# Git and Docs

This skill coordinates git commit creation and documentation generation.

## When to Use

- During `/end` when user requests commits
- During `/end` when user requests documentation
- When explicitly asked to commit session work
- When explicitly asked to generate documentation

## Input

- `sessionPath`: Path to the current session directory
- `request`:
  - `commit`: boolean - Whether to create commits
  - `docs`: boolean - Whether to generate documentation
- `commitOptions` (optional):
  - `message`: Custom commit message (optional)
  - `files`: Specific files to commit (optional, default: all staged)
- `docsOptions` (optional):
  - `outputPath`: Where to save docs (default: session directory)
  - `sections`: Which sections to generate

## Responsibilities

- Read `plan.md` and `session.md` via `artifact-fs-manager`
- If commits requested:
  - Call `git-commit-manager` agent
  - Pass plan and session context
  - Return commit result
- If docs requested:
  - Call `docs-generator` agent
  - Specify output path
  - Return generated docs path
- Return combined results

## MUST NOT

- Change session Status (owned by `session-manager` and `dev-session-closure`)
- Execute development steps
- Modify plan or session content
- Make decisions about what to commit (user decides)

## Process

### Step 1: Load Context

1. Use `artifact-fs-manager` to read `plan.md`
2. Use `artifact-fs-manager` to read `session.md`
3. Extract relevant context for commit/docs

### Step 2: Handle Commit Request

If `commit: true`:

1. Prepare context for `git-commit-manager`:

   - Task description from session
   - Completed steps from session
   - Files modified (from step completion tickets)

2. Call `git-commit-manager` agent with:

   - Session context (plan.md + session.md paths)
   - Custom message if provided
   - File list if provided

3. Receive commit result:
   - Commit hash
   - Commit message
   - Files committed

### Step 3: Handle Docs Request

If `docs: true`:

1. Determine output path:

   - Use provided `outputPath`
   - Or default to `{sessionPath}/documentation.md`

2. Prepare context for `docs-generator`:

   - plan.md content
   - session.md content
   - Note any uncommitted changes

3. Call `docs-generator` agent with:

   - Session context
   - Output path
   - Sections to generate:
     - Implementation Summary
     - Technical Details
     - Session Closure notes

4. Receive docs result:
   - Generated file path
   - Sections included

### Step 4: Return Combined Results

Aggregate results from both operations.

## Output

Returns:

```
{
  "success": true,
  "sessionPath": ".claude/dev-sessions/...",
  "commit": {
    "requested": true,
    "executed": true,
    "result": {
      "hash": "abc1234",
      "message": "feat(auth): implement login component\n\n...",
      "filesCommitted": ["src/components/Login.tsx", "src/hooks/useAuth.ts"]
    }
  },
  "docs": {
    "requested": true,
    "executed": true,
    "result": {
      "path": "{sessionPath}/documentation.md",
      "sections": ["Implementation Summary", "Technical Details", "Session Closure"]
    }
  }
}
```

## Generated Documentation Structure

When docs are generated, they follow this structure:

```markdown
# Session Documentation

## Implementation Summary

- Task: {task description}
- Status: {final status}
- Duration: {duration}

### Key Accomplishments

- {completed feature 1}
- {completed feature 2}

### Completed Steps

1. {step-001}: {description} - COMPLETED
2. {step-002}: {description} - COMPLETED ...

### Problems Encountered

- {problem 1}: {solution applied}

### Breaking Changes

- {if any}

## Technical Details

### Dependencies Added/Removed

- Added: {list}
- Removed: {list}

### Configuration Changes

- {config change 1}

### Database Changes

- {if any}

### API Changes

- {if any}

## Session Closure

### What Wasn't Completed

- {step-00X}: {reason}

### Lessons Learned

- {insight 1}

### Recommendations for Next Steps

- {recommendation 1}
```

## Error Handling

- **plan.md not found**: Report error, cannot generate meaningful commit/docs
- **session.md not found**: Report error, cannot generate meaningful commit/docs
- **git-commit-manager fails**: Report error, return partial result
- **docs-generator fails**: Report error, return partial result
- **No changes to commit**: Report that working tree is clean
- **Output path not writable**: Report permission error
