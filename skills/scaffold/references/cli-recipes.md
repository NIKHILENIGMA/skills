# Non-Interactive CLI Recipes, Glue Templates & Scaffold Review Card

Use this reference during `scaffold` execution to ensure CLI generators never hang on interactive prompts and glue files follow consistent standards.

---

## 1. Non-Interactive Official CLI Recipes

> **CRITICAL RULES:**
> 1. Always pass non-interactive flags (`--yes`, `--skip-git`, `--skip-install`, or `CI=true`) so the agent terminal never blocks waiting for user input.
> 2. After running any sub-app generator inside `apps/*`, verify that `apps/<name>/.git` does **not** exist.
> 3. **Prevent Split-Brain Linters:** When the workspace uses **Biome** (`templates/linters/biome.json`), pass `--biome` to `create-next-app` (instead of `--eslint`), and immediately delete any CLI-generated `.eslintrc.*`, `eslint.config.*`, or `.prettierrc*` inside sub-apps (`apps/api`, `apps/admin`) so the root `biome.json` is the single source of truth.

### Frontend & Mobile Surfaces

| Framework | Non-Interactive CLI Command |
|---|---|
| **Next.js (with Biome)** | `pnpm dlx create-next-app@latest apps/web --ts --tailwind --biome --app --src-dir --import-alias "@/*" --use-pnpm --yes --skip-install --disable-git` |
| **Next.js (with ESLint)** | `pnpm dlx create-next-app@latest apps/web --ts --tailwind --eslint --app --src-dir --import-alias "@/*" --use-pnpm --yes --skip-install --disable-git` |
| **React + Vite (Admin SPA + TS)** | `pnpm create vite@latest apps/admin --template react-ts --no-interactive` |
| **Expo (React Native Mobile)** | `pnpm dlx create-expo-app@latest apps/mobile --template default --no-install` |
| **Astro (Content / Marketing)** | `pnpm create astro@latest apps/marketing -- --template minimal --typescript strict --no-install --no-git --yes` |
| **shadcn/ui (Component Init)** | `pnpm dlx shadcn@latest init --defaults --yes` |

### Backend & Full-Stack Surfaces (Cross-Platform Safe)

| Framework | Non-Interactive CLI Command |
|---|---|
| **NestJS (Enterprise Node/TS)** | `pnpm dlx @nestjs/cli@latest new apps/api --package-manager pnpm --skip-git --skip-install --strict` *(Delete `apps/api/.eslintrc.js` and `apps/api/.prettierrc` if root uses Biome)* |
| **Fastify / Hono (Lightweight Node/TS)** | Initialize `apps/api/package.json` (`pnpm --filter api init`) and add `fastify` / `hono`, `tsx`, `typescript`, `@types/node` via `pnpm --filter api add ...` |
| **Nx Monorepo Workspace** | `pnpm dlx create-nx-workspace@latest . --preset=ts --nxCloud=skip --interactive=false --skipGit=true` |
| **Python (FastAPI / Django via `uv`)** | `uv init apps/api --no-readme --no-pin-python` followed by `uv --directory apps/api add fastapi uvicorn pydantic sqlalchemy alembic` |
| **Go (Fiber / Chi + `sqlc`)** | Create `apps/api` directory, then run `go mod init <module-name>/api` inside `apps/api` |
| **Java (Spring Boot via Initializr API)** | Use `curl.exe -sSL "https://start.spring.io/starter.tgz?type=maven-project&language=java&bootVersion=3.4.0&baseDir=apps/api&groupId=com.app&artifactId=api&dependencies=web,data-jpa,postgresql,validation" \| tar -xzf -` *(Always use `curl.exe` on Windows PowerShell so `Invoke-WebRequest` alias does not intercept flags)* |

---

## 2. Modular Configuration Templates (`../templates/`)

Do **not** invent linter, formatter, or git hook configs from scratch. Copy them directly from [`../templates/`](../templates/):

| Category | Template Path | Target Path in Workspace | Required DevDependencies / Setup |
|---|---|---|---|
| **Biome Linter/Formatter** | `templates/linters/biome.json` | `./biome.json` | `@biomejs/biome` |
| **ESLint v9 Flat Config** | `templates/linters/eslint.config.mjs` | `./eslint.config.mjs` | `eslint`, `typescript-eslint`, `@eslint/js` |
| **Prettier Config** | `templates/linters/.prettierrc` | `./.prettierrc` | `prettier` |
| **EditorConfig** | `templates/editor/.editorconfig` | `./.editorconfig` | *(None)* |
| **Commitlint (`[TASK-XX]` ready)** | `templates/git/commitlint.config.mjs` | `./commitlint.config.mjs` | `@commitlint/cli`, `@commitlint/config-conventional` |
| **Husky v9 Hooks** | `templates/git/husky/commit-msg` & `pre-commit` | `./.husky/commit-msg` & `./.husky/pre-commit` | `husky`, `lint-staged`, plus `"prepare": "husky"` in root `package.json` and `git config core.hooksPath .husky` |
| **Lint-Staged (Biome)** | `templates/git/.lintstagedrc.biome.json` | `./.lintstagedrc.json` | `lint-staged` |
| **Lint-Staged (ESLint)** | `templates/git/.lintstagedrc.eslint.json` | `./.lintstagedrc.json` | `lint-staged` |
| **GitHub Actions CI (JS/TS)** | `templates/ci/github-ci.yml` | `./.github/workflows/ci.yml` | *(Adapt steps for `uv` / `go` / `mvn` if non-JS backend)* |
| **Base TypeScript Config** | `templates/typescript/tsconfig.base.json` | `./packages/tsconfig/base.json` (or `./tsconfig.base.json`) | `typescript` |

---

## 3. Agent Context & Rules Templates (`AGENTS.md` + `docs/rules/`)

The `implement` skill reads `AGENTS.md` and every file inside `docs/rules/` before writing code for any microtask. Always generate these files during `scaffold`.

### Root `AGENTS.md` Template

```markdown
# Project Agent Instructions (`AGENTS.md`)

## 1. Source-of-Truth Specifications
- **Product Requirements:** [`docs/PRD.md`](./docs/PRD.md)
- **System Architecture & Stack Matrix:** [`docs/ARCHITECTURE.md`](./docs/ARCHITECTURE.md)
- **Master Slice & Task Index:** [`docs/tasks/README.md`](./docs/tasks/README.md)

## 2. Mandatory Engineering Rules (`docs/rules/`)
Agents must read and obey all files in `docs/rules/` before modifying code:
- [`docs/rules/code-standards.md`](./docs/rules/code-standards.md)
- [`docs/rules/testing-integrity.md`](./docs/rules/testing-integrity.md)

## 3. Workspace Verification Commands
- **Typecheck:** `<e.g., pnpm typecheck>`
- **Lint:** `<e.g., pnpm lint>`
- **Unit / Integration Tests:** `<e.g., pnpm test>`
- **Build:** `<e.g., pnpm build>`

## 4. Implementation Discipline
- Execute strictly **one microtask (`TASK-XX`) at a time** from the active `docs/tasks/00X-<slug>.md` slice.
- Never touch more than the **1–3 target files** declared in `TASK-XX` without following the 4th-File Deviation Protocol.
- Never commit without explicit developer approval.
```

### `docs/rules/code-standards.md` Template

```markdown
# Code & Architecture Standards

1. **Strict Type Safety:** Never use `any`, unsafe `as unknown as T` casts, or untyped dictionaries. All external inputs (HTTP bodies, query params, env vars) must be validated at the boundary using the schema library specified in `docs/ARCHITECTURE.md`.
2. **Layer Boundaries:**
   - Controllers / Route Handlers handle HTTP parsing, auth guards, and status codes only.
   - Domain Services contain business rules and database transactions.
   - Shared contracts (`packages/contracts`) must not import backend-only ORM internals.
3. **Money, Dates & IDs:** Store monetary amounts in integer minor units (cents/paise), timestamps in UTC ISO-8601, and enforce idempotency on state-mutating endpoints where specified.
```

### `docs/rules/testing-integrity.md` Template

```markdown
# Verification & Testing Integrity Rules

1. **Never Cheat a Failing Check:**
   - Forbidden: Deleting failing tests, adding `.skip` / `xit` / `@Disabled`, or commenting out assertions.
   - Forbidden: Weakening expected values just to make a broken implementation pass.
   - Forbidden: Suppressing compiler or linter errors with `@ts-ignore`, `@ts-expect-error`, `eslint-disable`, `biome-ignore`, or `# type: ignore`.
2. **Hook Discipline:** Never bypass git hooks with `--no-verify`.
3. **3-Strike Escalation:** If a verification command fails 3 times in a row during self-healing, stop immediately and report the exact blocker to the developer.
```

---

## 4. Scaffold Surveillance Review Card Template

When `scaffold` finishes Phase 3 verification, **halt** and render this card before committing:

````markdown
### 🏗️ Project Scaffold Review

- **Architecture Blueprint:** [`docs/ARCHITECTURE.md`](./docs/ARCHITECTURE.md)
- **Workspace Mode:** `<Mode A: Greenfield | Mode B: Incremental>`
- **Status:** ✅ All Verification Checks Passed · ⏸️ Not yet committed

#### 📦 Scaffolded Applications & Packages
1. `apps/web` — Next.js App Router (generated via `create-next-app` non-interactive CLI)
2. `apps/api` — NestJS strict API shell (generated via `@nestjs/cli` non-interactive CLI)
3. `packages/database` (`@repo/database`) — Drizzle ORM + PostgreSQL connection client skeleton
4. `packages/contracts` (`@repo/contracts`) — Shared Zod validation & `ApiResponse<T>` envelope
5. `packages/tsconfig` (`@repo/tsconfig`) — Shared strict TypeScript base configs

#### 🛡️ Tooling, Guardrails & Agent Rules Wired
- **Workspace & Tasks:** `package.json`, `pnpm-workspace.yaml`, `turbo.json`
- **Quality & Git Hooks:** `biome.json` (or ESLint+Prettier), `.editorconfig`, `commitlint.config.mjs`, `.lintstagedrc.json`, `.husky/commit-msg`, `.husky/pre-commit`
- **CI & Local Backing Services:** `.github/workflows/ci.yml`, `docker-compose.yml` (PostgreSQL 16 + Redis 7), `.env.example`
- **Agent Rules:** [`AGENTS.md`](./AGENTS.md), [`docs/rules/code-standards.md`](./docs/rules/code-standards.md), [`docs/rules/testing-integrity.md`](./docs/rules/testing-integrity.md)

#### 🧪 Verification Commands Executed
- **Typecheck (`pnpm typecheck`):** `Exit code: 0`
- **Linter (`pnpm lint`):** `Exit code: 0`
- **Smoke Build (`pnpm build`):** `Exit code: 0`

#### 📝 Proposed Commitlint Message
```text
chore(repo): scaffold project workspace and tooling from architecture
```

---
**What would you like to do?**
- [A] **(Recommended) Approve & Commit:** Commit the foundation scaffold, then proceed to `slice`
- [B] **Approve & Commit, then Pause:** Commit the scaffold and stop here for manual inspection
- [C] **Request Adjustments:** Modify configs, packages, or scripts before committing
````
