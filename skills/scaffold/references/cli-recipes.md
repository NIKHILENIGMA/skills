# Non-Interactive CLI Recipes, Glue Templates & Scaffold Review Card

Use this reference during `scaffold` execution to ensure CLI generators never hang on interactive prompts and glue files follow consistent standards.

---

## 1. Non-Interactive Official CLI Recipes

> **CRITICAL RULE:** Always pass non-interactive flags (`--yes`, `--skip-git`, `--skip-install`, or `CI=true`) so the agent terminal never blocks waiting for user input. After running any sub-app generator inside `apps/*`, verify that `apps/<name>/.git` does **not** exist.

### Frontend & Mobile Surfaces

| Framework | Non-Interactive CLI Command |
|---|---|
| **Next.js (App Router + TS + Tailwind)** | `pnpm dlx create-next-app@latest apps/web --ts --tailwind --eslint --app --src-dir --import-alias "@/*" --use-pnpm --yes --skip-install --disable-git` |
| **React + Vite (Admin SPA + TS)** | `pnpm create vite@latest apps/admin --template react-ts --no-interactive` |
| **Expo (React Native Mobile)** | `pnpm dlx create-expo-app@latest apps/mobile --template default --no-install` |
| **Astro (Content / Marketing)** | `pnpm create astro@latest apps/marketing -- --template minimal --typescript strict --no-install --no-git --yes` |
| **shadcn/ui (Component Init)** | `pnpm dlx shadcn@latest init --defaults --yes` |

### Backend & Full-Stack Surfaces

| Framework | Non-Interactive CLI Command |
|---|---|
| **NestJS (Enterprise Node/TS)** | `pnpm dlx @nestjs/cli@latest new apps/api --package-manager pnpm --skip-git --skip-install --strict` |
| **Fastify / Hono (Lightweight Node/TS)** | Initialize `apps/api/package.json` (`pnpm --filter api init`) and add `fastify` / `hono`, `tsx`, `typescript`, `@types/node` via `pnpm --filter api add ...` |
| **Nx Monorepo Workspace** | `pnpm dlx create-nx-workspace@latest . --preset=ts --nxCloud=skip --interactive=false --skipGit=true` |
| **Python (FastAPI / Django via `uv`)** | `uv init apps/api --no-readme --no-pin-python && uv --directory apps/api add fastapi uvicorn pydantic sqlalchemy alembic` |
| **Go (Fiber / Chi + `sqlc`)** | `mkdir -p apps/api && cd apps/api && go mod init <module-name>/api` |
| **Java (Spring Boot via Initializr API)** | `curl -sSL https://start.spring.io/starter.tgz -d type=maven-project -d language=java -d bootVersion=3.4.0 -d baseDir=apps/api -d groupId=com.app -d artifactId=api -d dependencies=web,data-jpa,postgresql,validation | tar -xzf -` |

---

## 2. Commitlint Configuration (`commitlint.config.mjs`)

Ensure Commitlint accepts the `<type>(<scope>): [TASK-XX] <subject>` format required by the `implement` skill (do not enforce `subject-case` rules that reject a leading `[TASK-XX]` tag):

```javascript
export default {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [
      2,
      'always',
      ['feat', 'fix', 'docs', 'style', 'refactor', 'perf', 'test', 'build', 'ci', 'chore', 'revert'],
    ],
    'subject-case': [0], // Allow [TASK-XX] prefix followed by lowercase prose or camelCase identifiers
    'header-max-length': [2, 'always', 100],
  },
};
```

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
3. `packages/database` — Drizzle ORM + PostgreSQL connection client skeleton
4. `packages/contracts` — Shared Zod validation & `ApiResponse<T>` envelope
5. `packages/tsconfig` — Shared strict TypeScript base configs

#### 🛡️ Tooling, Guardrails & Agent Rules Wired
- **Workspace & Tasks:** `package.json`, `pnpm-workspace.yaml`, `turbo.json`
- **Quality & Git Hooks:** `biome.json`, `commitlint.config.mjs`, `.husky/commit-msg`
- **Local Backing Services:** `docker-compose.yml` (PostgreSQL 16 + Redis 7), `.env.example`
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
