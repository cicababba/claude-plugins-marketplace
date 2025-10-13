**IMPORTANT**: This command must never produce code snippets, pseudocode, or implementations. Only analysis and specifications.  
**IMPORTANT**: All generated files (`context-analysis.md`, `specification.md`, etc.) MUST be written in ENGLISH, regardless of conversation language.

---

## Process Control & States

Maintain an internal state machine and do not deviate:

- **States**: `INIT` → `CONTEXT_SCANNING` → `SPEC_QA` → `SUMMARY_AWAIT_CONFIRMATION` → `AWAIT_FILE_GEN_DECISION` → `FILES_WRITTEN` → `AWAIT_PLANNER_DECISION` → `READY_TO_PLAN`.
- **Booleans**:
  - `summary_confirmed` (default `false`)
  - `files_generated` (default `false`)
  - `tests_requested` (`"unspecified" | "yes" | "no"`, default `"unspecified"`)

### Hard Guardrails

- You MUST NOT propose or mention planning while `files_generated=false`. If asked to plan earlier, respond:  
  “Planning is blocked until `context-analysis.md` and `specification.md` are created.”
- If new requirements are introduced after `FILES_WRITTEN`, revert to `SPEC_QA`, set `files_generated=false`, and regenerate files only after confirmation.
- Never output any code or pseudocode. Do not include fenced code blocks that look like implementations.

---

## 1) Identify the relevant context (`INIT` → `CONTEXT_SCANNING`)

- Ask which files/folders to analyze (e.g., `/src/auth`, `/src/models/user.ts`).
- If multiple paths, create one subagent per path. **Cap subagents at 8**; if more, batch and inform the user.
- Each subagent scans only its path and extracts:
  - Existing code patterns
  - Dependencies/imports
  - Related tests
  - Current implementation details
- Aggregate findings by path.
- Save a structured analysis and output a **Quick Summary** plus the per-path details.

Move to `SPEC_QA`.

---

## 2) Specification Gathering (`SPEC_QA`)

- Ask exactly **one** targeted question at a time, always the highest-impact unknown first.
- No assumptions; if an answer is unclear, ask a focused follow-up before moving on.
- If requirements conflict, present trade-offs and request prioritization.
- Stop only when details are sufficient to produce an actionable implementation plan.

Then:

- Provide a **Quick Summary** of requirements.
- Set state to `SUMMARY_AWAIT_CONFIRMATION`.

### After gathering all requirements

1. Provide a quick summary.
2. Wait for explicit confirmation from the user that the summary is correct. Set `summary_confirmed=true`.
3. Move to `AWAIT_FILE_GEN_DECISION` and ASK:  
   “Do you want me to generate `context-analysis.md` and `specification.md` now?”
4. **ONLY** if the user says **YES**:

   - Generate slug from task description (lowercase, hyphens, `[a-z0-9-]`, strip accents/punctuation). Handle collisions by appending `-2`, `-3`, etc.
   - Create directory: `.claude/dev-sessions/YYYY-MM-DD-HHMM-${slug}/`
   - Write:

     - `.claude/dev-sessions/YYYY-MM-DD-HHMM-${slug}/context-analysis.md`:

       ```md
       # Context Analysis

       [The saved structured analysis from step 1 exactly as it is. Do not regenerate.]
       ```

     - `.claude/dev-sessions/YYYY-MM-DD-HHMM-${slug}/specification.md`:

       ```md
       # Specification

       [Include here the complete detailed specification gathered through the Q&A process in step 2. Do not summarize - include all requirements, constraints, acceptance criteria, and technical details discussed with the user.]
       ```

   - Update `.claude/dev-sessions/.current-session` with the session path.
   - Set `files_generated=true` and move to `FILES_WRITTEN`.

5. Inform the user the files are ready and provide the session path.
6. Move to `AWAIT_PLANNER_DECISION` and **ONLY THEN** ask:  
   “Do you want me to start the development-planner agent?”

---

## 3) Error Handling & Idempotency

- If context scan fails: ask the user to verify paths exist and are accessible.
- If file creation fails: attempt current working directory as fallback; report final location.
- If slug collision: append `-2`, `-3`, etc.
- If the user said they do not want tests, record `tests_requested="no"` and include this in both files.
- If the user asks to regenerate files, overwrite in the same session directory and re-update `.current-session`.

---

## 4) Validation Gates (must pass before moving forward)

1. `context-analysis.md` contains the structured analysis from step 1 verbatim.
2. `specification.md` contains the complete detailed specification from step 2 (no summaries).
3. `.current-session` points to `.claude/dev-sessions/YYYY-MM-DD-HHMM-${slug}`.
4. No code/pseudocode has been produced.
5. Planning can start **only** when `files_generated=true`.

---

## 5) Test Policy

- After specifications: explicitly ask whether to include tests.
- If the answer is unclear, set `tests_requested="unspecified"` and do not generate tests.
- If “no,” record this in both files.

---

## 6) Next Steps Messaging

- After `FILES_WRITTEN`:  
  “Analysis is complete. Please review `context-analysis.md` and `specification.md`. Planning can start on request.”

---

## 7) NEVER start coding

- This phase only initializes a new session and creates the two files.
