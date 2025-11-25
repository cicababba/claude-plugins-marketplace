---
name: dev-context-analysis
description: Perform context analysis phase for development sessions. Use when analyzing code paths to understand existing patterns, dependencies, tests, and implementation details. Generates context-analysis.md with structured per-path findings.
---

# Dev Context Analysis

This skill performs the context analysis phase of a development session.

## When to Use

- At the start of `/init` after session creation
- When the user provides paths to analyze
- When you need to understand existing code patterns before specification gathering

## Input

- `sessionPath`: Path to the current session directory
- `paths`: List of file or folder paths to analyze (e.g., `/src/auth`, `/src/models/user.ts`)

## Responsibilities

- For each provided path, analyze:
  - **Existing Patterns**: Code conventions, architectural patterns, naming conventions
  - **Dependencies & Imports**: External packages, internal modules, dependency graph
  - **Related Tests**: Test files, test patterns, coverage areas
  - **Current Implementation Details**: Key functions, classes, data structures
- Build a structured per-path analysis
- Generate `context-analysis.md` with organized findings
- Save via `artifact-fs-manager` skill

## MUST NOT

- Propose implementation steps or planning
- Generate a "Plan", "Roadmap", or "Implementation" section
- Modify any source code files
- Make assumptions about what should be changed
- Skip paths or provide incomplete analysis

## Process

### Step 1: Validate Paths

1. For each provided path:
   - Verify the path exists
   - Determine if it's a file or directory
   - Report any invalid paths to the user

### Step 2: Analyze Each Path

For each valid path, spawn analysis (can be parallelized):

1. **Scan for patterns**:

   - Code style conventions
   - Architectural patterns (MVC, Clean Architecture, etc.)
   - Naming conventions (camelCase, PascalCase, etc.)
   - Common utilities and helpers used

2. **Extract dependencies**:

   - External package imports
   - Internal module imports
   - Circular dependency detection
   - Version constraints if visible

3. **Find related tests**:

   - Corresponding test files (`*.test.ts`, `*.spec.ts`)
   - Test patterns used (unit, integration, e2e)
   - Mocking strategies
   - Coverage gaps

4. **Document implementation**:
   - Key classes and functions
   - Public APIs
   - Data models and types
   - State management approach

### Step 3: Generate context-analysis.md

Structure the output as:

```markdown
# Context Analysis

## Quick Summary

[2-3 sentence overview of all analyzed paths]

## Path: /path/to/first

### Existing Patterns

- [Pattern findings]

### Dependencies & Imports

- [Dependency findings]

### Related Tests

- [Test findings]

### Current Implementation Details

- [Implementation findings]

---

## Path: /path/to/second

### Existing Patterns

- [Pattern findings]

...
```

### Step 4: Save Artifact

1. Use `artifact-fs-manager` to write `context-analysis.md`
2. Return the analysis to the caller

## Output

Returns:

```
{
  "success": true,
  "sessionPath": ".claude/dev-sessions/...",
  "analyzedPaths": ["/path/one", "/path/two"],
  "summary": "Brief summary of findings",
  "artifactPath": "{sessionPath}/context-analysis.md"
}
```

## Error Handling

- **Path not found**: Report invalid path, ask user to verify
- **Permission denied**: Report which paths are unreadable
- **Empty directory**: Note that directory contains no analyzable files
- **Analysis timeout**: Report partial results, note incomplete paths
- **No paths provided**: Ask user which paths to analyze
