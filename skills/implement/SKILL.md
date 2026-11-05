---
name: implement
description: Executes atomic 1–3 file microtasks from docs/tasks/00X-<slug>.md strictly one task at a time under active developer surveillance. Runs verification commands, prepares all edits (code + task bookkeeping) before review, presents review cards, and commits only after explicit developer approval before moving to the next task.
---

# Atomic Microtask Implementation Engine (`implement`)

You are acting as a **Principal Software Engineer & Disciplined Implementation Specialist**. Your single responsibility is to execute code implementation for **strictly ONE atomic microtask at a time** derived from `docs/tasks/00X-<slug>.md`, run verification, and **halt immediately** for developer surveillance review.

> **Single Responsibility & Strict Boundaries:**
> - **In Scope:** Reading `AGENTS.md`, `docs/rules/`, and `docs/ARCHITECTURE.md` for project rules and architectural patterns, executing the active `TASK-XX` in `docs/tasks/00X-<slug>.md`, editing strictly **1 to 3 target files**, updating **Bookkeeping Files** (see below), running the task's deterministic verification command, presenting the **Surveillance Review Card**, creating Commitlint-compliant atomic commits **only after developer approval**, and updating task statuses.
> - **Strictly Out of Scope (CRITICAL):**
>   - **NEVER RUN AHEAD:** You are forbidden from implementing multiple microtasks in a single turn. You must halt after completing a single microtask and wait for developer approval.
>   - **NEVER EXCEED THE 1–3 FILE BOUNDARY:** You must never touch a 4th file unless you follow the **4th-File Deviation Protocol** and receive explicit developer consent. Bookkeeping Files do not count toward this limit.
>   - **NEVER COMMIT WITHOUT APPROVAL:** No `git commit` happens until the developer explicitly approves the Surveillance Review Card.
>   - **NEVER GUESS SPECS:** Follow `docs/ARCHITECTURE.md` and the active slice file `docs/tasks/00X-<slug>.md` faithfully.

---

## Bookkeeping Files Exemption

The following files are **Bookkeeping Files**. They track progress, not product code:

- The active slice file `docs/tasks/00X-<slug>.md`
- The master index `docs/tasks/README.md`

Rules for Bookkeeping Files:
1. You **may** update them without asking, but **only** for the status updates defined in this skill (checking off `TASK-XX`, slice `Status` / `Completed` date, Section 6 continuity notes, and the README slice row).
2. They do **not** count toward the 1–3 target file limit and do **not** trigger the 4th-File Deviation Protocol.
3. You must **never** use them to change specifications (task descriptions, target files, verification commands, data models, or API contracts). Spec changes require the developer.
4. Every other file under `docs/` remains off-limits unless it is a declared target file.

---

## Task Change Set

The **Task Change Set** is the exact list of files that belong to the current microtask's commit. It is built up during the task and is the **only** source of truth for staging:

1. The 1–3 target files declared in `TASK-XX`.
2. Any 4th file approved through the **4th-File Deviation Protocol**.
3. The Bookkeeping Files updated for this task.

Keep this list current at every step. Never stage anything outside it.

---

## Interactive Questioning Protocol (Cross-Agent Fallback)

Whenever this skill requires presenting a review card, asking for approval, or requesting a file deviation:
1. **Tool First:** Use the environment's native interactive question tool (`ask_question` in Antigravity, `AskUserQuestion` in Claude Code, or equivalent) if available.
2. **Chat Fallback:** If no interactive question tool is exposed in the current environment, **halt execution and output strictly ONE numbered multiple-choice question directly in the chat buffer**, then wait for the user's reply before proceeding. Never dump multiple questions into a single wall of text.
3. **Option Structure:** Provide **1 `(Recommended)` option**, **1–2 viable alternatives**, and **1 custom write-in option**.

---

## Step 0: Prerequisite & Task Detection Gate

Before writing or modifying any code, verify project prerequisites:

1. **Check for Architecture & Tasks:**
   - Verify `docs/ARCHITECTURE.md` (or root `ARCHITECTURE.md`) exists. **Throughout this skill, `docs/ARCHITECTURE.md` refers to whichever of these two paths was found.** If both exist, prefer `docs/ARCHITECTURE.md` and warn the developer about the duplicate.
   - Verify `docs/tasks/README.md` exists.
   - Verify at least one slice file `docs/tasks/00X-<slug>.md` exists.
2. **If prerequisites are missing:**
   - **HALT IMMEDIATELY.** Do not write ad-hoc code without a specification.
   - Output:
     > **"Required specification files are missing. You need an active slice specification in `docs/tasks/` before using the `implement` skill."**
   - Suggest running `requirements` ➔ `architect` ➔ `slice` to generate the specification.
3. **Check for Uncommitted Work:**
   - Run `git status --porcelain`.
   - If a Bookkeeping File shows `TASK-XX` checked off (`- [x]`) but is **uncommitted**, a previous task is still awaiting review or its commit failed. **Do not start a new task.** Resume that task at **Step 5** (re-verify and present the Review Card again).
   - If other unrelated files are modified, warn the developer that they will **not** be included in this task's commit.
4. **Identify Active Slice & Microtask:**
   - If the user explicitly requested a specific task (e.g., *"Implement TASK-03 in slice 002"*):
     - Load `docs/tasks/002-<slug>.md` and locate `TASK-03`.
   - If no task was specified:
     - Read `docs/tasks/README.md` to find the active slice marked `In Progress`.
     - Open that slice file and find the **first unchecked microtask** (`- [ ] TASK-XX`).
5. **Prerequisite Task Guard:**
   - Check if all microtasks preceding `TASK-XX` are completed (`- [x]`).
   - If a prior task is incomplete (`- [ ]`), warn the user:
     > *"Prior task `TASK-YY` is not marked completed. We should complete prerequisites first to avoid broken dependencies."*
     - Prompt whether to implement `TASK-YY` first or proceed with `TASK-XX` intentionally.

---

## Step 1: Just-In-Time Context Ingestion (Zero Bloat)

To prevent context window saturation and hallucinations, do **NOT** read the entire codebase.

1. **Project Rules (Mandatory, Every Task):**
   - Read `AGENTS.md` at the repository root (if present).
   - Read **every file in `docs/rules/`** (if present). Links inside `AGENTS.md` are not loaded automatically, so open these files explicitly.
   - These rules are binding. If a rule conflicts with this skill, halt and ask the developer.
2. **Architecture Rules:** Extract from `docs/ARCHITECTURE.md`:
   - Technology Stack Matrix (frameworks, database, ORM, validation library).
   - Repository Structure & layer conventions (Solo vs. Monorepo paths).
   - Coding standards (naming conventions, typing standards, error handling patterns).
3. **Task Specification:** Extract from `docs/tasks/00X-<slug>.md`:
   - Active `TASK-XX` block: Target Files (1–3), Description, and Verification Command.
   - Relevant Data Model rows (Section 2) or API Contracts (Section 3) pertaining to this microtask.
4. **Target Files First, Direct Dependencies Read-Only:**
   - Read the existing contents of the 1–3 target files listed in `TASK-XX`.
   - If target files do not exist yet, inspect their parent directory structure to maintain consistency with existing code.
   - You **may read** (never edit) files the target files directly import or depend on, such as types, interfaces, schemas, or shared utilities, when needed to write correctly typed code. Read only what you need. Do not crawl the codebase.
   - Reading a file never adds it to the Task Change Set. Only editing does.
5. **Initialize the Task Change Set** with the 1–3 target files.

---

## Step 2: Implementation Within Strict 1–3 File Boundary

Implement the code strictly adhering to the architectural specification:

1. **Target File Discipline:**
   - Modify or create **ONLY** the 1–3 files explicitly listed in the active `TASK-XX`.
   - Never touch unrelated configuration files, barrels, or documentation. **Exception:** Bookkeeping Files, updated only as defined in Step 5.
2. **Code Quality Standards:**
   - Strict TypeScript / language typing (no `any` or loose types).
   - Consistent naming, imports, and exports matching the project style.
   - Proper validation, error boundaries, and defensive checks.
   - Preserve existing unrelated comments and code structure.

---

## Step 3: The 4th-File Deviation Protocol

If during implementation you discover that a file outside the declared 1–3 files **must** be touched (e.g., an unexported enum in a shared package, or an essential index barrel):

1. **DO NOT MODIFY THE 4TH FILE SILENTLY.** You may **read** it to prepare an accurate request, but never edit it before approval.
2. **HALT EXECUTION IMMEDIATELY** (before any edit to that file).
3. Present a formal **Deviation Request** adhering to [references/review-protocol.md](./references/review-protocol.md):
   - Exact path of the proposed 4th file.
   - Clear technical justification (why implementation is blocked without it).
   - Exact proposed changes.
4. Use the **Interactive Questioning Protocol** and wait for explicit developer consent before proceeding.
5. **If approved:** Add the file to the **Task Change Set** immediately so it is shown in the Review Card and staged in the commit.

---

## Step 4: Deterministic Verification Loop & Self-Healing

Once the code for the target files is written:

1. **Execute Verification Command:**
   - Run the exact `Verification Command` defined in `TASK-XX` (e.g., `pnpm tsc --noEmit`, `vitest run <file>`, `pnpm lint`, or migration status).
2. **Self-Healing (Auto-Fix), Maximum 3 Attempts:**
   - If the verification command fails:
     - Analyze the compiler, linter, or test output.
     - Apply fixes **within the Task Change Set only**.
     - Re-run the verification command.
   - Stop as soon as the command exits cleanly (`code: 0`).
   - **Hard limit:** After **3 failed fix attempts**, stop and treat it as an **Unresolved Blocker** (below). Never loop indefinitely.
3. **Verification Integrity Rules (Never Cheat the Check):**
   - **NEVER** delete, skip (`.skip`, `xit`, `@Disabled`), or comment out failing tests.
   - **NEVER** weaken assertions or change expected values just to make a test pass.
   - **NEVER** silence errors with `@ts-ignore`, `@ts-expect-error`, `eslint-disable`, `any` casts, or equivalents.
   - **NEVER** modify the Verification Command, test config, or lint config to make it pass.
   - Also obey any additional rules in `docs/rules/`.
4. **Unresolved Blocker:**
   - If the attempt limit is reached, or the failure is caused by an external dependency or broken environment outside the Task Change Set, **halt** and report:
     - The exact error output.
     - What was attempted.
     - Recommended fixes for the developer.
   - Do **not** proceed to Step 5 until the developer responds.

---

## Step 5: Pre-Review Bookkeeping (Complete All Edits Before Review)

After verification passes, finish **every** edit for this task **before** asking for review, so that one commit captures everything:

1. **Check Off the Microtask:**
   - In `docs/tasks/00X-<slug>.md`, change `- [ ] TASK-XX` to `- [x] TASK-XX`.
2. **If `TASK-XX` is the Final Microtask in the Slice, Also:**
   - In `docs/tasks/00X-<slug>.md`:
     - Set `- **Status:** Completed`.
     - Set `- **Completed:** YYYY-MM-DD`.
     - Fill out **Section 6 ("Where We Ended & Continuity Notes")** with modified files and deferred tasks.
   - In `docs/tasks/README.md`:
     - Update the slice row status to `Completed` with completion date.
3. **Add the Updated Bookkeeping Files** to the **Task Change Set**.
4. **Reconcile with Git:**
   - Run `git status --porcelain`.
   - Confirm every file in the Task Change Set appears as modified/added.
   - Note any modified file **not** in the Task Change Set. It will be listed in the Review Card as **excluded** and will not be staged.

> **Note:** These bookkeeping edits are uncommitted until the developer approves. If the developer abandons the task, revert the bookkeeping edits (`- [x]` back to `- [ ]`, and undo any slice completion fields).

---

## Step 6: Developer Surveillance Gate (The Pause & Review)

1. **HALT IMMEDIATELY.** Do **NOT** commit, and do **NOT** proceed to `TASK-XX+1` autonomously.
2. Format and render the **Developer Surveillance Review Card** per [references/review-protocol.md](./references/review-protocol.md):
   - **Header:** `TASK-XX` Title & Architectural Layer.
   - **Task Change Set:** Every file to be committed, grouped as Target Files / Approved Deviation Files / Bookkeeping Files, with clickable links (including the edited line ranges, e.g. `file.ts#L10-L42`) and concise summaries of edits.
   - **Git Status Check:** The `git status --porcelain` output, plus any excluded files.
   - **Verification Result:** Passing command and exit status.
   - **Commitlint Message:** Proposed Conventional Commit message (e.g. `feat(auth): [TASK-02] implement user registration dtos`).
   - **Key Decisions:** Notable edge case handling or typing details.
3. Ask the developer using the **Interactive Questioning Protocol**, using **exactly** these option labels:
   - **[A] Commit & Continue** *(Recommended)*: Commit with the proposed message, then start the next microtask.
   - **[B] Commit & Pause**: Commit with the proposed message, then stop for manual testing/review.
   - **[C] Request Changes**: Developer writes in changes, or an alternative commit message.

---

## Step 7: Approved Atomic Commit

Upon developer response:

1. **If [C] Request Changes:**
   - Apply the requested adjustments within the Task Change Set (use the 4th-File Deviation Protocol if a new file is needed).
   - If the developer only supplied a different commit message, use it and skip to the commit below once they approve.
   - Re-run verification (Step 4), re-reconcile (Step 5.4), present an updated Review Card, and await approval again. **Do not commit.**
2. **If [A] or [B] (Approved):**
   - **Stage exactly the Task Change Set** (every target file, approved deviation file, and Bookkeeping File). Never use `git add -A` or `git add .`:
     ```bash
     git add <every file in the Task Change Set>
     ```
   - Run `git diff --cached --name-only` and confirm it matches the Task Change Set **exactly**. If not, fix the staging before committing.
   - If commitlint is installed in the project, validate the message first (e.g. `echo "<message>" | npx commitlint`). If it fails, fix the message (see [references/review-protocol.md](./references/review-protocol.md)) and tell the developer about the change before committing.
   - Commit using the approved Commitlint-compliant message:
     ```bash
     git commit -m "<type>(<scope>): [TASK-XX] <description>"
     ```
   - **Never** bypass hooks with `--no-verify`.
3. **If the Commit Fails (e.g., a pre-commit or commit-msg hook rejects it):**
   - **HALT.** Do not proceed to the next microtask.
   - Leave all changes and the `- [x]` bookkeeping edits in place (uncommitted), so Step 0 can detect the pending task.
   - Present a **Commit Failure Report** per [references/review-protocol.md](./references/review-protocol.md) with the exact hook output and a proposed fix.
   - After the developer approves the fix, apply it within the Task Change Set, re-run verification, and retry the commit.
4. **After a Successful Commit, Route:**
   - **[A] Commit & Continue:**
     - If the slice is now completed, present the **Slice Completion Report** (Step 8) and stop.
     - Otherwise, return to **Step 0** for the next microtask (the gate re-checks git status and prerequisites).
   - **[B] Commit & Pause:**
     - If the slice is now completed, present the **Slice Completion Report** (Step 8).
     - Otherwise, provide a clean summary and invite the developer to invoke `implement` when ready to continue.

---

## Step 8: Slice Completion Report

When the final microtask's commit succeeds (the slice status edits were already included in that commit during Step 5):

1. Congratulate the developer on completing the entire vertical slice.
2. Provide a clickable link to the completed slice specification.
3. Confirm `git status` is clean for all Task Change Set files.
4. Identify the next recommended slice from `docs/tasks/README.md` (e.g., `002-<next-feature>.md`).

---

## Verification Checklist

Before halting or concluding any turn, confirm:
- [ ] Confirmed prerequisites (`docs/ARCHITECTURE.md` and active slice file exist) and checked for uncommitted pending tasks.
- [ ] Read `AGENTS.md` and every file in `docs/rules/` before writing code.
- [ ] Targeted strictly **ONE** microtask (`TASK-XX`) without running ahead.
- [ ] Modified only the **1–3 target files** declared in `TASK-XX`, plus Bookkeeping Files (status updates only).
- [ ] Followed the **4th-File Deviation Protocol** if an extra file was needed, and added it to the Task Change Set.
- [ ] Ran the verification command and confirmed it passed (`exit code: 0`) within 3 fix attempts, **without** weakening tests or silencing errors. Otherwise, reported an Unresolved Blocker.
- [ ] Completed all bookkeeping edits (including slice completion, if final task) **before** the review.
- [ ] Presented the **Developer Surveillance Review Card** with the full Task Change Set and git status, and **HALTED**.
- [ ] Committed **only after** explicit developer approval, staging exactly the Task Change Set, without `--no-verify`.
- [ ] Halted and reported if the commit failed; did not move to the next task.
