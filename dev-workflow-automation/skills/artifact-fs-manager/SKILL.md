---
name: artifact-fs-manager
description: Unified file handler for dev session artifacts. Use when reading or writing context-analysis.md, specification.md, plan.md, session.md, architecture-plan.yaml, or quality reports. Handles file persistence without modifying semantic content.
---

# Artifact FS Manager

This skill provides unified file operations for all development session artifacts.

## When to Use

- When reading or writing `context-analysis.md`
- When reading or writing `specification.md`
- When reading or writing `plan.md`
- When reading or writing `session.md` (content sections, not metadata)
- When reading or writing `architecture-plan.yaml`
- When reading or writing quality reports under `.claude/dev-sessions/quality-reports/`
- When generating quality report filenames

## Responsibilities

- Read artifact files from session directory
- Write artifact files to session directory
- Generate quality report filenames with format: `YYYYMMDD-HHmm-qa-<reportId>.md`
- Generate 8-character lowercase hex tokens for report IDs
- Ensure `.claude/dev-sessions/quality-reports/` directory exists

## Supported Artifacts

| Artifact | Path | Purpose |
|----------|------|---------|
| `context-analysis.md` | `{sessionPath}/context-analysis.md` | Code analysis findings |
| `specification.md` | `{sessionPath}/specification.md` | Complete requirements |
| `plan.md` | `{sessionPath}/plan.md` | Implementation plan |
| `session.md` | `{sessionPath}/session.md` | Progress and metadata |
| `architecture-plan.yaml` | `{sessionPath}/architecture-plan.yaml` | Complex project architecture |
| Quality Report | `.claude/dev-sessions/quality-reports/YYYYMMDD-HHmm-qa-{id}.md` | Quality analysis |

## MUST NOT

- Change semantic content of artifacts (just persist strings as provided)
- Modify source code files or project configuration
- Touch `.current-session` (owned by `session-manager`)
- Interpret or transform artifact content
- Generate new content (only persist what is provided)

## Process

### Reading an Artifact

1. Receive artifact type and session path
2. Construct full file path
3. Read file content
4. Return content as string (or null if not found)

### Writing an Artifact

1. Receive artifact type, session path, and content
2. Construct full file path
3. Ensure parent directory exists
4. Write content to file
5. Return success/failure status

### Generating Quality Report Path

1. Generate timestamp: `YYYYMMDD-HHmm`
2. Generate reportId: 8-char lowercase hex (e.g., `a3c9d1b2`)
3. Ensure `.claude/dev-sessions/quality-reports/` exists
4. Return path: `.claude/dev-sessions/quality-reports/{timestamp}-qa-{reportId}.md`

### Checking Artifact Existence

1. Receive artifact type and session path
2. Construct full file path
3. Check if file exists and is readable
4. Return boolean

## Output

### Read Operation
```
{
  "success": true,
  "content": "file content as string",
  "path": "full/path/to/file.md"
}
```

### Write Operation
```
{
  "success": true,
  "path": "full/path/to/file.md",
  "bytesWritten": 1234
}
```

### Generate Report Path
```
{
  "reportId": "a3c9d1b2",
  "path": ".claude/dev-sessions/quality-reports/20250125-1430-qa-a3c9d1b2.md"
}
```

### Check Existence
```
{
  "exists": true,
  "path": "full/path/to/file.md",
  "readable": true
}
```

## Error Handling

- **File not found**: Return `success: false` with clear message
- **Permission denied**: Report permission error with path
- **Directory not found**: Create directory if writing, report error if reading
- **Invalid artifact type**: List valid artifact types
- **Invalid session path**: Report that session path doesn't exist
