# Developer Surveillance Review Protocol & Commit Standards

This reference guides the `implement` skill in presenting work for developer review, handling the 4th-file deviation protocol, and generating Commitlint-compliant atomic commits.

---

## 1. The Developer Surveillance Review Card

Whenever an atomic microtask passes its verification command, the agent **must halt** and render this standardized review card:

```markdown
### 🛡️ Developer Surveillance Review: `TASK-XX` [<Layer>]

- **Slice:** `00X-<slug>` (`<FEAT-ID>`)
- **Microtask:** `TASK-XX: <Task Title>`
- **Status:** ✅ Verification Passed

#### 📁 Modified Files (1–3 Target Files)
1. [`apps/api/src/modules/auth/dto/register.dto.ts`](./apps/api/src/modules/auth/dto/register.dto.ts) — Added user registration DTO with Zod schema validation
2. [`apps/api/src/modules/auth/dto/index.ts`](./apps/api/src/modules/auth/dto/index.ts) — Exported DTO and response contracts from barrel

#### 🧪 Verification Command Executed
- **Command:** `pnpm tsc --noEmit`
- **Result:** `Exit code: 0` (Clean compilation, zero errors)

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
- [1] **(Recommended)** Approve & Commit, then proceed to `TASK-XX+1`
- [2] Approve & Commit, but pause here for manual inspection
- [3] Request revisions or changes to this code (write-in)
```

---

## 2. The 4th-File Deviation Protocol

If during implementation the agent discovers that a file outside the 1–3 target files declared in `TASK-XX` must be modified (e.g., an unexported symbol, a missing type in a shared barrel, or a database index adjustment):

> **THE GOLDEN RULE:**  
> The agent is **strictly forbidden** from touching any 4th file silently. It must pause immediately and present a **Deviation Request**.

### Deviation Request Format

```markdown
### ⚠️ Scope Deviation Request: Additional File Required

While implementing `TASK-XX`, a change outside the declared 1–3 target files is required to proceed.

- **Proposed 4th File:** [`packages/types/src/user.ts`](./packages/types/src/user.ts)
- **Technical Justification:** The active service needs the `UserRole` enum which is currently missing from the shared types barrel, causing compiler error TS2305.
- **Proposed Modification:** Add and export `export type UserRole = 'ADMIN' | 'CUSTOMER';` (3 lines).
- **Risk Assessment:** Low. Pure type export, no runtime impact on existing modules.

**May I proceed with editing this additional file?**
- [1] **(Recommended)** Approve: Allow modifying this specific file for this task
- [2] Deny: Propose an alternative approach without touching this file
- [3] Write custom instructions
```

The agent **must wait for explicit developer consent** before opening or editing the file.

---

## 3. Commitlint & Conventional Commits Reference

When generating atomic git commits, the agent must inspect the workspace for `.commitlintrc`, `.commitlintrc.json`, `.commitlintrc.js`, or `commitlint.config.js`. If found, it strictly adheres to configured rules.

### Standard Format
```text
<type>(<scope>): [TASK-XX] <lowercase subject without trailing period>
```

### Type Mapping by Architectural Layer

| Architectural Layer | Microtask Focus | Commit Type | Example Message |
|---|---|---|---|
| **Data Layer** | Schemas, migrations, seeders, indexes | `feat` | `feat(db): [TASK-01] add customer table schema and migrations` |
| **Contracts** | DTOs, Zod/class-validator schemas, envelopes | `feat` | `feat(auth): [TASK-02] add user registration and login dtos` |
| **Domain Service** | Business logic, calculations, transactions | `feat` | `feat(billing): [TASK-03] implement stripe invoice calculation service` |
| **Controller / Route** | Route wiring, HTTP guards, route parameters | `feat` | `feat(users): [TASK-04] add user profile controller and auth guard` |
| **Client Hook / API** | Fetchers, TanStack Query hooks, Zustand stores | `feat` | `feat(checkout): [TASK-05] add usecheckout query and mutation hooks` |
| **UI Presentation** | Views, forms, modals, skeletons, empty states | `feat` | `feat(dashboard): [TASK-06] implement metrics cards and loading skeletons` |
| **Smoke / Tests** | Unit, integration, or e2e smoke tests | `test` | `test(auth): [TASK-07] add user registration end-to-end smoke test` |
| **Bug Fix / Refactor** | Fixing an existing bug or refactoring during review | `fix` / `refactor` | `fix(auth): [TASK-03] normalize email casing before password hash check` |
| **Config / Tooling** | Build configs, package scripts, linting adjustments | `chore` | `chore(deps): [TASK-01] update database client dependencies` |

### Critical Rules for Commitlint Compliance
1. **Lowercase Type & Scope:** Always lowercase (e.g. `feat(auth):`, never `Feat(Auth):`).
2. **Subject in Imperative Mood:** Use "add", "implement", "wire", "configure" (never "added", "implements", "wiring").
3. **No Trailing Period:** Never end the commit subject with `.`.
4. **Header Length Limit:** Keep the entire commit header under 72–100 characters (standard commitlint `header-max-length`).
5. **No WIP Commits:** Every commit is a fully passing, verified atomic unit.
