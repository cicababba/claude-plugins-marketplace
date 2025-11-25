---
name: dev-specification-qa
description: Handle specification gathering with one question at a time approach. Use when collecting requirements, constraints, and acceptance criteria from users. Builds structured specification.md through iterative Q&A, never assuming, always clarifying.
---

# Dev Specification Q&A

This skill handles the specification gathering phase through iterative questioning.

## When to Use

- After context analysis is complete in `/init`
- When gathering requirements for a new feature
- When clarifying constraints and acceptance criteria
- Before any planning or implementation begins

## Input

- `sessionPath`: Path to the current session directory
- `contextAnalysis`: Content or reference to `context-analysis.md`
- `taskDescription`: Initial task description from user
- `partialSpec` (optional): Existing partial specification to continue from

## Responsibilities

- Drive an interactive Q&A loop
- Ask exactly ONE high-impact question at a time
- Ask follow-ups for unclear answers (max 4 sequential follow-ups)
- Prioritize blocking unknowns (requirements, contracts, constraints)
- Build a structured Specification including:
  - Overview
  - Functional requirements
  - Non-functional requirements
  - Constraints
  - Integration points
  - Error handling & edge cases
  - Test decision: `testsRequired: true | false | "unclear"`
- Present a Quick Summary for user confirmation
- Write `specification.md` via `artifact-fs-manager` only after confirmation

## MUST NOT

- Generate any "Plan", "Roadmap", or implementation tasks
- Generate or change code
- Make assumptions without clarification
- Ask multiple questions at once
- Proceed to planning or implementation
- Skip user confirmation before saving specification

## Process

### Step 1: Initialize Q&A

1. Review `contextAnalysis` to understand existing codebase
2. Review `taskDescription` to understand the goal
3. Identify the most critical unknown that blocks progress
4. Formulate ONE targeted question

### Step 2: Q&A Loop

For each question cycle:

1. **Identify highest-impact unknown**:

   - What requirement is most unclear?
   - What constraint could change the approach?
   - What integration point needs clarification?

2. **Ask ONE question**:

   - Be specific and targeted
   - Provide context why this question matters
   - Offer examples if helpful

3. **Process answer**:

   - If clear: Record and move to next unknown
   - If unclear: Ask follow-up (max 4 follow-ups per topic)
   - If contradictory: Surface conflict and ask for prioritization

4. **Repeat until**:
   - All critical unknowns are resolved
   - User indicates specification is complete
   - Sufficient detail exists for planning

### Step 3: Test Decision

After main requirements are gathered:

1. Ask: "Should tests be written for this feature?"
2. Record decision as:
   - `testsRequired: true` - Tests required
   - `testsRequired: false` - No tests needed
   - `testsRequired: "unclear"` - User undecided

### Step 4: Present Quick Summary

1. Generate a concise summary of gathered requirements
2. Present to user for confirmation
3. Ask: "Is this specification correct and complete?"
4. If user requests changes: Return to Q&A loop
5. If user confirms: Proceed to save

### Step 5: Generate specification.md

Structure the output as:

```markdown
# Specification

## Overview

[Brief description of what needs to be built]

## Task Description

[Original task from user]

## Functional Requirements

1. [Requirement 1]
2. [Requirement 2] ...

## Non-Functional Requirements

- Performance: [constraints]
- Security: [constraints]
- Scalability: [constraints]

## Constraints

- [Technical constraints]
- [Business constraints]
- [Timeline constraints]

## Integration Points

- [API/service 1]
- [API/service 2]

## Error Handling & Edge Cases

- [Edge case 1]: [How to handle]
- [Edge case 2]: [How to handle]

## Acceptance Criteria

1. [Criterion 1]
2. [Criterion 2]

## Test Decision

testsRequired: [true | false | "unclear"] [Rationale for test decision]

## Additional Notes

[Any other relevant information gathered during Q&A]
```

### Step 6: Save Artifact

1. Use `artifact-fs-manager` to write `specification.md`
2. Return confirmation to caller

## Output

Returns:

```
{
  "success": true,
  "sessionPath": ".claude/dev-sessions/...",
  "testsRequired": true | false | "unclear",
  "summary": "Brief summary of specification",
  "artifactPath": "{sessionPath}/specification.md",
  "questionsAsked": 5,
  "confirmed": true
}
```

## Error Handling

- **User abandons Q&A**: Save partial spec with `Status: INCOMPLETE`
- **Contradictory requirements**: Surface conflicts, ask for prioritization
- **Unclear answers**: Allow up to 4 follow-up questions, then note as "needs clarification"
- **Context analysis missing**: Request context analysis be run first
