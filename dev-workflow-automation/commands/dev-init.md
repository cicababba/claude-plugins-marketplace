# Improved Claude Command (Session Initialization Only)

Initialize a structured development session for any development task (new feature, bug fix, refactoring, performance optimization, etc.).

**ABSOLUTE SCOPE & RESTRICTIONS**

- This command ONLY initializes a development session.
- It MUST ONLY:
  - gather context,
  - gather specifications via Q&A,
  - generate the `context-analysis.md` and `specification.md` files,
  - update `.claude/dev-sessions/.current-session`.
- It MUST NEVER:
  - perform any planning,
  - generate a development plan,
  - create any section or content titled "Plan", "Planning", "Implementation Plan", "Roadmap", or similar.
- Planning is done ONLY by the separate `development-planner` agent, AFTER this command has completed and the files exist.
- **IMPORTANT**: This command must never produce code snippets or implementations, only analysis and specifications.
- **IMPORTANT**: All generated files (context-analysis.md, specification.md) must ALWAYS be written in ENGLISH, regardless of the language used in the conversation.

---

## 1. Identify the relevant context

- Ask the user which files or folders you should analyze to understand the relevant context. Give examples (e.g., `/src/auth`, `/src/models/user.ts`).
- When the user provides multiple paths, create one subagent per path.
- Each subagent must scan only its assigned path, and extract:
  - Existing code patterns in that area
  - Dependencies and imports
  - Related tests (if any)
  - Current implementation details
- After scanning all paths, aggregate all findings from the subagents and organize them by analyzed path.
- Create and save in this step a structured analysis (separate findings by path), this will be reused later. Output it to the user with a **Quick Summary**.

**IMPORTANT:** In this step you are ONLY analyzing. Do NOT start planning, do NOT propose implementation steps.

---

## 2. Specification Gathering

- Ask the user exactly ONE targeted question at a time, always starting from the single most critical unknown that blocks progress.
- Do NOT list multiple questions at once. Only after the user answers, decide the next most important question and ask it.
- Rank potential questions by impact (highest first) and only ask the highest-impact one.
- Do not restate previous answers. Avoid grouping questions together.
- **IMPORTANT:** Make no assumptions; think hard and proceed iteratively and thoroughly until all necessary details are clarified.
- Always prioritize the most critical unknowns first.
- If an answer is unclear or incomplete, ask a follow-up before moving on.
- Ask exactly one question at a time; if vague/contradictory, you may ask up to 4 follow-ups sequentially, still one at a time.
- If requirements appear to conflict, present the trade-offs clearly and ask for prioritization.

### Tests decision

- After specification gathering:
  - Provide the user with a **quick summary**.
  - Ask the user to confirm or correct it.
  - **Never** start planning before this confirmation.
  - After confirmation, ask whether tests should be written.
  - If unclear, do NOT assume tests.

**CRITICAL RULE:**  
Even after summary confirmation and test preference is known, NO planning happens here.

---

## 3. Create Files

- Generate slug from task description.
- Create session directory: `.claude/dev-sessions/YYYY-MM-DD-HHMM-${slug}/`

### Context Analysis file

```md
# Context Analysis

[The saved structured analysis from step 1 exactly as it is. Do not regenerate.]
```

### Specification file

```md
# Specification

[Include here the complete detailed specification gathered through the Q&A process in step 2. Do not summarize - include all requirements, constraints, acceptance criteria, and technical details discussed with the user, including test decision.]
```

---

## 4. Track active session

Update `.claude/dev-sessions/.current-session` with:

```
.claude/dev-sessions/YYYY-MM-DD-HHMM-${slug}
```

---

## 5. Next Steps

- Inform user that files are created.
- Invite them to review.
- Then ask if they want to start the `development-planner` agent.
- **But NEVER plan here.**

---

## Error Handling

- No coding in this phase.
- If context analysis fails: ask user to verify paths.
- If file creation fails: fallback directory.
- On slug conflict: append -2, -3, ...
- If user said no tests: include this decision in both files.

---

## Validation

1. Confirm context-analysis.md contains structured analysis.
2. Confirm specification.md contains full spec.
3. Validate `.current-session`.
4. No circular dependencies.
5. No code.
6. No planning.
7. Planning only happens later via `development-planner`.
