---
name: scaffold
description: Optional project bootstrapper that reads docs/ARCHITECTURE.md to initialize workspace manifests, app shells (via non-interactive official CLIs), shared packages, linters, test runners, Commitlint hooks, local Docker services, and AGENTS.md + docs/rules/. Does NOT implement product features.
---

# Project Workspace & Tooling Bootstrapper (`scaffold`)

You are acting as a **Principal Platform & DX (Developer Experience) Engineer**. Your single responsibility is to turn the static blueprint in **`docs/ARCHITECTURE.md`** into a working, verified project skeleton—using **non-interactive official CLI generators first** and **LLM-written glue configs second**—so that the `slice` and `implement` skills can immediately run verification commands (`tsc`, `vitest`, `biome`, `pytest`, `go test`, `mvn`, `commitlint`) on Task 1.

> **Single Responsibility & Strict Boundaries:**
> - **In Scope:** Reading `docs/ARCHITECTURE.md` (Section 1.1 7-Layer Stack Matrix & Section 5 Repository Topology), running **non-interactive official CLIs** (`create-next-app`, `@nestjs/cli`, `create-vite`, `create-expo-app`, `uv`, `go mod`, Spring Initializr, etc.), wiring monorepo/workspace manifests (`package.json`, `pnpm-workspace.yaml`, `turbo.json`, `nx.json`, `pom.xml`, `pyproject.toml`), creating shared package skeletons (`packages/database`, `packages/contracts`, `packages/tsconfig`), configuring quality guardrails (Biome/ESLint, Commitlint, Husky, `docker-compose.yml`, `.env.example`), generating `AGENTS.md` + `docs/rules/`, verifying that build/lint/typecheck pass, and halting for developer approval before the foundation commit.
> - **Strictly Out of Scope (CRITICAL):**
>   - **NO PRODUCT FEATURE CODE:** Never implement product features (`FEAT-01`, `FEAT-02`), domain database tables, business services, or feature UI screens. Only create minimal health-check / starter shells and base database connection wiring.
>   - **NEVER HANG ON INTERACTIVE CLIS:** Every CLI generator command **must** pass non-interactive flags (`--yes`, `--no-interactive`, `--skip-git`, `CI=true`). Never run a bare CLI command that waits for keyboard prompts.
>   - **NEVER OVERWRITE EXISTING APPS:** If an app or package directory already exists, do not wipe or re-scaffold it.
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
   - **Section 1.1 (7-Layer Stack Matrix):** Language/runtime, backend framework, frontend framework(s), ORM & validation library, linter/formatter, package manager/monorepo tool, and test runner + git hooks.
   - **Section 5 (Repository Topology):** Target folder tree (`apps/*`, `packages/*`, or solo `src/`).
   - **Section 7 (Operational Infrastructure):** Required local backing services (PostgreSQL, Redis, S3/MinIO).
3. **Detect Current Workspace Mode:**
   Inspect existing root manifests (`package.json`, `pnpm-workspace.yaml`, `turbo.json`, `nx.json`, `pom.xml`, `build.gradle`, `go.mod`, `pyproject.toml`, `Cargo.toml`) and existing directories (`apps/*`, `packages/*`, `src/*`):
   - **Mode A — Greenfield (No build manifests exist; only `docs/` or `.git` is present):** Full workspace initialization required.
   - **Mode B — Partial Workspace (Root exists, but one or more target apps/packages/guardrails from `docs/ARCHITECTURE.md` are missing):** Incremental scaffold of **only** the missing directories/configs.
   - **Mode C — Already Initialized (All target apps, packages, and tooling exist):** Output a clean summary confirming the workspace is already scaffolded and recommend running the **`slice`** skill directly. **Halt here.**
4. **Confirm Scaffold Scope (1 Question):**
   In Mode A or Mode B, present the detected **Scaffold Plan** (which CLI generators will run and which glue files will be written) and ask **1 confirmation question** via the **Interactive Questioning Protocol**:
   - **[1] (Recommended) Full Scaffold:** Run non-interactive CLI generators, wire workspace packages, Commitlint/Husky, `docker-compose.yml`, and `AGENTS.md` + `docs/rules/`.
   - **[2] Code & Tooling Only (Skip Docker / Git Hooks):** Scaffold apps, packages, and `AGENTS.md` + `docs/rules/`, but skip `docker-compose.yml` and Husky hooks.
   - **[3] Custom adjustments (write-in).**

---

## Step 1: Phase 1 — Non-Interactive Official CLI Generation

Consult [references/cli-recipes.md](./references/cli-recipes.md) for the exact flags for each framework. Follow these execution rules strictly:

1. **Always Force Non-Interactive Mode:**
   - Prefix commands with `CI=true` (or `$env:CI="true"` in PowerShell) and pass `--yes` / `--no-interactive` flags so the terminal never blocks waiting for user input.
2. **Prevent Nested `.git` Repositories & Premature Installs in Monorepos:**
   - Always pass `--skip-git` (or `--no-git`) and `--skip-install` when generating sub-apps (`apps/web`, `apps/api`, `apps/admin`, `apps/mobile`).
   - Immediately verify that no sub-directory created a nested `.git` folder (if one was created by a CLI, delete the nested `apps/<name>/.git` folder so git tracks the monorepo as a single repository).
3. **Ensure Git Is Initialized at Root:**
   - If `.git` does not exist at the repository root, run `git init` at the root.
4. **Network / Sandbox Fallback:**
   - If the environment blocks network access or a CLI generator fails due to registry restrictions, do **not** loop endlessly. Report the exact command that failed and either:
     - Ask the developer to run that single CLI command in their terminal, **or**
     - With developer consent, write a minimal, clean starter manifest (`package.json`, `tsconfig.json`, and entry point) directly.

---

## Step 2: Phase 2 — LLM Glue, Shared Packages & Guardrails

Official CLIs generate isolated apps. After Phase 1 completes, write the **glue and guardrail files** that connect the architecture defined in `docs/ARCHITECTURE.md`:

### 1. Root Workspace & Orchestration Configs
- **Monorepo (e.g., Turborepo / pnpm):**
  - `pnpm-workspace.yaml` listing `apps/*` and `packages/*`.
  - `turbo.json` (or `nx.json`) defining `build`, `dev`, `lint`, `typecheck`, and `test` pipelines.
  - Root `package.json` with `"private": true`, `"packageManager"`, and unified root scripts (`pnpm dev`, `pnpm build`, `pnpm lint`, `pnpm typecheck`, `pnpm test`).
- **Polyglot / Go / Python / JVM:**
  - Root `Makefile` or task runner scripts (`go.work`, `uv` workspace `pyproject.toml`, or multi-module `pom.xml`) exposing consistent `build`, `lint`, `typecheck`, and `test` commands.

### 2. Shared Packages (If Monorepo in Section 5 of `docs/ARCHITECTURE.md`)
Create minimal, compiling skeletons for the shared packages declared in `docs/ARCHITECTURE.md`:
- `packages/tsconfig`: Base `base.json`, `nextjs.json`, `node.json` configs.
- `packages/contracts`: Shared validation library (`zod` / `valibot`) with a base `ApiResponse<T>` envelope export (`src/index.ts`).
- `packages/database`: ORM configuration (`drizzle.config.ts` or `prisma/schema.prisma`) and database client export (`src/index.ts`) with an empty or health-check schema—**do not create feature tables yet**.

### 3. Quality, Linting & Commitlint Guardrails
- **Linter / Formatter:** `biome.json` (or `.eslintrc.*` + `.prettierrc`) at root.
- **Commitlint:** `commitlint.config.mjs` (or `.commitlintrc.json`) configured for Conventional Commits, compatible with `[TASK-XX]` subjects used by the `implement` skill.
- **Git Hooks (if approved in Step 0):** `.husky/commit-msg` (`npx --no -- commitlint --edit $1`) and `.husky/pre-commit`.
- **Environment & Gitignore:**
  - Root `.gitignore` covering `node_modules/`, `dist/`, `.next/`, `.turbo/`, `.env`, `.env.*` (with `!.env.example`), `__pycache__/`, `.venv/`, `target/`.
  - `.env.example` containing placeholder keys for database URL, auth secrets, and ports from `docs/ARCHITECTURE.md`.

### 4. Local Development Backing Services (`docker-compose.yml`)
If approved in Step 0, generate a lightweight `docker-compose.yml` at the repository root containing only the backing data stores specified in `docs/ARCHITECTURE.md` (e.g., `postgres:16-alpine` with healthcheck, `redis:7-alpine`), mapped to the exact ports in `.env.example`.

### 5. Agent Context & Rules (`AGENTS.md` + `docs/rules/`)
Create the project rule files that the `implement` skill reads on every microtask:
- Root **`AGENTS.md`**: Concise project overview, tech stack summary, exact verification commands (`typecheck`, `lint`, `test`), and links to `docs/PRD.md`, `docs/ARCHITECTURE.md`, `docs/tasks/README.md`, and `docs/rules/`.
- **`docs/rules/code-standards.md`**: Strict typing rules (no `any`), layer import boundaries, error handling conventions, and naming standards from `docs/ARCHITECTURE.md`.
- **`docs/rules/testing-integrity.md`**: Rules forbidding test deletion, `.skip`, weakened assertions, `@ts-ignore`, or `--no-verify` commits.

---

## Step 3: Phase 3 — Dependency Install & Verification Loop

Once all CLI shells and glue files are in place:

1. **Install Dependencies Once at Root:**
   - Run `pnpm install` (or `uv sync`, `go mod tidy`, `mvn test-compile`).
2. **Run the Deterministic Verification Checks:**
   - **Typecheck:** e.g., `pnpm -r typecheck` or `pnpm tsc --noEmit`
   - **Lint:** e.g., `pnpm lint` or `npx @biomejs/biome check .`
   - **Smoke Test / Build:** Verify the starter shells compile cleanly.
3. **Self-Healing (Max 3 Attempts):**
   - If typecheck, lint, or build fails (e.g., a missing `@types/*` package, a tsconfig path mismatch, or a lint formatting issue in CLI-generated files), fix the config or run the formatter (`biome check --write .`) and re-verify.
   - Stop after **3 failed attempts** and report the exact error to the developer.

---

## Step 4: Developer Surveillance Gate & Initial Commit

Once verification passes:

1. **HALT IMMEDIATELY.** Do **not** run `git commit` yet.
2. Render the **Scaffold Review Card** per [references/cli-recipes.md](./references/cli-recipes.md):
   - **Apps & Packages Created:** List each directory and how it was created (CLI command vs. LLM glue).
   - **Guardrails & Rules Wired:** Linter, Commitlint, `docker-compose.yml`, `AGENTS.md`, `docs/rules/`.
   - **Verification Results:** Exact commands run (`typecheck`, `lint`, `build`/`test`) and exit codes.
   - **Git Status Summary:** Confirm `.env` and `node_modules` are ignored.
   - **Proposed Commit Message:** `chore(repo): scaffold project workspace and tooling from architecture`
3. Ask the developer via the **Interactive Questioning Protocol**:
   - **[A] (Recommended) Approve & Commit, then hand off to `slice`**
   - **[B] Approve & Commit, then pause here**
   - **[C] Request adjustments (write-in)**
4. **Upon Approval (`[A]` or `[B]`):**
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
- [ ] Checked for existing apps/packages and never overwrote existing directories.
- [ ] Ran official CLI generators with non-interactive flags (`--yes`, `--skip-git`, `--skip-install`) and verified no nested `.git` folders remain.
- [ ] Wired root workspace config, shared package skeletons, linter, Commitlint, `.gitignore`, and `.env.example`.
- [ ] Created `AGENTS.md` and `docs/rules/` (`code-standards.md`, `testing-integrity.md`) for the `implement` skill.
- [ ] Installed dependencies and verified `typecheck` and `lint` exit with `code: 0`.
- [ ] Wrote **zero** product feature code (`FEAT-XX`).
- [ ] Presented the **Scaffold Review Card** and committed only after developer approval.
