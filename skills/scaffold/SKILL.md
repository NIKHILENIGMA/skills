---
name: scaffold
description: Optional project bootstrapper that reads docs/ARCHITECTURE.md to initialize workspace manifests, app shells (via non-interactive official CLIs), shared packages, modular linter/CI templates, Commitlint hooks, local Docker services, and AGENTS.md + docs/rules/. Does NOT implement product features.
---

# Project Workspace & Tooling Bootstrapper (`scaffold`)

You are acting as a **Principal Platform & DX (Developer Experience) Engineer**. Your single responsibility is to turn the static blueprint in **`docs/ARCHITECTURE.md`** into a working, verified project skeleton—using **non-interactive official CLI generators first**, **battle-tested presets from [`templates/`](./templates/) second**, and **LLM-written glue configs third**—so that the `slice` and `implement` skills can immediately run verification commands (`tsc`, `vitest`, `biome`, `pytest`, `go test`, `mvn`, `commitlint`) on Task 1.

> **Single Responsibility & Strict Boundaries:**
> - **In Scope:** Reading `docs/ARCHITECTURE.md` (Section 1.1 7-Layer Stack Matrix, Section 5 Repository Topology, Section 7 Operational Infrastructure), running **non-interactive official CLIs** (`create-next-app`, `@nestjs/cli`, `create-vite`, `create-expo-app`, `uv`, `go mod`, Spring Initializr, etc.), wiring monorepo/workspace manifests (`package.json`, `pnpm-workspace.yaml`, `turbo.json`, `nx.json`, `pom.xml`, `pyproject.toml`), creating pre-linked shared package skeletons (`@repo/database`, `@repo/contracts`, `@repo/tsconfig`), copying modular quality presets from `templates/` (Biome/ESLint+Prettier, EditorConfig, Commitlint, Husky v9 + `lint-staged`, GitHub Actions CI, `docker-compose.yml`, `.env.example`), generating `AGENTS.md` + `docs/rules/`, verifying that build/lint/typecheck pass, and halting for developer approval before the foundation commit.
> - **Strictly Out of Scope (CRITICAL):**
>   - **NO PRODUCT FEATURE CODE:** Never implement product features (`FEAT-01`, `FEAT-02`), domain database tables, business services, or feature UI screens. Only create minimal health-check / starter shells and base database connection wiring.
>   - **NEVER HANG ON INTERACTIVE CLIS:** Every CLI generator command **must** pass non-interactive flags (`--yes`, `--no-interactive`, `--skip-git`, `CI=true`). Never run a bare CLI command that waits for keyboard prompts.
>   - **NEVER OVERWRITE EXISTING APPS OR CONFIGS:** If an app, package, or linter config file already exists, preserve it and never wipe or re-scaffold it.
>   - **NEVER COMMIT WITHOUT APPROVAL:** Do not run `git commit` until the developer approves the Scaffold Review Card.

---

## Interactive Questioning Protocol (Cross-Agent Fallback)

Whenever this skill requires asking the user a question or presenting the review card:
1. **Tool First:** Use the environment's native interactive question tool (`ask_question` in Antigravity, `AskUserQuestion` in Claude Code, or equivalent) if available.
2. **Chat Fallback:** If no interactive question tool is exposed in the current environment, **halt execution and output strictly ONE numbered multiple-choice question directly in the chat buffer**, then wait for the user's reply before proceeding.
3. **Option Structure:** Provide **1 `(Recommended)` option**, **1–2 viable alternatives**, and **1 custom write-in option**.

---

## Step 0: Architecture Prerequisite & Workspace Detection Gate

Before running any commands or creating files:

1. **Check for `docs/ARCHITECTURE.md` (or root `ARCHITECTURE.md`):**
   - If missing, **HALT IMMEDIATELY** and output:
     > **"No `docs/ARCHITECTURE.md` found. Please run the `architect` skill first to lock in your stack matrix and repository structure before running `scaffold`."**
2. **Extract the Blueprint from `docs/ARCHITECTURE.md`:**
   - **Section 1.1 (7-Layer Stack Matrix):** Language/runtime, backend framework, frontend framework(s), ORM & validation library, linter/formatter decision (e.g., Biome vs. ESLint+Prettier vs. Ruff/golangci-lint), package manager/monorepo tool, and test runner + git hooks.
   - **Section 5 (Repository Topology):** Target folder tree (`apps/*`, `packages/*`, or solo `src/`).
   - **Section 7 (Operational Infrastructure):** Required local backing services (PostgreSQL, Redis, RabbitMQ, MongoDB, S3/MinIO).
3. **Detect Current Workspace Mode & Existing Tooling Configs:**
   Inspect existing root manifests (`package.json`, `pnpm-workspace.yaml`, `turbo.json`, `nx.json`, `pom.xml`, `build.gradle`, `go.mod`, `pyproject.toml`, `Cargo.toml`) and existing directories (`apps/*`, `packages/*`, `src/*`):
   - **Check for Existing Linter / Formatter Configs:** Inspect the root for `biome.json`, `.eslintrc.*`, `eslint.config.*`, `.prettierrc*`, `.editorconfig`, `ruff.toml`, or `.golangci.yml`. If detected, mark them as **Existing Config (Preserved)** so they are never overwritten.
   - **Mode A — Greenfield (No build manifests exist; only `docs/` or `.git` is present):** Full workspace initialization required.
   - **Mode B — Partial Workspace (Root exists, but one or more target apps/packages/guardrails from `docs/ARCHITECTURE.md` are missing):** Incremental scaffold of **only** the missing directories/configs.
   - **Mode C — Already Initialized (All target apps, packages, and tooling exist):** Output a clean summary confirming the workspace is already scaffolded and recommend running the **`slice`** skill directly. **Halt here.**
4. **Pre-Flight Interrogation & `docs/INITIALIZATION.md` Generation:**
   Before running any CLI commands or creating files:
   a. **Pre-Flight Interrogation:** Conduct a focused interrogation (1 question per declared surface) to lock in the minute CLI parameters that `docs/ARCHITECTURE.md` leaves open:
      - **Frontend Surface (`apps/web`):** Styling & component combo (e.g., Tailwind CSS + shadcn/ui vs. Tailwind only vs. Vanilla).
      - **Backend Surface (`apps/api`):** Underlying HTTP engine / driver (e.g., Express standard vs. Fastify high-throughput).
      - **Local Services & Ports:** Docker Compose port assignments (default 5432/6379 vs. offset 5433/6380 to prevent local port collisions).
      - **Tooling Presets:** Confirm Linter preset (Biome vs. ESLint+Prettier vs. BYOC custom config).
   b. **Generate `docs/INITIALIZATION.md`:** Write `docs/INITIALIZATION.md` (based on [`templates/docs/INITIALIZATION.template.md`](./templates/docs/INITIALIZATION.template.md)) containing:
      - Section 1: **Locked Decisions Matrix** (all exact flags, engines, ports, and packages).
      - Section 2: **Foundation Step-by-Step Execution Board** (`STEP-01` through `STEP-08`), initially marked as `[TODO]`.
      - Header Status: `🟡 AWAITING_DEVELOPER_REVIEW`.
   c. **Developer Review Gate (Strictly 1 Question):** Present `docs/INITIALIZATION.md` for review before touching disk:
      - **[1] (Recommended) Approve INITIALIZATION.md & Begin Foundation Execution:** Lock decisions, set status to `🔵 IN_PROGRESS`, and proceed to Step 1.
      - **[2] Request Adjustments:** Modify flags, styling, ports, or steps before any command is run (write-in).

---

## Foundation Execution Discipline (Strict Single-Task Loop)

The agent must **NEVER** execute multiple foundation steps simultaneously or skip ahead:
1. Open `docs/INITIALIZATION.md` and mark the active step as `[IN_PROGRESS]`.
2. Execute strictly that single foundation step (CLI command, template copy, or package creation).
3. Verify the step completed cleanly, then mark it `- [x] ... [DONE]` in `docs/INITIALIZATION.md`.
4. Proceed to the next step. If any step fails, stop immediately, report the blocker, and do not proceed to subsequent steps.

---

## Step 1: Phase 1 — Non-Interactive Official CLI Generation

Consult [references/cli-recipes.md](./references/cli-recipes.md) for the exact flags for each framework. Follow these execution rules strictly:

1. **Always Force Non-Interactive & Cross-Platform Safe Mode:**
   - Prefix commands with `CI=true` (or `$env:CI="true"` in PowerShell) and pass `--yes` / `--no-interactive` flags so the terminal never blocks waiting for user input.
   - On Windows PowerShell, always use `curl.exe` (never bare `curl`, which aliases `Invoke-WebRequest`) and avoid chaining POSIX-only flags like `mkdir -p`.
2. **Prevent Nested `.git` Repositories & Premature Installs in Monorepos:**
   - Always pass `--skip-git` (or `--no-git`) and `--skip-install` when generating sub-apps (`apps/web`, `apps/api`, `apps/admin`, `apps/mobile`).
   - Immediately verify that no sub-directory created a nested `.git` folder (if one was created by a CLI, delete the nested `apps/<name>/.git` folder so git tracks the monorepo as a single repository).
3. **Prevent Split-Brain Linter Configs in Sub-Apps:**
   - If the workspace uses **Biome** at the root (`biome.json`), pass `--biome` to `create-next-app` (never `--eslint`) and immediately delete any CLI-generated `.eslintrc.*`, `eslint.config.*`, or `.prettierrc*` files inside sub-apps (`apps/api`, `apps/admin`, etc.) so the root `biome.json` is the single source of truth.
4. **Ensure Git Is Initialized at Root:**
   - If `.git` does not exist at the repository root, run `git init` at the root.
5. **Network / Sandbox Fallback:**
   - If the environment blocks network access or a CLI generator fails due to registry restrictions, do **not** loop endlessly. Report the exact command that failed and either:
     - Ask the developer to run that single CLI command in their terminal, **or**
     - With developer consent, write a minimal, clean starter manifest (`package.json`, `tsconfig.json`, and entry point) directly.

---

## Step 2: Phase 2 — Modular Templates, Shared Packages & Glue

Official CLIs generate isolated apps. After Phase 1 completes, copy the approved presets from [`templates/`](./templates/) and write the **glue files** that connect the architecture defined in `docs/ARCHITECTURE.md`:

### 1. Root Workspace & Orchestration Configs
- **Monorepo (e.g., Turborepo / pnpm):**
  - `pnpm-workspace.yaml` listing `apps/*` and `packages/*`.
  - `turbo.json` (or `nx.json`) defining `build` (with `"dependsOn": ["^build"]`), `dev`, `lint`, `typecheck`, and `test` pipelines.
  - Root `package.json` with `"private": true`, `"packageManager"`, `"prepare": "husky"`, and unified root scripts (`pnpm dev`, `pnpm build`, `pnpm lint`, `pnpm typecheck`, `pnpm test`).
- **Polyglot / Go / Python / JVM:**
  - Root `Makefile` or task runner scripts (`go.work`, `uv` workspace `pyproject.toml`, or multi-module `pom.xml`) exposing consistent `build`, `lint`, `typecheck`, and `test` commands.

### 2. Pre-Linked Shared Packages (`@repo/*`)
If Section 5 of `docs/ARCHITECTURE.md` specifies a monorepo, create minimal compiling skeletons using the standardized **`@repo/*`** namespace and **pre-link them into `apps/*`** so `implement` never fails on Task 1 imports:
- `packages/tsconfig` (`"name": "@repo/tsconfig"`): Copy [`templates/typescript/tsconfig.base.json`](./templates/typescript/tsconfig.base.json) to `packages/tsconfig/base.json` (uses `"moduleResolution": "Bundler"` so extensionless TypeScript imports work cleanly across bundlers), plus `nextjs.json` and `node.json`.
- `packages/contracts` (`"name": "@repo/contracts"`): Shared validation library (`zod` / `valibot`) with a base `ApiResponse<T>` envelope export (`src/index.ts`).
- `packages/database` (`"name": "@repo/database"`): ORM configuration (`drizzle.config.ts` or `prisma/schema.prisma`) and database client export (`src/index.ts`) with an empty or health-check schema—**do not create feature tables yet**.
- **Pre-Link into Apps:** Add `"@repo/contracts": "workspace:*"` (and `"@repo/database": "workspace:*"` in backend apps) to the `dependencies` of `apps/*/package.json`, and `"@repo/tsconfig": "workspace:*"` to `devDependencies`.

### 3. Quality, Linting, Git Hooks & CI Guardrails (via [`templates/`](./templates/))
Copy directly from [`templates/`](./templates/) rather than inventing configs from memory:
- **Linter & Formatter Presets:**
  - **If Existing Config:** Retain existing file without modification.
  - **If Biome Preset (JS/TS):** Copy [`templates/linters/biome.json`](./templates/linters/biome.json) to `./biome.json` and [`templates/git/.lintstagedrc.biome.json`](./templates/git/.lintstagedrc.biome.json) to `./.lintstagedrc.json`. Add `@biomejs/biome` to root `devDependencies` and set `"lint": "biome check ."` in root `package.json`.
  - **If ESLint + Prettier Preset (JS/TS):** Copy [`templates/linters/eslint.config.mjs`](./templates/linters/eslint.config.mjs) to `./eslint.config.mjs`, [`templates/linters/.prettierrc`](./templates/linters/.prettierrc) to `./.prettierrc`, and [`templates/git/.lintstagedrc.eslint.json`](./templates/git/.lintstagedrc.eslint.json) to `./.lintstagedrc.json`. Add `eslint`, `prettier`, `typescript-eslint`, and `@eslint/js` to root `devDependencies` and set `"lint": "eslint ."` in root `package.json`.
  - **If Python / Go / JVM Only:** Configure `ruff` in `pyproject.toml` (Python), `.golangci.yml` (Go), or Spotless/Checkstyle in `pom.xml` (Java) matching Section 1.1 of `docs/ARCHITECTURE.md`.
  - **If BYOC:** Copy the user-specified local config file or install and extend their shared organization package (e.g., `@myorg/eslint-config`).
- **Editor Consistency:** Copy [`templates/editor/.editorconfig`](./templates/editor/.editorconfig) to `./.editorconfig`.
- **Commitlint & Husky v9 Git Hooks (if approved in Step 0):**
  - Copy [`templates/git/commitlint.config.mjs`](./templates/git/commitlint.config.mjs) to `./commitlint.config.mjs` (supports `[TASK-XX]` subjects).
  - Copy [`templates/git/husky/commit-msg`](./templates/git/husky/commit-msg) and [`templates/git/husky/pre-commit`](./templates/git/husky/pre-commit) into `./.husky/`.
  - Add `husky`, `lint-staged`, `@commitlint/cli`, and `@commitlint/config-conventional` to root `devDependencies`.
  - Ensure `"prepare": "husky"` is set in root `package.json` and run `git config core.hooksPath .husky` so Git activates the hooks immediately.
- **CI / CD Pipeline:** Copy [`templates/ci/github-ci.yml`](./templates/ci/github-ci.yml) to `./.github/workflows/ci.yml` (if the project includes Python/Go/Java services, adapt the runner steps to match the workspace commands).
- **Environment & Gitignore:**
  - Root `.gitignore` covering `node_modules/`, `dist/`, `.next/`, `.turbo/`, `.env`, `.env.*` (with `!.env.example`), `__pycache__/`, `.venv/`, `target/`.
  - `.env.example` containing placeholder keys for database URL, auth secrets, third-party SDK DSNs (e.g., `SENTRY_DSN`), and ports from `docs/ARCHITECTURE.md`.

### 4. Local Development Backing Services (`docker-compose.yml`)
If approved in Step 0, generate a lightweight `docker-compose.yml` at the repository root containing only the backing data stores specified in `docs/ARCHITECTURE.md` (e.g., `postgres:16-alpine` with healthcheck, `redis:7-alpine`, `rabbitmq:3-management-alpine`), mapped to the exact ports in `.env.example`.

### 5. Agent Context & Rules (`AGENTS.md` + `docs/rules/`)
Create the project rule files per [references/cli-recipes.md](./references/cli-recipes.md) that the `implement` skill reads on every microtask:
- Root **`AGENTS.md`**: Concise project overview, tech stack summary, exact verification commands (`typecheck`, `lint`, `test`, `build`), and links to `docs/PRD.md`, `docs/ARCHITECTURE.md`, `docs/tasks/README.md`, and `docs/rules/`.
- **`docs/rules/code-standards.md`**: Strict typing rules (no `any`), layer import boundaries, error handling conventions, and naming standards from `docs/ARCHITECTURE.md`.
- **`docs/rules/testing-integrity.md`**: Rules forbidding test deletion, `.skip`, weakened assertions, `@ts-ignore`, or `--no-verify` commits.

---

## Step 3: Phase 3 — Dependency Install, Format Normalization & Verification Loop

Once all CLI shells, `templates/` presets, and glue files are in place:

1. **Install Dependencies Once at Root:**
   - Run `pnpm install` (or `uv sync`, `go mod tidy`, `mvn test-compile`).
2. **Normalize CLI-Generated Formatting First:**
   - Official CLIs (`create-next-app`, `@nestjs/cli`, `create-vite`) generate files with inconsistent quote/indent styles. Before running strict lint verification, run one auto-format pass at the root:
     - **Biome:** `npx @biomejs/biome check --write .`
     - **Prettier:** `npx prettier --write .`
3. **Run the Deterministic Verification Checks:**
   - **Typecheck:** e.g., `pnpm typecheck` (or `pnpm -r typecheck`)
   - **Lint:** e.g., `pnpm lint`
   - **Smoke Test / Build:** e.g., `pnpm build` — Verify the starter shells and `@repo/*` packages compile cleanly.
4. **Self-Healing (Max 3 Attempts):**
   - If typecheck, lint, or build fails (e.g., a missing `@types/*` package, a tsconfig path mismatch, or an unused import in CLI boilerplate), fix the issue and re-verify.
   - Stop after **3 failed attempts** and report the exact error to the developer.

---

## Step 4: Developer Surveillance Gate & Initial Commit

Once verification passes:

1. **HALT IMMEDIATELY.** Do **not** run `git commit` yet.
2. Update `docs/INITIALIZATION.md` header status from `🔵 IN_PROGRESS` to `🟢 COMPLETED`, and confirm all steps on the board are marked `- [x] ... [DONE]`.
3. Render the **Scaffold Review Card** per [references/cli-recipes.md](./references/cli-recipes.md):
   - **Apps & Packages Created:** List each directory and how it was created (CLI command vs. `templates/` preset vs. LLM glue).
   - **Guardrails & Rules Wired:** Linter, `.editorconfig`, Commitlint, `.lintstagedrc.json`, Husky hooks, `.github/workflows/ci.yml`, `docker-compose.yml`, `AGENTS.md`, `docs/rules/`.
   - **Verification Results:** Exact commands run (`typecheck`, `lint`, `build`/`test`) and exit codes.
   - **Git Status Summary:** Confirm `.env` and `node_modules` are ignored.
   - **Proposed Commit Message:** `chore(repo): scaffold project workspace and tooling from architecture`
4. Ask the developer via the **Interactive Questioning Protocol**:
   - **[A] (Recommended) Approve & Commit, then hand off to `slice`**
   - **[B] Approve & Commit, then pause here**
   - **[C] Request adjustments (write-in)**
5. **Upon Approval (`[A]` or `[B]`):**
   - Stage the scaffolded files (verify `git status --porcelain` contains no `.env` secrets or build artifacts).
   - Commit with the approved message:
     ```bash
     git commit -m "chore(repo): scaffold project workspace and tooling from architecture"
     ```
   - Recommend running the **`slice`** skill next to decompose `FEAT-01` from `docs/PRD.md` into `docs/tasks/001-<slug>.md`.

---

## Verification Checklist

Before concluding, confirm:
- [ ] Verified `docs/ARCHITECTURE.md` exists before scaffolding.
- [ ] Checked for existing apps/packages/configs and never overwrote existing directories or files.
- [ ] Interrogated developer for surface-level minute details (styling, engine, ports, linter) and wrote `docs/INITIALIZATION.md`.
- [ ] Received explicit developer approval on `docs/INITIALIZATION.md` before executing any foundation commands.
- [ ] Executed foundation steps strictly one-by-one, updating `[IN_PROGRESS]` $\rightarrow$ `[DONE]` in `docs/INITIALIZATION.md`.
- [ ] Ran official CLI generators with non-interactive flags (`--yes`, `--skip-git`, `--skip-install`), stripped conflicting sub-app linter configs when using root Biome, and verified no nested `.git` folders remain.
- [ ] Created `@repo/*` shared packages (`@repo/tsconfig`, `@repo/contracts`, `@repo/database`) and pre-linked them via `workspace:*` into `apps/*/package.json`.
- [ ] Copied modular presets from `templates/` (`biome.json` or ESLint+Prettier, `.editorconfig`, `commitlint.config.mjs`, `.lintstagedrc.json`, Husky v9 hooks with `core.hooksPath`, `.github/workflows/ci.yml`).
- [ ] Created `AGENTS.md` and `docs/rules/` (`code-standards.md`, `testing-integrity.md`) for the `implement` skill.
- [ ] Normalized formatting on CLI-generated files and verified `typecheck`, `lint`, and `build` exit with `code: 0`.
- [ ] Wrote **zero** product feature code (`FEAT-XX`).
- [ ] Presented the **Scaffold Review Card** and committed only after developer approval.
