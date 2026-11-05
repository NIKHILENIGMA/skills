# Vertical Slice & Master Barrel Templates (`docs/tasks/`)

Use these templates when creating or updating the master barrel index (`docs/tasks/README.md`) and generating an individual vertical slice microtask file (`docs/tasks/00X-<slug>.md`).

---

## 1. Master Barrel Index Template (`docs/tasks/README.md`)

```markdown
# Project Master Task & Slice Index (`docs/tasks/`)

> **Master Execution Index:**
> - This `README.md` is the single source of truth tracking all product features from `docs/PRD.md` and `docs/ARCHITECTURE.md`.
> - Each feature has a dedicated vertical slice file (`001-<slug>.md`, `002-<slug>.md`, ...).
> - **The 1–3 File Rule:** Never edit 10–12 files at once. When implementing, open only the active `00X-<slug>.md` and execute one atomic microtask (`TASK-01`, `TASK-02`, ...) at a time with instant verification.
> - **Slice Statuses:** `Not Sliced` ➔ `Planned` ➔ `In Progress` ➔ `Completed`. `slice` sets `Not Sliced` / `Planned`; `implement` sets `In Progress` / `Completed`. **Only one slice may be `In Progress` at a time.**

## Master Execution Index

| Slice # | Slice File | Feature ID | Phase | Priority | Status | Started | Completed | Summary |
|---|---|---|---|---|---|---|---|---|
| `001` | [001-foundation-and-setup.md](./001-foundation-and-setup.md) | `CORE` | Phase 1: Foundation | **P0** | `Completed` | YYYY-MM-DD | YYYY-MM-DD | Base workspace setup, DB schema barrel, seed data |
| `002` | [002-<feat-slug>.md](./002-<feat-slug>.md) | `FEAT-01` | Phase 2: MVP Core | **P0** | `In Progress` | YYYY-MM-DD | — | <End-to-end slice summary> |
| `003` | [003-<next-feat-slug>.md](./003-<next-feat-slug>.md) | `FEAT-02` | Phase 2: MVP Core | **P0** | `Planned` | — | — | <End-to-end slice summary> |
| — | — | `FEAT-03` | Phase 2: MVP Core | **P0** | `Not Sliced` | — | — | <Feature summary from PRD> |
```

---

## 2. Individual Vertical Slice Template (`docs/tasks/00X-<slug>.md`)

```markdown
# Slice `00X`: <Feature Name> (`<FEAT-ID>`)

- **Feature ID:** `<FEAT-ID>`
- **Phase / Priority:** `<Phase 2: MVP Core / P0>`
- **Status:** `Planned` *(`implement` sets `In Progress` when TASK-01 starts and `Completed` when all microtasks are `COMPLETED`)*
- **Derived From:** [`docs/PRD.md`](../PRD.md) & [`docs/ARCHITECTURE.md`](../ARCHITECTURE.md)
- **Started:** `—`
- **Completed:** `—`

---

## 1. Slice Boundary & Requirements Context
- **Goal:** <1–2 sentence summary of what this slice accomplishes from database to UI>
- **Actor / Client Surface:** <e.g., Customer on Web App / Admin on Dashboard>
- **Prerequisites & Dependencies:** <Which existing tables, shared modules, or base routes this builds on>
- **Permissions & RBAC:** <Public / Authenticated User / Role: Admin / Owner only>
- **Critical Business Rules & Edge Cases:**
  - **Rule 1:** <e.g., Email addresses must be normalized to lowercase before validation>
  - **Rule 2:** <e.g., Monetary transactions must use integer minor units (cents/paise) with server-side calculation>
  - **Edge Case 1:** <e.g., Duplicate email registration returns 409 Conflict with code `EMAIL_ALREADY_REGISTERED`>
  - **Edge Case 2:** <e.g., Rate limit exceeded (>5 attempts/min) returns 429 Too Many Requests>

---

## 2. Data Model & Schema Changes
*(Omit if this slice requires no database or state schema updates)*

| Table / Store | Column / Field | Type & Constraints | Indexes & Relations | Cascade / Audit |
|---|---|---|---|---|
| `<table_name>` | `<column_name>` | `<type>` NOT NULL | FOREIGN KEY (`ref_id`) REFERENCES ... | ON DELETE CASCADE |
| `<table_name>` | `<status>` | `enum('draft','active','archived')` | INDEX(`status`) | — |

---

## 3. API Contract Specifications
*(Omit if this is a Frontend-Only slice without backend endpoints)*

### Endpoint: `<METHOD> /api/v1/<path>`
- **Authentication:** `<Bearer JWT / Cookie / Public>`
- **URL / Query Params:** `<None or ?page=1&limit=20>`
- **Request Body (Example):**
```json
{
  "fieldA": "value",
  "fieldB": 100
}
```
- **Success Response (`200 OK` or `201 Created`):**
```json
{
  "success": true,
  "data": {
    "id": "res_123",
    "fieldA": "value",
    "createdAt": "2026-11-04T12:00:00.000Z"
  }
}
```
- **Error Responses:**
  - `400 Bad Request` (`VALIDATION_ERROR`): Missing or invalid payload fields.
  - `401 Unauthorized` (`UNAUTHORIZED`): Expired or missing token.
  - `409 Conflict` (`RESOURCE_ALREADY_EXISTS`): Duplicate entry.

---

## 4. UI Surfaces & Interaction States
*(Omit if this is a Backend-Only slice without frontend views)*

| Component / Screen | Route Path | State / Hook | Loading State | Empty State | Error State |
|---|---|---|---|---|---|
| `<FeatureView>` | `/feature` | `useFeatureQuery()` | Shimmer skeleton cards | Illustration + "Get Started" CTA | Toast alert + retry button |
| `<FeatureFormModal>` | Modal Dialog | `useFeatureMutation()` | Submit button spinner + inputs disabled | — | Inline field errors under inputs |

---

## 5. Atomic Microtask Execution Plan (1–3 Files per Task)

> **Golden Execution Rule:**
> 1. Edit ONLY the 1–3 target files listed per microtask.
> 2. Run the specified `Verification Command`.
> 3. Fix any compiler/lint/test failure, then get developer approval before setting the microtask to `COMPLETED`.
>
> **Microtask Statuses:** `TO_DO` ➔ `IN_PROGRESS` ➔ `COMPLETED`. Tasks stay in this order; only the status changes. Keep the Progress Overview table and each task's `Status` line in sync.

### Progress Overview

| Task | Layer | Title | Status |
|---|---|---|---|
| `TASK-01` | Data Layer | Database Schema & Migration | `TO_DO` |
| `TASK-02` | Contracts | Request/Response DTOs & Validation Schema | `TO_DO` |
| `TASK-03` | Service | Domain Business Logic & Error Handling | `TO_DO` |
| `TASK-04` | Controller & Module | Route Wiring & Guards | `TO_DO` |
| `TASK-05` | Client Hook | Typed Fetcher & State/Query Hook | `TO_DO` |
| `TASK-06` | UI Components | View, Forms & Edge States | `TO_DO` |
| `TASK-07` | Integration & Verification | Route Mount & End-to-End Smoke Test | `TO_DO` |

### `TASK-01` [Data Layer]: Database Schema & Migration
- **Target Files (1–2):**
  1. `apps/api/src/prisma/schema.prisma` (or migration file / entity definition)
- **Description:** Define the new entities, indexes, and relations according to Section 2.
- **Verification Command:** `pnpm db:migrate` (or `npx prisma db push --dry-run` / equivalent)
- **Status:** `TO_DO`

### `TASK-02` [Contracts]: Request/Response DTOs & Validation Schema
- **Target Files (1–2):**
  1. `apps/api/src/modules/<feature>/dto/create-<feature>.dto.ts`
  2. `apps/api/src/modules/<feature>/dto/<feature>-response.dto.ts`
- **Description:** Implement class-validator or Zod schemas with complete validation rules and error messages.
- **Verification Command:** `pnpm tsc --noEmit`
- **Status:** `TO_DO`

### `TASK-03` [Service]: Domain Business Logic & Error Handling
- **Target Files (1–2):**
  1. `apps/api/src/modules/<feature>/<feature>.service.ts`
  2. `apps/api/src/modules/<feature>/<feature>.service.spec.ts`
- **Description:** Implement core business rules, idempotency checks, transactions, and throw standard domain exceptions.
- **Verification Command:** `pnpm vitest run apps/api/src/modules/<feature>/<feature>.service.spec.ts`
- **Status:** `TO_DO`

### `TASK-04` [Controller & Module]: Route Wiring & Guards
- **Target Files (1–2):**
  1. `apps/api/src/modules/<feature>/<feature>.controller.ts`
  2. `apps/api/src/modules/<feature>/<feature>.module.ts`
- **Description:** Wire controller endpoints, route guards (auth/roles), attach validation pipes, and register module into main app.
- **Verification Command:** `pnpm tsc --noEmit`
- **Status:** `TO_DO`

### `TASK-05` [Client Hook]: Typed Fetcher & State/Query Hook
- **Target Files (1–2):**
  1. `apps/web/src/features/<feature>/api/<feature>.client.ts`
  2. `apps/web/src/features/<feature>/hooks/use-<feature>.ts`
- **Description:** Implement typed HTTP client methods and React Query / SWR / Zustand hooks with cache invalidation keys.
- **Verification Command:** `pnpm tsc --noEmit`
- **Status:** `TO_DO`

### `TASK-06` [UI Components]: View, Forms & Edge States
- **Target Files (1–3):**
  1. `apps/web/src/features/<feature>/components/<feature>-form.tsx`
  2. `apps/web/src/features/<feature>/components/<feature>-card.tsx`
  3. `apps/web/src/features/<feature>/components/<feature>-skeleton.tsx`
- **Description:** Build presentation components handling loading, empty, and validation error states.
- **Verification Command:** `pnpm lint && pnpm tsc --noEmit`
- **Status:** `TO_DO`

### `TASK-07` [Integration & Verification]: Route Mount & End-to-End Smoke Test
- **Target Files (1–2):**
  1. `apps/web/src/app/(dashboard)/<feature>/page.tsx`
  2. `apps/web/src/features/<feature>/__tests__/<feature>-smoke.test.tsx`
- **Description:** Mount the feature into the page route, verify navigation, and run smoke test confirming happy path and 404/400 edge cases.
- **Verification Command:** `pnpm test:feature` (or `pnpm tsc --noEmit && pnpm test`)
- **Status:** `TO_DO`

---

## 6. Where We Ended & Continuity Notes
*(To be filled out by the implementation agent once all microtasks are complete)*

- **Final Status:** `Completed` *(filled by `implement` with the completion date)*
- **Files Modified / Created:**
  - `<file_1>`
  - `<file_2>`
- **Known Limitations / Deferred to Future Slices:**
  - `<None or details of next slice>`
- **Recommended Next Slice:** `[00X+1-<next-feature>.md](./00X+1-<next-feature>.md)`
```
