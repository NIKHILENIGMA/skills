---
name: nextjs-core
description: Use whenever building, refactoring, or debugging Next.js (16+ App Router) applications, pages, layouts, Server Components, Client Components, Server Actions, Route Handlers, React Hook Form + Zod forms, TanStack Query, Tailwind CSS, or shadcn/ui components.
---

# Next.js 16+ Core Architecture & Standards

You are acting as a Principal Next.js Architect. Follow these rules whenever writing or modifying Next.js code.

> **Mandatory Skill Chain:** Whenever creating or consuming backend API routes (`app/api/v1/**/route.ts`) or external NestJS endpoints, you **MUST** also read and enforce [api-contract](../api-contract/SKILL.md) and [api-contract/examples/nextjs.md](../api-contract/examples/nextjs.md).

---

## 1. Step 0: Detect Project Topology (Mode A vs. Mode B)

Inspect `package.json`, `pnpm-workspace.yaml`, `turbo.json`, `apps/`, and `packages/` before creating files:

### Mode A: Full-Stack Solo Next.js Project
Next.js owns the Drizzle database (`src/db/schema/index.ts` barrel + `src/db/migrations/`), Server Actions, and `src/app/api/v1/` routes:
```text
src/
├── app/                         # Thin routing only: (public)/, (auth)/, (dashboard)/, api/v1/
├── components/
│   ├── layout/                  # Header, Sidebar, Footer
│   └── ui/                      # shadcn/ui primitives (never hand-roll existing components)
├── db/                          # client.ts, schema/index.ts barrel, migrations/
├── features/
│   └── <feature>/               # Feature-sliced domain modules
│       ├── actions/             # Server Actions ("use server")
│       ├── components/          # Feature-specific Server & Client components
│       ├── hooks/               # TanStack Query / UI hooks
│       ├── queries/             # Server-only Drizzle data fetchers
│       └── schemas/             # Zod schemas + z.infer DTO types
├── lib/                         # env.ts (Zod/t3-env), auth.ts, utils.ts (cn)
└── middleware.ts (or proxy.ts)  # Edge route protection (Clerk / Supabase)
```

### Mode B: Monorepo (`apps/web` + External NestJS `apps/api`)
Next.js is the frontend/BFF consuming NestJS endpoints via a typed `apiClient` that unwraps `ApiResponse<T>`:
```text
├── apps/web/src/
│   ├── app/                     # Thin route groups + page.tsx, layout.tsx, loading.tsx, error.tsx
│   ├── components/              # Local layout or app-specific UI (or import from @repo/ui)
│   ├── features/<feature>/      # api/, components/, hooks/ (useQuery/useMutation), schemas/
│   └── lib/                     # api-client.ts (injects Bearer token), env.ts, utils.ts
└── packages/
    ├── types/                   # @repo/types: shared Zod schemas, DTOs, ApiResponse<T>
    └── ui/ (optional)           # @repo/ui: shared shadcn/ui components & Tailwind preset
```
- **Monorepo Rule:** Never duplicate Zod schemas or DTO interfaces in `apps/web` when `@repo/types` exists; import them directly.

---

## 2. Next.js 16+ Runtime & Rendering Rules

1. **Next.js 16+ Async Dynamic APIs (Mandatory):**
   - `params`, `searchParams`, `cookies()`, and `headers()` are **always Promises** in Next.js 16+.
   - Always `await params` and `await searchParams` in `page.tsx`, `layout.tsx`, `route.ts`, and `generateMetadata`.
2. **Server Components by Default:**
   - Every `page.tsx`, `layout.tsx`, and display component is a React Server Component unless it requires hooks, event listeners, or browser APIs.
   - Push `"use client"` down to the smallest interactive leaf component (e.g., `<OrderFilters />`, `<CreateOrderForm />`). Never mark an entire `page.tsx` as `"use client"`.
3. **Resilience Boundaries:**
   - Every data-fetching route segment should define `loading.tsx` (using shadcn `<Skeleton />`) and `error.tsx` (`"use client"` with reset button), or wrap slow sections in `<Suspense fallback={...}>`.

---

## 3. Data Fetching, State & Mutations

1. **Server-First Fetching + TanStack Query:**
   - **Initial Load:** Fetch data directly via `async/await` in Server Components (calling `queries/*.ts` in Mode A, or server-side `apiClient` in Mode B).
   - **Interactive Client Data:** Use TanStack Query (`useQuery`, `useInfiniteQuery`) in Client Components for live polling, client-side pagination, or optimistic updates. **Never** fetch data inside `useEffect`.
   - **URL State:** Store shareable filter, search, sort, and pagination state in URL query params using `nuqs` (or typed `searchParams`), not `useState`.
   - **Global UI State:** Use `zustand` strictly for ephemeral client UI state (modals, sidebar collapse, multi-step wizard drafts).
2. **Forms & Mutations (`react-hook-form` + `zod` + `shadcn` + `sonner`):**
   - Build all forms with `useForm<Input>({ resolver: zodResolver(schema) })`, shadcn `<Form>`, and `sonner` (`toast.success` / `toast.error`).
   - **Mode A (Solo Next.js):** Submit to a `"use server"` Server Action that validates input with Zod, verifies auth via `requireAuth()`, runs `async/await` Drizzle queries, calls `revalidatePath()`/`revalidateTag()`, and returns `{ success, message, data?, error? }`.
   - **Mode B (Monorepo with NestJS):** Submit via TanStack `useMutation` calling `apiClient.post<OrderDto>('/v1/orders', input)`, then invalidate queries via `queryClient.invalidateQueries()` and show `toast.success(res.message)`.

---

## 4. Two-Layer Authentication & Security

1. **Layer 1 — Edge Route Gate (`src/middleware.ts` or `src/proxy.ts`):**
   - Use Clerk (`clerkMiddleware`) or Supabase Auth middleware to redirect unauthenticated users away from protected route groups (`/(dashboard)(.*)`).
2. **Layer 2 — Data & Action Boundary (`requireAuth()`):**
   - Never rely solely on middleware. Every Server Action, server query, and `app/api` Route Handler must call `await requireAuth()` to verify the current user session before accessing data.
   - In Mode B, `apiClient` automatically attaches `Authorization: Bearer <token>` from the active Clerk/Supabase session when calling NestJS.
3. **Fail-Fast Environment Validation (`src/lib/env.ts`):**
   - Validate all `NEXT_PUBLIC_*` and server env variables using `@t3-oss/env-nextjs` or Zod at build/boot time. Never access unvalidated `process.env` in components.

---

## 5. Tailwind CSS & shadcn/ui Standards

- **Never hand-write primitives** (Button, Dialog, Select, Table, Form, Sheet, DropdownMenu) that exist in `shadcn/ui`. Install missing ones via `npx shadcn@latest add <component>`.
- **Use Semantic Design Tokens:** Use `bg-background`, `text-foreground`, `text-muted-foreground`, `border-border`, `bg-primary`, `text-destructive`. Never hardcode hex colors (`#fff`, `bg-[#123456]`).
- **Class Merging:** Always use `cn(...)` from `@/lib/utils` for conditional classes. No inline `style={{ ... }}`.
- **Responsive & Accessible:** Mobile-first breakpoints (`sm:`, `md:`, `lg:`), semantic HTML landmarks, and `next/image` + `next/font`.

---

## 6. Reference Implementations (Load On-Demand Only)

- **Next.js Route Handlers (`app/api/v1/**/route.ts`):** [../api-contract/examples/nextjs.md](../api-contract/examples/nextjs.md)
- **Next.js 16+ Server Component Page, Typed `apiClient` (Mode B), TanStack Query & `nuqs` URL State:** [examples/data-fetching-and-api-client.md](./examples/data-fetching-and-api-client.md)
- **React Hook Form + Zod + shadcn `<Form>` + Server Actions (Mode A) & Two-Layer Auth + `env.ts`:** [examples/forms-actions-and-auth.md](./examples/forms-actions-and-auth.md)

## 7. Verification Checklist
- [ ] Checked topology (Mode A Solo vs. Mode B Monorepo) and placed code in `src/features/<feature>/`.
- [ ] All Next.js 16+ `params`, `searchParams`, `cookies()`, and `headers()` are `await`ed.
- [ ] `"use client"` is restricted to leaf interactive components; zero `useEffect` data fetching.
- [ ] Forms use `react-hook-form` + `zodResolver` + shadcn `<Form>` + `sonner` toast feedback.
- [ ] Server Actions and data fetchers verify auth via `await requireAuth()`.
- [ ] `npx tsc --noEmit` passes with zero errors.
