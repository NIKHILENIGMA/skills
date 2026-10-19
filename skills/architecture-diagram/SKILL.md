---
name: architecture-diagram
description: Use when asked to draw or explain the system or cloud architecture of the project (services, database, cache, queues, storage, third-party services, deployment) from the code and config.
---

# Architecture Diagram

## Steps
1. Read only these sources (skip any that don't exist):
   - `package.json` dependencies (e.g. `bullmq`, `ioredis`, `drizzle-orm`, `pg`, `@aws-sdk/*`, `stripe`, `resend`)
   - `.env.example` (reveals DB, Redis, S3, third-party keys; never read `.env`)
   - `Dockerfile`, `docker-compose.yml`, `vercel.json`, `.github/workflows/*`
   - IaC: `terraform/`, `cdk/`, `pulumi/`, `serverless.yml`, `k8s/`
   - `drizzle.config.ts`, NestJS `app.module.ts` imports
2. Identify: frontend, API, workers/cron, database, cache, queue, storage, external services, CI/CD, hosting.
3. Map connections from imports and config (e.g. API -> Redis via bullmq).

## Output
1. A Mermaid `flowchart LR`:
   - Subgraphs for Client, App (Next.js, NestJS, Workers), Data (DB, Redis, S3), External.
   - Solid arrows = confirmed in code. Dashed arrows (`-.->`) = inferred.
2. A short table: `Component | Evidence (file) | Confidence`.
3. **Unknown**: infra not defined in code (load balancers, CDN, IAM, console-configured resources). Never guess them.

## Rules
- Quote node labels with special characters: `id["Label (info)"]`.
- Use real names from config, not generic placeholders.
- Keep under ~15 nodes; group the rest.
