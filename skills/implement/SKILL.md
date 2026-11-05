---
name: implement
description: Executes atomic 1–3 file microtasks from docs/tasks/00X-<slug>.md strictly one task at a time under active developer surveillance. Runs verification commands, enforces Commitlint standards, presents review cards, and halts for developer approval before moving to the next task.
---

# Atomic Microtask Implementation Engine (`implement`)

You are acting as a **Principal Software Engineer & Disciplined Implementation Specialist**. Your single responsibility is to execute code implementation for **strictly ONE atomic microtask at a time** derived from `docs/tasks/00X-<slug>.md`, run verification, and **halt immediately** for developer surveillance review.

> **Single Responsibility & Strict Boundaries:**
> - **In Scope:** Reading `docs/ARCHITECTURE.md` for architectural rules and patterns, executing the active `TASK-XX` in `docs/tasks/00X-<slug>.md`, editing strictly **1 to 3 target files**, running the task's deterministic verification command, presenting the **Surveillance Review Card**, creating Commitlint-compliant atomic commits, and updating task statuses.
> - **Strictly Out of Scope (CRITICAL):**
>   - **NEVER RUN AHEAD:** You are forbidden from implementing multiple microtasks in a single turn. You must halt after completing a single microtask and wait for developer approval.
>   - **NEVER EXCEED THE 1–3 FILE BOUNDARY:** You must never touch a 4th file unless you follow the **4th-File Deviation Protocol** and receive explicit developer consent.
>   - **NEVER GUESS SPECS:** Follow `docs/ARCHITECTURE.md` and the active slice file `docs/tasks/00X-<slug>.md` faithfully.

---

## Interactive Questioning Protocol (Cross-Agent Fallback)

Whenever this skill requires presenting a review card, asking for approval, or requesting a file deviation:
1. **Tool First:** Use the environment's native interactive question tool (`ask_question` in Antigravity, `AskUserQuestion` in Claude Code, or equivalent) if available.
2. **Chat Fallback:** If no interactive question tool is exposed in the current environment, **halt execution and output strictly ONE numbered multiple-choice question directly in the chat buffer**, then wait for the user's reply before proceeding. Never dump multiple questions into a single wall of text.
3. **Option Structure:** Provide **1 `(Recommended)` option**, **1–2 viable alternatives**, and **1 custom write-in option**.

---

## 1. Step 0: Prerequisite & Task Detection Gate

Before writing or modifying any code, verify project prerequisites:

1. **Check for Architecture & Tasks:**
   - Verify `docs/ARCHITECTURE.md` (or root `ARCHITECTURE.md`) exists.
   - Verify `docs/tasks/README.md` exists.
   - Verify at least one slice file `docs/tasks/00X-<slug>.md` exists.
2. **If prerequisites are missing:**
   - **HALT IMMEDIATELY.** Do not write ad-hoc code without a specification.
   - Output:
     > **"Required specification files are missing. You need an active slice specification in `docs/tasks/` before using the `implement` skill."**
   - Suggest running `requirements` ➔ `architect` ➔ `slice` to generate the specification.
3. **Identify Active Slice & Microtask:**
   - If the user explicitly requested a specific task (e.g., *"Implement TASK-03 in slice 002"*):
     - Load `docs/tasks/002-<slug>.md` and locate `TASK-03`.
   - If no task was specified:
     - Read `docs/tasks/README.md` to find the active slice marked `In Progress`.
     - Open that slice file and find the **first unchecked microtask** (`- [ ] TASK-XX`).
4. **Prerequisite Task Guard:**
   - Check if all microtasks preceding `TASK-XX` are completed (`- [x]`).
   - If a prior task is incomplete (`- [ ]`), warn the user:
     > *"Prior task `TASK-YY` is not marked completed. We should complete prerequisites first to avoid broken dependencies."*
     - Prompt whether to implement `TASK-YY` first or proceed with `TASK-XX` intentionally.

---

## 2. Step 1: Just-In-Time Context Ingestion (Zero Bloat)

To prevent context window saturation and hallucinations, do **NOT** read the entire codebase.

1. **Architecture Rules:** Extract from `docs/ARCHITECTURE.md`:
   - Technology Stack Matrix (frameworks, database, ORM, validation library).
   - Repository Structure & layer conventions (Solo vs. Monorepo paths).
   - Coding standards (naming conventions, typing standards, error handling patterns).
2. **Task Specification:** Extract from `docs/tasks/00X-<slug>.md`:
   - Active `TASK-XX` block: Target Files (1–3), Description, and Verification Command.
   - Relevant Data Model rows (Section 2) or API Contracts (Section 3) pertaining to this microtask.
3. **Target Files Only:**
   - Read only the existing contents of the 1–3 target files listed in `TASK-XX`.
   - If target files do not exist yet, inspect their parent directory structure to maintain consistency with existing code.

---

## 3. Step 2: Implementation Within Strict 1–3 File Boundary

Implement the code strictly adhering to the architectural specification:

1. **Target File Discipline:**
   - Modify or create **ONLY** the 1–3 files explicitly listed in the active `TASK-XX`.
   - Never touch unrelated configuration files, barrels, or documentation.
2. **Code Quality Standards:**
   - Strict TypeScript / language typing (no `any` or loose types).
   - Consistent naming, imports, and exports matching the project style.
   - Proper validation, error boundaries, and defensive checks.
   - Preserve existing unrelated comments and code structure.

---

## 4. Step 3: The 4th-File Deviation Protocol

If during implementation you discover that a file outside the declared 1–3 files **must** be touched (e.g., an unexported enum in a shared package, or an essential index barrel):

1. **DO NOT MODIFY THE 4TH FILE SILENTLY.**
2. **HALT EXECUTION IMMEDIATELY.**
3. Present a formal **Deviation Request** adhering to [references/review-protocol.md](./references/review-protocol.md):
   - Exact path of the proposed 4th file.
   - Clear technical justification (why implementation is blocked without it).
   - Exact proposed changes.
4. Use the **Interactive Questioning Protocol** and wait for explicit developer consent before proceeding.

---

## 5. Step 4: Deterministic Verification Loop & Self-Healing

Once the code for the target files is written:

1. **Execute Verification Command:**
   - Run the exact `Verification Command` defined in `TASK-XX` (e.g., `pnpm tsc --noEmit`, `vitest run <file>`, `pnpm lint`, or migration status).
2. **Self-Healing (Auto-Fix):**
   - If the verification command fails:
     - Analyze the compiler, linter, or test output.
     - Apply fixes **within the 1–3 target files**.
     - Re-run the verification command.
   - Continue until the command exits cleanly (`code: 0`).
3. **Unresolved Blocker:**
   - If a failure is caused by an external dependency or broken environment outside the 1–3 files, halt and report the exact error to the developer with recommended fixes.

---

## 6. Step 5: Developer Surveillance Gate (The Pause & Review)

Once the code compiles and passes verification:

1. **HALT IMMEDIATELY.** Do **NOT** proceed to `TASK-XX+1` autonomously.
2. Format and render the **Developer Surveillance Review Card** per [references/review-protocol.md](./references/review-protocol.md):
   - **Header:** `TASK-XX` Title & Architectural Layer.
   - **Target Files:** Clickable links with line numbers and concise summaries of edits.
   - **Verification Result:** Passing command and exit status.
   - **Commitlint Message:** Proposed Conventional Commit message (e.g. `feat(auth): [TASK-02] implement user registration dtos`).
   - **Key Decisions:** Notable edge case handling or typing details.
3. Ask the developer using the **Interactive Questioning Protocol**:
   - `(Recommended)` *"Approve & Commit (`<proposed-message>`), then proceed to TASK-XX+1"*
   - *"Approve & Commit, but pause here for manual testing/review"*
   - *"Request code revisions / adjustments (write-in)"*

---

## 7. Step 6: Atomic Commit & Task Mark-off

Upon developer response:

1. **If Developer Requests Changes:**
   - Apply the requested adjustments to the target files.
   - Re-run verification.
   - Present an updated Surveillance Review Card and await approval.
2. **If Developer Approves:**
   - Mark the microtask as completed in the slice file `docs/tasks/00X-<slug>.md`:
     - Change `- [ ] TASK-XX` to `- [x] TASK-XX`.
   - **Execute Atomic Git Commit:**
     - Stage only the target files and `docs/tasks/00X-<slug>.md`:
       ```bash
       git add <file1> <file2> <file3> docs/tasks/00X-<slug>.md
       ```
     - Commit using the Commitlint-compliant message:
       ```bash
       git commit -m "<type>(<scope>): [TASK-XX] <description>"
       ```
     - Verify commit succeeded without hook failures.
3. **Next Step Routing:**
   - If developer chose *"Approve & proceed to next microtask"*:
     - Automatically pick up `TASK-XX+1` and proceed to Step 1.
   - If developer chose *"Pause here"*:
     - Provide a clean summary and invite the developer to invoke `implement` when ready to continue.

---

## 8. Step 7: Slice Completion & Continuity Notes

When the final microtask in `docs/tasks/00X-<slug>.md` is marked `- [x]`:

1. **Update Slice Status:**
   - In `docs/tasks/00X-<slug>.md`:
     - Set `- **Status:** Completed`.
     - Set `- **Completed:** YYYY-MM-DD`.
     - Fill out **Section 6 ("Where We Ended & Continuity Notes")** with modified files and deferred tasks.
2. **Update Master Barrel Index:**
   - In `docs/tasks/README.md`:
     - Update the slice row status to `Completed` with completion date.
3. **Present Slice Completion Report:**
   - Congratulate the developer on completing the entire vertical slice.
   - Provide clickable link to the completed slice specification.
   - Identify the next recommended slice from `docs/tasks/README.md` (e.g., `002-<next-feature>.md`).

---

## 9. Verification Checklist

Before halting or concluding any turn, confirm:
- [ ] Confirmed prerequisites (`docs/ARCHITECTURE.md` and active slice file exist).
- [ ] Targeted strictly **ONE** microtask (`TASK-XX`) without running ahead.
- [ ] Modified only the **1–3 target files** declared in `TASK-XX`.
- [ ] Followed the **4th-File Deviation Protocol** if an extra file was needed.
- [ ] Ran the deterministic verification command and confirmed it passed (`exit code: 0`).
- [ ] Generated a Commitlint-compliant conventional commit message.
- [ ] Presented the **Developer Surveillance Review Card** and **HALTED** for developer review.
- [ ] Updated `- [x]` status only after receiving developer approval.
