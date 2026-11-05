# Developer Surveillance Review Protocol & Commit Standards

This reference guides the `implement` skill in presenting work for developer review, handling the 4th-file deviation protocol, reporting commit failures, and generating Commitlint-compliant atomic commits.

> **Core Principle:** All edits for a microtask (code **and** bookkeeping) are completed **before** review. Nothing is committed until the developer explicitly approves. One approved commit captures the entire **Task Change Set**.

---

## 1. The Developer Surveillance Review Card

Whenever an atomic microtask passes its verification command and its bookkeeping edits are done, the agent **must halt** and render this standardized review card. Do **not** commit before the developer answers.

````markdown
### 🛡️ Developer Surveillance Review: `TASK-XX` [<Layer>]

- **Slice:** `00X-<slug>` (`<FEAT-ID>`)
- **Microtask:** `TASK-XX: <Task Title>`
- **Status:** ✅ Verification Passed · ⏸️ Not yet committed

#### 📦 Task Change Set (Files That Will Be Committed)

**Target Files (1–3):**
1. [`apps/api/src/modules/auth/dto/register.dto.ts`](./apps/api/src/modules/auth/dto/register.dto.ts#L1-L42): Added user registration DTO with Zod schema validation
2. [`apps/api/src/modules/auth/dto/index.ts`](./apps/api/src/modules/auth/dto/index.ts#L3-L4): Exported DTO and response contracts from barrel

**Approved Deviation Files:**
- _None_ (or list each approved 4th file with a link and summary)

**Bookkeeping Files:**
- [`docs/tasks/001-auth.md`](./docs/tasks/001-auth.md): Checked off `TASK-02`
- _(Final task only)_ [`docs/tasks/README.md`](./docs/tasks/README.md): Slice marked `Completed`

#### 🗂️ Git Status Check
```text
 M apps/api/src/modules/auth/dto/index.ts
?? apps/api/src/modules/auth/dto/register.dto.ts
 M docs/tasks/001-auth.md
```
- **Excluded (modified but NOT part of this task, will not be staged):** _None_

#### 🧪 Verification Command Executed
- **Command:** `pnpm tsc --noEmit`
- **Result:** `Exit code: 0` (Clean compilation, zero errors)
- **Fix Attempts Used:** 1 of 3

#### 📝 Proposed Commitlint Message
```text
feat(auth): [TASK-02] implement user registration dtos and validation schemas
```

#### 🔍 Key Implementation Notes
- Applied strict lowercase email transformation in schema.
- Added password complexity regex matching `docs/ARCHITECTURE.md` security rules.
- Handled empty string edge case for optional referral code.

---
**What would you like to do?**
- [A] **(Recommended) Commit & Continue:** Commit with the proposed message, then start `TASK-XX+1`
- [B] **Commit & Pause:** Commit with the proposed message, then stop for manual inspection
- [C] **Request Changes:** Write in code changes or an alternative commit message
````

> **Important:** Use these exact option labels (`[A] Commit & Continue`, `[B] Commit & Pause`, `[C] Request Changes`). The routing logic in `SKILL.md` depends on them.

---

## 2. The 4th-File Deviation Protocol

If during implementation the agent discovers that a file outside the 1–3 target files declared in `TASK-XX` must be modified (e.g., an unexported symbol, a missing type in a shared barrel, or a database index adjustment):

> **THE GOLDEN RULE:**  
> The agent is **strictly forbidden** from touching any 4th file silently. It must pause immediately and present a **Deviation Request**.
>
> **Bookkeeping Files** (`docs/tasks/00X-<slug>.md` and `docs/tasks/README.md`) are exempt. Updating their task/slice status does **not** require a Deviation Request.

### Deviation Request Format

```markdown
### ⚠️ Scope Deviation Request: Additional File Required

While implementing `TASK-XX`, a change outside the declared 1–3 target files is required to proceed.

- **Proposed 4th File:** [`packages/types/src/user.ts`](./packages/types/src/user.ts)
- **Technical Justification:** The active service imports the `UserRole` type, which is not yet exported from `packages/types/src/user.ts`, causing compiler error TS2305 (`Module has no exported member 'UserRole'`).
- **Proposed Modification:** Add `export type UserRole = 'ADMIN' | 'CUSTOMER';` (1 line).
- **Risk Assessment:** Low. Pure type export, no runtime impact on existing modules.

**May I proceed with editing this additional file?**
- [1] **(Recommended)** Approve: Allow modifying this specific file for this task
- [2] Deny: Propose an alternative approach without touching this file
- [3] Write custom instructions
```

The agent **may read** the proposed file to write an accurate request, but **must wait for explicit developer consent** before editing it.

**If approved:** The file is added to the **Task Change Set** immediately. It must appear under **Approved Deviation Files** in the Review Card and must be staged in the task's commit.

---

## 3. Commit Failure Report

If `git commit` fails after developer approval (e.g., a `pre-commit` lint hook or `commit-msg` commitlint hook rejects it), the agent must:

1. **HALT.** Do not proceed to the next microtask.
2. **Keep all changes in place**, including the `- [x]` bookkeeping edits, uncommitted. On the next run, Step 0 detects the pending task.
3. **Never** retry with `--no-verify`.
4. Render this report:

````markdown
### ❌ Commit Failed: `TASK-XX`

- **Attempted Message:** `feat(auth): [TASK-02] implement user registration dtos`
- **Rejected By:** `commit-msg` hook (commitlint)
- **State:** Changes are staged but **not committed**. `TASK-XX` is checked off in the working tree only.

#### 🧾 Hook Output
```text
✖   header must not be longer than 72 characters [header-max-length]
```

#### 🔧 Proposed Fix
- Shorten message to: `feat(auth): [TASK-02] add registration dtos`

---
**How should we proceed?**
- [1] **(Recommended)** Apply the proposed fix and retry the commit
- [2] Leave it uncommitted. I will fix it manually
- [3] Write custom instructions
````

If the fix requires code changes (e.g., a pre-commit linter failure), the agent applies them **within the Task Change Set**, re-runs the verification command, and retries the commit only after the developer approves.

---

## 4. Commitlint & Conventional Commits Reference

When generating atomic git commits, the agent must first look for a commitlint configuration. If one is found, its rules **override** the defaults below. Check for:

- `commitlint.config.{js,cjs,mjs,ts,cts,mts}`
- `.commitlintrc`, `.commitlintrc.{json,yaml,yml,js,cjs,mjs,ts,cts,mts}`
- A `"commitlint"` key in `package.json`

If commitlint is installed, validate the message before committing (e.g. `echo "<message>" | npx commitlint`).

### Standard Format
```text
<type>(<scope>): [TASK-XX] <subject>
```

- `[TASK-XX]` is always written in **uppercase** exactly as it appears in the slice file. It is the only uppercase part of the prefix.
- `<subject>` is written in **lowercase** prose, imperative mood, with no trailing period.
- Do **not** lowercase code identifiers (writing `usecheckout` for `useCheckout` mangles the name). Prefer describing the change in plain words (`add checkout query and mutation hooks`). If an identifier is essential, keep its original casing.
- **Compatibility check:** Some commitlint `subject-case` configurations may reject a subject that starts with `[`. If validation fails for this reason, keep the type and scope and move the task ID to a footer instead (`Refs: TASK-XX`), and tell the developer about the change.

### Type Mapping by Architectural Layer

| Architectural Layer | Microtask Focus | Commit Type | Example Message |
|---|---|---|---|
| **Data Layer** | Schemas, migrations, seeders, indexes | `feat` | `feat(db): [TASK-01] add customer table schema and migrations` |
| **Contracts** | DTOs, Zod/class-validator schemas, envelopes | `feat` | `feat(auth): [TASK-02] add user registration and login dtos` |
| **Domain Service** | Business logic, calculations, transactions | `feat` | `feat(billing): [TASK-03] implement stripe invoice calculation service` |
| **Controller / Route** | Route wiring, HTTP guards, route parameters | `feat` | `feat(users): [TASK-04] add user profile controller and auth guard` |
| **Client Hook / API** | Fetchers, TanStack Query hooks, Zustand stores | `feat` | `feat(checkout): [TASK-05] add checkout query and mutation hooks` |
| **UI Presentation** | Views, forms, modals, skeletons, empty states | `feat` | `feat(dashboard): [TASK-06] implement metrics cards and loading skeletons` |
| **Smoke / Tests** | Unit, integration, or e2e smoke tests | `test` | `test(auth): [TASK-07] add user registration end-to-end smoke test` |
| **Bug Fix / Refactor** | Fixing an existing bug or refactoring during review | `fix` / `refactor` | `fix(auth): [TASK-03] normalize email casing before password hash check` |
| **Config / Tooling** | Build configs, package scripts, linting adjustments | `chore` | `chore(deps): [TASK-01] update database client dependencies` |

### Critical Rules for Commitlint Compliance
1. **Lowercase Type & Scope:** Always lowercase (e.g. `feat(auth):`, never `Feat(Auth):`). The `[TASK-XX]` tag stays uppercase.
2. **Subject in Imperative Mood:** Use "add", "implement", "wire", "configure" (never "added", "implements", "wiring").
3. **No Trailing Period:** Never end the commit subject with `.`.
4. **Header Length Limit:** Use the project's `header-max-length` if configured. Otherwise, the hard limit is **100 characters** (the `@commitlint/config-conventional` default), and the target is **72 characters or fewer** for readability in `git log`.
5. **No WIP Commits:** Every commit is a fully passing, verified atomic unit.
6. **Approval First:** Never run `git commit` before the developer selects `[A] Commit & Continue` or `[B] Commit & Pause`.
7. **Stage the Task Change Set Only:** Stage files explicitly by path. Never use `git add -A` or `git add .`. Confirm with `git diff --cached --name-only` before committing.
8. **No Hook Bypass:** Never use `--no-verify`.
