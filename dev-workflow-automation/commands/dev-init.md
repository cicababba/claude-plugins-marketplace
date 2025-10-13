Initialize a structured development session for any development task (new feature, bug fix, refactoring, performance optimization, etc.):
**IMPORTANT**: This command must never produce code snippets or implementations, only analysis and specifications.
**IMPORTANT**: All generated files (context-analysis.md, specification.md, session.md, etc.) must ALWAYS be written in ENGLISH, regardless of the language used in the conversation.

## 1. Identify the relevant context

- Ask the user which files or folders you should analyze to understand the relevant context. Give examples (e.g., /src/auth, /src/models/user.ts).
- When the user provides multiple paths, create one subagent per path.
- Each subagent must scan only its assigned path, and extract:
  - Existing code patterns in that area
  - Dependencies and imports
  - Related tests (if any)
  - Current implementation details
- After scanning all paths, aggregate all findings from the subagents and organize them by analyzed path.
- Create and save in this step a structured analysis (separate findings by path), this will be reused later, output it to the user but with a Quick Summary.

## 2. Specification Gathering

- Ask the user exactly ONE targeted question at a time, always starting from the single most critical unknown that blocks progress.
- Do NOT list multiple questions at once. Only after the user answers, decide the next most important question and ask it.
- Rank potential questions by impact (highest first) and only ask the highest-impact one.
- Do not restate previous answers. Avoid grouping questions together.
- **IMPORTANT:** No assumptions, think hard and proceed iteratively and thoroughly until all necessary details are clarified.
- Always prioritize the most critical unknowns first.
- If an answer is unclear or incomplete, ask a follow-up before moving on.
- Wait for each answer before asking the next question.
- Be adaptive and not rigid: let each answer guide the next question logically.
- If requirements appear to conflict, present the trade-offs clearly and ask for prioritization.
- Ask exactly one question at a time; if vague/contradictory, you may ask up to 4 follow-ups sequentially, still one at a time.
- **IMPORTANT:** Think hard and stop only when you have enough clear and concrete detail to produce an actionable implementation plan.
- **IMPORTANT**: Once you've gathered all the requirements, provide the user with a quick summary of what needs to be done.
- **IMPORTANT**: NEVER start planning before the user confirms the quick summary.
- **IMPORTANT**: Even if the user confirms the quick summary, ask if you can start planning the development-planner agent.
- **IMPORTANT**: After you gathered all the specification, ask the user if he wants test to be written and follow the next steps accordingly.
  - If the the answer is not clear, DO NOT generate tests.

## 3. Create Files

- Generate slug from task description: lowercase, spaces to hyphens, remove accents/punctuation, keep [a-z0-9-].
- Handle collisions by appending -2, -3, etc.
- Create session directory: `.claude/dev-sessions/YYYY-MM-DD-HHMM-${slug}/`

**A. Context Analysis file:**

- Path: `.claude/dev-sessions/`
- Format: `YYYY-MM-DD-HHMM-${slug}/context-analysis.md`
- Structure:

```md
# Context Analysis

[The saved structured analysis from step 1 exactly as it is. Do not regenerate.]
```

**B. Specification file:**

- Path: `.claude/dev-sessions/`
- Format: `YYYY-MM-DD-HHMM-${slug}/specification.md`
- Structure:

```md
# Specification

[Include here the complete detailed specification gathered through the Q&A process in step 2. Do not summarize - include all requirements, constraints, acceptance criteria, and technical details discussed with the user.]
```

## 4. Track the active session

- Create or update `.claude/dev-sessions/.current-session` with the active session filename, the filename MUST BE `.claude/dev-sessions/YYYY-MM-DD-HHMM-${slug}`.

## 5. Next Steps

Inform the user that the analysis is complete, invite him to take a look at the generated files and
start planning with development-planner agent **only** after `context-analysis.md` and `specification.md` are created.

## Error Handling

- NEVER START CODING, in this phase you only generate a new session, you ONLY create the `context-analysis` and the `specification`
- If context analysis fails: Ask user to verify file paths exist and are accessible
- If file creation fails: Attempt alternative directory structure in current working directory
- If slug generation produces conflicts, suggest appending an incrementing number (e.g., -2, -3) or propose alternative unique slugs
- If the user said he do not want tests, BE SURE to add this in the `context-analysis.md` and `session.md`
- IMPORTANT: If the user provides new specifications or adds features during the session:
  - If session status is `PLANNING` or `READY`:
    - Return to Step 2 (Specification Gathering) to collect and validate the new requirements
    - Resume the full process from that point onward (Steps 2 through execution), using existing context and files as a base
  - If session status is not `PLANNING` or `READY` (e.g., `IN_PROGRESS`, `COMPLETED` etc.):
    - Inform the user that the current session is already in [status]
    - Suggest starting a new session to avoid disrupting the integrity of a finalized or ongoing execution plan

## Validation Steps

1. Confirm context-analysis.md contains the structured analysis from step 1
2. **IMPORTANT**: Confirm specification.md contains the complete detailed specification gathered in step 2
3. Validate that `.claude/dev-sessions/.current-session` file points to the correct session
4. **IMPORTANT**: Check that all dependencies between steps are logically sound (no circular dependencies)
5. Ensure no code generation has occurred in any step.
6. **IMPORTANT**: Start planning **only** after `context-analysis.md` and `specification.md` are created.
7. **IMPORTANT**: NEVER start coding.
