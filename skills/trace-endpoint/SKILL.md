---
name: trace-endpoint
description: Use when asked to explain, trace or diagram how an API endpoint, server action or function works end to end (client, server, background jobs, database, external services).
---

# Trace Endpoint

Input: `METHOD /path`, a controller/handler name, or a server action name. If missing, ask for it.

## Steps
1. Locate the entry point with a targeted search, never a repo scan:
   - NestJS: grep `@Get|@Post|@Put|@Patch|@Delete` plus the path segment.
   - Next.js: `app/**/route.ts` or the `"use server"` action.
2. Find the client caller: grep the path string or action name in `app/`, `components/`, `lib/`.
3. Follow the call chain one hop at a time, reading only the functions involved:
   - Middleware, guards, pipes, interceptors, DTO validation
   - Service methods and the Drizzle queries (note tables, transactions)
   - Background work: `queue.add`, `@Processor`, `@Cron`, event emitters, webhooks
   - External calls: fetch/SDKs (payments, email, storage)
4. Stop at 3 hops of unrelated code; summarize instead of reading deeper.

## Output
1. A Mermaid `sequenceDiagram` (participants: Client, Next.js, API, Guard, Service, Queue, Worker, DB, External). Use `-->>` for responses and `alt`/`opt` for error and branch paths.
2. A table: `Step | File:line | What happens`.
3. **Not verified**: anything inferred or not visible in code (dynamic routes, runtime config).

## Rules
- Only include steps seen in code. Never invent participants.
- Keep labels short (under 6 words).
- Offer to save the result to `docs/flows/<endpoint>.md`; don't write it unasked.
