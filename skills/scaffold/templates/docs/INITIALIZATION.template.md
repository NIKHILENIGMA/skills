# Workspace Initialization Plan (`docs/INITIALIZATION.md`)

- **Architecture Source:** [`docs/ARCHITECTURE.md`](./ARCHITECTURE.md)
- **Status:** 🟡 AWAITING_DEVELOPER_REVIEW | 🔵 IN_PROGRESS | 🟢 COMPLETED
- **Initialized At:** <TIMESTAMP>

---

## 1. Locked Decisions Matrix

| Surface / Tooling | Decision | Exact Flags / Configuration |
|---|---|---|
| **Package Manager** | `<pnpm | bun | npm>` | Workspace manifests & lockfile |
| **Frontend (`apps/web`)** | `<Next.js | Vite | Expo>` | `<Exact CLI command with flags>` |
| **UI & Styling** | `<Tailwind + shadcn/ui | Tailwind only | Vanilla>` | `<Color palette, components.json>` |
| **Backend (`apps/api`)** | `<NestJS | Fastify | FastAPI | Go>` | `<Exact CLI command with flags>` |
| **Shared Packages** | `@repo/*` monorepo | `@repo/tsconfig`, `@repo/contracts`, `@repo/database` |
| **Linter & Formatter** | `<Biome | ESLint + Prettier>` | Presets from `templates/linters/` |
| **Git Guardrails** | Commitlint + Husky v9 + lint-staged | `[TASK-XX]` subject format accepted |
| **Local Services** | Docker Compose | `<Postgres / Redis / RabbitMQ with assigned ports>` |
| **CI/CD Pipeline** | GitHub Actions | `.github/workflows/ci.yml` |

---

## 2. Foundation Step-by-Step Execution Board

- [ ] **STEP-01: Root Manifests & Workspace Orchestration**
  - *Action:* Write `package.json`, `pnpm-workspace.yaml`, and `turbo.json`.
  - *Status:* `[TODO]`

- [ ] **STEP-02: Frontend App Shell (`apps/web`)**
  - *Action:* Execute non-interactive frontend CLI generator with approved styling & UI flags.
  - *Status:* `[TODO]`

- [ ] **STEP-03: Backend App Shell (`apps/api`)**
  - *Action:* Execute non-interactive backend CLI generator and purge conflicting sub-app linter files.
  - *Status:* `[TODO]`

- [ ] **STEP-04: Shared Packages & Workspace Linking**
  - *Action:* Create `@repo/tsconfig`, `@repo/contracts`, `@repo/database` and pre-link via `"workspace:*"` into `apps/*/package.json`.
  - *Status:* `[TODO]`

- [ ] **STEP-05: Quality, Tooling & Git Guardrails**
  - *Action:* Copy approved presets from `templates/` (`biome.json`/ESLint, `.editorconfig`, `commitlint.config.mjs`, `.lintstagedrc.json`, `.husky/`).
  - *Status:* `[TODO]`

- [ ] **STEP-06: Local Backing Services & Environment Stubs**
  - *Action:* Generate `docker-compose.yml` and `.env.example` with exact port and service mappings.
  - *Status:* `[TODO]`

- [ ] **STEP-07: CI/CD Pipeline & Agent Engineering Rules**
  - *Action:* Copy `.github/workflows/ci.yml`, create root `AGENTS.md`, and write `docs/rules/` (`code-standards.md`, `testing-integrity.md`).
  - *Status:* `[TODO]`

- [ ] **STEP-08: Dependency Install, Format Normalization & Verification**
  - *Action:* Run root install (`pnpm install`), normalize CLI formatting, and verify `typecheck`, `lint`, and `build` exit `code: 0`.
  - *Status:* `[TODO]`
