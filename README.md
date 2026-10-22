# Playbook — Agentic Development Skills

A curated collection of modular, token-efficient **AI Agent Skills** designed for building, architecting, and documenting production-grade applications with **TypeScript, Next.js (16+ App Router), NestJS, Drizzle ORM, Zod, Tailwind CSS, and shadcn/ui**.

---

## Why This Playbook?

1. **Progressive Disclosure (Token-Efficient):** Only each skill's `name` and 1-sentence `description` are loaded into context up front. Detailed runbooks (`SKILL.md`), framework code (`examples/`), and lookup tables (`references/`) are loaded **on demand** only when relevant.
2. **Automatic Skill Chaining:** Framework skills (`nestjs-core`, `nextjs-core`) automatically chain to `api-contract` whenever HTTP endpoints, DTOs, or API clients are touched—guaranteeing uniform validation and JSON envelopes across the stack.
3. **Topology-Aware (Solo vs. Monorepo):** Skills inspect your workspace first (`package.json`, `pnpm-workspace.yaml`, `turbo.json`, `apps/`, `packages/`) and adapt between standalone repositories and multi-package monorepos (`@repo/db`, `@repo/types`, `@repo/ui`).

---

## Repository Structure

```text
├── .gitignore
├── LICENSE
├── package.json
├── README.md
└── skills/
    ├── product-planner/                     # 0 -> 1 Collaborative product & system architecture discovery
    │   ├── SKILL.md
    │   └── references/
    │       └── doc-templates.md             # Templates for PRD.md, ARCHITECTURE.md, adr/ & tasks/README.md
    ├── feature-planner/                     # 1 -> N Feature-to-microtask planner & step-by-step executor (1-3 files cap)
    │   ├── SKILL.md
    │   └── references/
    │       └── task-template.md             # Templates for docs/tasks/README.md & docs/tasks/00X-<slug>.md
    ├── api-contract/                        # Framework-agnostic REST API & envelope standard
    │   ├── SKILL.md
    │   ├── examples/
    │   │   ├── nestjs.md                    # NestJS Zod pipe, controller, service & exception filter
    │   │   └── nextjs.md                    # Next.js route.ts, Zod validation & error mapper
    │   └── references/
    │       └── error-codes.md               # Standard HTTP status -> UPPER_SNAKE_CASE error codes
    ├── nestjs-core/                         # Principal NestJS architecture, DI, Drizzle, queues & ops
    │   ├── SKILL.md
    │   └── examples/
    │       ├── module-and-di.md             # Solo/Monorepo DB wiring, ExternalAuthGuard & decorators
    │       └── queues-cache-and-ops.md      # BullMQ, EventEmitter, Cron, Redis cache, Throttler & health
    ├── nextjs-core/                         # Next.js 16+ App Router, RSC, TanStack Query, RHF & shadcn
    │   ├── SKILL.md
    │   └── examples/
    │       ├── data-fetching-and-api-client.md # Typed apiClient, async RSC params & nuqs URL state
    │       └── forms-actions-and-auth.md    # RHF + Zod + shadcn Form, Server Actions & 2-layer auth
    ├── trace-endpoint/                      # End-to-end API & background job flow tracer
    │   └── SKILL.md
    └── architecture-diagram/                # Code-driven system & cloud architecture diagrammer
        └── SKILL.md
```

---

## Skills Catalog

| Skill | Scope | When It Activates | Example Prompt |
|---|---|---|---|
| **[`product-planner`](./skills/product-planner/SKILL.md)** | New (`0 → 1`) & Legacy Project Blueprint Onboarding | Starting a new project OR onboarding a legacy project missing `PRD.md`, `ARCHITECTURE.md`, `adr/`, or `tasks/README.md` | *"I want to build a multi-tenant B2B invoicing SaaS—help me plan it."* / *"Onboard this existing repo into `docs/`."* |
| **[`feature-planner`](./skills/feature-planner/SKILL.md)** | 1 → N Feature-to-Microtask Decomposition | Breaking a feature/task into 1–3 file atomic microtasks in `docs/tasks/00X-<slug>.md` (tracking start, end & next steps) | *"Plan and break `FEAT-01` into microtasks in `docs/tasks/` so we can build it step-by-step."* |
| **[`api-contract`](./skills/api-contract/SKILL.md)** | Any Backend Framework | Creating, updating, or reviewing REST endpoints, DTOs, validation schemas, or response envelopes | *"Create a `POST /v1/orders` endpoint with pagination on `GET`."* |
| **[`nestjs-core`](./skills/nestjs-core/SKILL.md)** | NestJS + Drizzle | Building NestJS modules, controllers, services, guards, decorators, Drizzle transactions, queues, or caching | *"Create an `OrdersModule` with stock reservation in a Drizzle transaction and a receipt queue job."* |
| **[`nextjs-core`](./skills/nextjs-core/SKILL.md)** | Next.js 16+ + UI | Building App Router pages, Server/Client Components, Server Actions, RHF + Zod forms, TanStack Query, or shadcn/ui | *"Build the `/dashboard/orders` page with URL search filters and a create order dialog."* |
| **[`trace-endpoint`](./skills/trace-endpoint/SKILL.md)** | Codebase Analysis | Explaining or diagramming how an endpoint or Server Action executes from client to DB and background workers | *"Trace `POST /v1/orders` end-to-end and draw the sequence diagram."* |
| **[`architecture-diagram`](./skills/architecture-diagram/SKILL.md)** | Infrastructure & Config | Drawing the system or cloud architecture from `package.json`, `.env.example`, Docker, CI, and IaC files | *"Draw the system and cloud architecture diagram for this repo."* |

---

## Installation & Setup

Install these skills into any project using the [`skills`](https://www.npmjs.com/package/skills) CLI:

### 1. Install All Skills into a Project
Downloads all skills into your project's `.agents/skills/` directory:

```bash
npx skills@latest add nikhilenigma/skills
```

### 2. Install Specific Skills Only (`--skill`)
Install only the skills needed for a particular repository:

```bash
# Install only backend skills
npx skills@latest add nikhilenigma/skills --skill api-contract --skill nestjs-core

# Install only the product planner
npx skills@latest add nikhilenigma/skills --skill product-planner
```

### 3. Global Installation (`-g`)
Install the skills globally so your agent can use them across every project on your machine:

```bash
npx skills@latest add nikhilenigma/skills -g
```

### 4. Target a Specific Agent (`-a`)
By default, skills install into `.agents/skills/`. You can also target specific agents such as Antigravity, Cursor, or Claude Code:

```bash
npx skills@latest add nikhilenigma/skills -a antigravity
npx skills@latest add nikhilenigma/skills -a claude-code
npx skills@latest add nikhilenigma/skills -a cursor
```

---

## How to Use the Skills

### 1. Automatic Activation (Default)
You do **not** need to type special commands. Simply describe your task naturally—the agent reads the skill descriptions and loads the matching skill(s) automatically:
```text
Create a PATCH /v1/products/:id endpoint that invalidates the Redis product cache.
```
*The agent automatically loads `nestjs-core` + `api-contract`, validates `:id` and body with Zod, updates via `async/await` in `ProductsService`, invalidates `product:<id>` in Redis, and returns `{ success: true, message, data, timestamp }`.*

### 2. Explicit Invocation
You can also name a skill or reference its path directly in your prompt:
```text
Use product-planner to help me architect a real-time fleet tracking platform.
```

---

## Detailed Skill Guide

### 1. `product-planner` — Product & System Architecture Discovery (New & Legacy Projects)
Turns a raw project idea—or an existing pre-`agy` codebase—into a complete specification blueprint in `docs/` without wasting tokens.
- **Step 0 Smart `docs/` Discovery Gate (New vs. Legacy):**
  - Checks if `docs/` exists and specifically checks for the 4 blueprint files (`docs/PRD.md`, `docs/ARCHITECTURE.md`, `docs/adr/README.md`, `docs/tasks/README.md`):
    1. **All 4 exist:** Generates nothing and asks if you want to switch to `feature-planner` or update a specific section.
    2. **Legacy / Existing Repo missing blueprint files** (even if `docs/` has other system docs): Preserves existing `docs/` files, infers your stack from lightweight manifests (`package.json`, `go.mod`, `pom.xml`) without scanning the whole repo, skips settled stack questions, and asks which missing blueprint files you want generated.
    3. **Brand-New Repo (`0 → 1`):** Runs full scope-aligned discovery (**Frontend-Only**, **Backend-Only**, **App-Code Only**, or **Full-Stack + Cloud**).
- **How it works:** Collaborates with the developer through multi-turn sub-questions (`ask_question`) across the relevant discovery domains:
  1. Target Users, Scale & **Monthly Budget Constraint**
  2. **Collaborative Feature Discovery & Pipeline Prioritization** (`P0 MVP` vs. `P1 In-Pipeline` vs. `P2 Backlog`)
  3. **Platform & Language Ecosystem** (Web, Expo/Mobile, Tauri/Electron Desktop, CLI, API across TS, Go, Java Spring Boot, Python, Rust)
  4. **Granular Frameworks, State, UI Shell & Repo Topology** (SWR vs. TanStack Query, Express vs. NestJS/Spring Boot, Sidebar/Top-Nav layout shell, Solo vs. Nx/Turborepo)
  5. **Build vs. Buy Integrations** (Custom Auth vs. Better Auth/Clerk, Razorpay/Cashfree/PayU/Paytm/Stripe, Resend, Sentry/Grafana, OpenAI/Gemini)
  6. **Infrastructure, Redis, Load Balancing & CI/CD** (VPS/Hostinger + Coolify vs. AWS/GCP/Vercel, Redis now vs. later, Nginx vs. AWS ALB/CloudFront, GitHub Actions vs. Webhooks)
  7. **Summary Endpoints, UI Navigation, Testing/Seed Data & Handoff Confirmation**
- **Outputs generated in `docs/` (Zero Duplicate Files):**
  - `docs/PRD.md` — Personas, scale targets, monthly budget envelope, **Master Feature Registry (`P0 MVP`, `P1 In-Pipeline`, `P2 Backlog`)**, user stories, acceptance criteria, and non-goals.
  - `docs/ARCHITECTURE.md` — Mermaid `flowchart LR` (scope-aligned architecture), `erDiagram` + **Table Specifications, Enums, Indexes & FK Cascade Table**, summary API endpoint list, UI Layout Shell + **UI page navigation diagram** (`flowchart TD`), `sequenceDiagram`, and stack matrix.
  - `docs/tasks/README.md` — Master Task & Phase Index (`001-*.md`, `002-*.md`...) tracking every feature's phase, priority, status, start/end dates, and individual task file link.
  - `docs/adr/README.md` + `docs/adr/001-<slug>.md`, ... — Architecture Decision Records.
  - Optional root `AGENTS.md` & `.env.example` (if confirmed by the developer).

---

### 2. `feature-planner` — 1 → N Feature-to-Microtask Decomposition & Step-by-Step Executor
Prevents the #1 failure mode of AI coding agents—editing 10–12 files simultaneously—by decomposing **one feature/task at a time** into its own numbered file in `docs/tasks/00X-<slug>.md` with sequential, verifiable **1–3 file microtasks**.
- **When to use:** Building any planned task/feature from `docs/tasks/README.md` (or `docs/PRD.md`), promoting a `P1 In-Pipeline` feature, or adding a brand-new feature to an existing/legacy project.
- **How it works:**
  1. Inspects `docs/tasks/README.md` (creating it if missing in a legacy repo), `PRD.md`, `ARCHITECTURE.md`, and 2–4 relevant codebase files.
  2. Asks 2–4 focused feature questions (`ask_question`) on business rules, edge cases, exact API payload schemas, and UI Loading/Empty/Error states.
  3. Generates **`docs/tasks/00X-<slug>.md`** containing **Where This Feature Starts**, exact JSON request/response contracts, UI edge-state matrix, sequential microtasks (`TASK-01`, `TASK-02`... capped at **1–3 target files + an immediate verification command**), and **Where We Ended & Future Extension Notes**.
  4. **Pauses for developer review**, then executes **strictly 1 microtask at a time**—verifying and marking `[x]` before moving to the next microtask, and finally marking `00X-<slug>.md` as `Completed` in `docs/tasks/README.md`.

---

### 3. `api-contract` — Unified REST API Standard (Framework-Agnostic)
Enforces a strict 5-stage execution pipeline across **any** backend framework (NestJS, Next.js Route Handlers, Express, FastAPI, Spring Boot):
1. **Sanitize & Validate All 3 Inputs (Zod Default):**
   - **Route Params:** `uuid`, `cuid`, `slug`, or `numericId` schemas.
   - **Query Params:** Coerced `page`/`limit` bounds (`1..100`), trimmed `search`/`sortBy`, `sortOrder` (`asc` | `desc`).
   - **Body Payload:** `z.object({...}).strict()`, `.trim()` strings, `.toLowerCase()` emails/slugs.
2. **Strict Input & Output DTO Boundaries:** `z.infer` input types passed from controller to service; explicit Output DTO mapping (`toDto`) so sensitive DB columns never leak.
3. **Thin Controller / Isolated Service:** Zero business logic or DB queries in controllers; all `async/await` DB work lives in the service (`Promise.all` only for parallel independent reads).
4. **Uniform JSON Envelopes (Always Includes `message`):**
   ```json
   // Success (200 / 201)
   {
     "success": true,
     "message": "Orders fetched successfully",
     "data": [],
     "meta": { "page": 1, "limit": 20, "total": 42, "totalPages": 3 },
     "timestamp": "2026-10-20T00:00:00.000Z"
   }

   // Error (4xx / 5xx)
   {
     "success": false,
     "message": "Invalid request input",
     "error": {
       "code": "VALIDATION_FAILED",
       "details": [{ "field": "email", "issue": "Invalid email format" }]
     },
     "timestamp": "2026-10-20T00:00:00.000Z",
     "path": "/v1/orders"
   }
   ```

---

### 4. `nestjs-core` — Production NestJS & Drizzle Architecture
Chains with `api-contract` and enforces:
- **Topology Detection:** Adapts between **Solo Repo** (`src/db/schema/index.ts` barrel + `src/db/migrations/`) and **Monorepo** (`apps/api` wrapping `@repo/db` and `@repo/types`).
- **Lifecycle Placement:** `CorrelationIdMiddleware` → `ExternalAuthGuard` (Clerk/Supabase/Auth0 token verification + webhook/idempotent Drizzle user upsert) & `RolesGuard` → `ZodValidationPipe` → Controller → Service (`db.transaction()`) → `ApiExceptionFilter`.
- **Custom Decorators:** `@Public()`, `@Roles()`, `@CurrentUser()`, and composed `@Auth()`.
- **Two-Tier Async & Caching:** `@nestjs/event-emitter` for fast in-process events, `@nestjs/bullmq` (with retries, exponential backoff, and idempotent `jobId`) for heavy jobs, `@nestjs/schedule` for cron, global `ThrottlerGuard`, and Redis cache-aside with explicit mutation invalidation.
- **Ops Hardening:** Bans `console.log` (enforces Nest `Logger`/`nestjs-pino`), configures `helmet()`, strict CORS, `rawBody: true`, `enableShutdownHooks()`, and `@Public() @Get('v1/health')`.

---

### 5. `nextjs-core` — Next.js 16+ App Router, RSC & UI Standard
Chains with `api-contract` and enforces:
- **Topology Detection:** Supports **Mode A** (Full-stack Next.js with Drizzle & Server Actions) and **Mode B** (Monorepo `apps/web` calling NestJS via a typed `apiRequest<T>()` client that injects Bearer tokens and unwraps `ApiResponse<T>`).
- **Feature-Sliced Structure:** Thin `src/app/` route groups (`(public)`, `(auth)`, `(dashboard)`) + `src/features/<feature>/{components,hooks,actions,queries,schemas}`.
- **Next.js 16+ Async Dynamic APIs:** Always `await params`, `await searchParams`, `await cookies()`, and `await headers()`.
- **Server-First + TanStack Query + `nuqs`:** Server Components by default; leaf `"use client"` components; TanStack Query (`useQuery`/`useMutation`) for interactive client fetching (never `useEffect` + `fetch`); `nuqs` for URL filter/pagination state; `zustand` for ephemeral UI state.
- **Forms & Two-Layer Auth:** `react-hook-form` + `zodResolver` + shadcn `<Form>` + `sonner` toasts; edge `middleware.ts` (`proxy.ts`) route gate **plus** mandatory `await requireAuth()` inside every Server Action and query.

---

### 6. `trace-endpoint` & `architecture-diagram` — Visual Code Intelligence
- **`trace-endpoint`:** Traces any `METHOD /path` or Server Action hop-by-hop through client callers, guards, pipes, services, Drizzle queries, BullMQ queues, and external SDKs. Outputs a Mermaid `sequenceDiagram`, a `Step | File:line | What happens` table, and a **Not verified** list.
- **`architecture-diagram`:** Inspects `package.json`, `.env.example` (never `.env`), Dockerfiles, CI workflows, and IaC to output a Mermaid `flowchart LR` (solid arrows = confirmed in code, dashed arrows = inferred) with a component confidence table.

---

## Recommended End-to-End Workflow

```mermaid
flowchart LR
  A["1. Raw Idea"] -->|product-planner| B["docs/PRD.md + ARCHITECTURE.md +\nadr/ + docs/tasks/README.md"]
  B -->|feature-planner| FT["docs/tasks/00X-<slug>.md\n(1-3 File Microtasks)"]
  FT -->|nestjs-core + api-contract| C["Backend Modules, Drizzle DB & Queues"]
  FT -->|nextjs-core + api-contract| D["Next.js 16+ Pages, Forms & API Client"]
  C & D -->|trace-endpoint & architecture-diagram| E["Living Sequence & System Diagrams"]
```

---

## Optional: Guaranteed Routing with `AGENTS.md`

To guarantee 100% skill activation in any repository, drop a lightweight `AGENTS.md` file at your project root:

```markdown
# Project Agent Rules & Skill Router

## Core Rules
- Never scan the entire repository; use targeted searches and read only relevant files.
- Never edit 10-12 files simultaneously: decompose features via `feature-planner` into 1-3 file microtasks (`docs/tasks/00X-<slug>.md`) and verify each microtask before moving on.
- Strict TypeScript (`no any`). Default to Zod for input sanitization and validation.
- Never read or commit `.env` secrets or run destructive DB migrations without approval.

## Mandatory Skill Routing
Before writing code or docs, read and follow the matching skill in `.agents/skills/`:
- Starting a new project from scratch (0 -> 1 architecture & PRD) -> `product-planner`
- Planning, adding, or building a specific feature into 1-3 file microtasks -> `feature-planner`
- Creating or modifying any API endpoint, DTO, or response -> `api-contract`
- Working in NestJS backend code or Drizzle services -> `nestjs-core` (and `api-contract`)
- Working in Next.js pages, components, forms, or actions -> `nextjs-core`
- Tracing how an endpoint or action works end-to-end -> `trace-endpoint`
- Drawing system or cloud architecture diagrams -> `architecture-diagram`
```
