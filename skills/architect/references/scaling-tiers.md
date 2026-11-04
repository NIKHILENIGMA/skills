# Architecture Scaling Stages & Operational Infrastructure

Use this reference to map an application's scale requirements (derived from `docs/PRD.md`) into a resilient, production-ready operational infrastructure.

---

## Case 1: 1 to 10,000 Users (Day 1 Production Viability)

At this stage, the primary goals are:
1. **Low operating costs** ($0 to $25/month).
2. **Zero accidental outages** (preventing connection exhaustion and server crashes from file uploads).
3. **Automated deployment and observability** without managing Kubernetes.

### 1. Hosting & Compute Topology
- **PaaS (Vercel / Netlify + Render / Railway):**
  - *When to use:* Rapid prototyping, solo developers, or teams that want zero server administration.
  - *Pattern:* Deploy frontend to Vercel/Netlify; deploy backend API containers to Render or Railway with automatic branch previews.
- **Cost-Optimized VPS (Hostinger / Hetzner / DigitalOcean — $5 to $15/mo):**
  - *When to use:* Teams wanting predictable costs and full infrastructure ownership.
  - *Capacity:* A single $10 VPS (2 vCPUs, 4GB RAM) running Docker Compose (Backend + Redis + Nginx) comfortably handles 500+ HTTP requests/sec (tens of thousands of daily users) when configured properly.

### 2. Reverse Proxy & Edge Termination: Nginx or Caddy
- **Rule:** Never expose your application container port (e.g. `localhost:3000` or `:8080`) directly to the public internet.
- **Responsibility:** Sits on ports 80/443 handling SSL/TLS termination (automated Let's Encrypt certificates), Gzip/Brotli compression, connection rate limiting, and internal reverse proxying to backend containers.

### 3. Direct Object Storage & CDN: S3 / Cloudflare R2 / CloudFront
- **Rule:** Never upload user media (2MB–10MB photos, invoices, PDFs) directly through backend server memory.
- **Pattern (Presigned Direct Uploads):**
  1. Client asks backend for an upload ticket (`POST /v1/uploads/presign`).
  2. Backend generates a cryptographically signed, short-lived Pre-Signed S3/R2 URL.
  3. Client uploads the binary payload directly to S3/Cloudflare R2, bypassing backend CPU and memory entirely.
  4. Client notifies backend upon upload completion.

### 4. Database Connection Pooler: PgBouncer / Supavisor / RDS Proxy
- **The Problem:** Default PostgreSQL allocates ~5–10MB of RAM per connection and caps at ~100 connections. Serverless lambdas, Next.js API routes, or multi-threaded workers can exhaust connection limits within seconds (`too many clients already`).
- **The Solution:** A connection pooler (PgBouncer, Supavisor on Supabase, or AWS RDS Proxy) maintains a warm pool of 15–25 connections to PostgreSQL and multiplexes thousands of incoming client requests using transaction-level pooling.

### 5. Error Tracking & Day 1 Observability: Sentry & Uptime Kuma
- **Sentry (or self-hosted GlitchTip):** Captures unhandled client and server exceptions with full stack traces, breadcrumbs, user IDs, and environment tags.
- **Uptime Kuma (free self-hosted) or Better Stack:** Pings the `/health` endpoint every 60 seconds and alerts Discord/Slack/Telegram the moment disk space exhausts or Nginx crashes.

### 6. Secrets Management: Infisical or Doppler
- **Rule:** Never commit `.env` files to git or share production credentials in Slack.
- **Pattern:** Centralize encrypted production keys in Infisical or Doppler. Inject secrets directly into CI/CD build environments and runtime containers.

### 7. Transactional Communication: Resend / Postmark / Amazon SES
- **Rule:** Never send emails via local VPS `sendmail` / `postfix` (IP addresses get blacklisted within 24 hours).
- **Pattern:** Use dedicated transactional delivery engines (Resend, Postmark, or Amazon SES) with configured SPF, DKIM, and DMARC records to guarantee inbox placement.

### 8. CI/CD Automation: GitHub Actions / GitLab CI
- **Rule:** Never SSH into production, run `git pull`, and compile TypeScript/Docker on the live server (causes 100% CPU spikes and drops active user connections).
- **Pattern:** Build and test Docker images in GitHub Actions, push to a registry (GHCR / Docker Hub), and trigger a zero-downtime rolling container replacement (`docker compose up -d --no-deps backend`).

---

## Case 2: 10,000 to 100,000 Users (Protecting Database & Latency)

At this stage, traffic volume increases, write throughput grows, and the database becomes the primary bottleneck. The architecture shifts from single-box deployment to distributed state, async queues, and read replication.

### 1. In-Memory Caching: Redis
- **Responsibility:** Sits in front of PostgreSQL to cache read-heavy catalog items, authenticated session tokens, rate-limit buckets, and high-frequency GPS or status updates. Drops PostgreSQL CPU back to safe thresholds.

### 2. Message Brokers & Queuing: Task Queue vs. Event Streaming
| Category | Technology | Operational Pattern | Ideal Use Case |
|---|---|---|---|
| **Task Queue (Post Office)** | **Redis BullMQ / RabbitMQ** | Jobs are enqueued, executed once by a worker, and deleted upon acknowledgment (ACK). Supports retries, delays, and Dead Letter Queues (DLQ). | Transactional emails, PDF generation, image resizing, webhooks (`Customer orders -> queue job -> send WhatsApp invoice`). |
| **Distributed Event Log (Time Machine)** | **Apache Kafka / Redpanda** | Sequential, append-only log written to disk and retained for 7–30 days. Multiple independent consumer groups can read and replay the same event stream. | High-throughput event streams (>10k events/sec). e.g., 50,000 scooters emit GPS pings every 3s consumed simultaneously by ETA, Fraud, Surge Pricing, and Analytics. |

### 3. Database Read Scaling: Primary + Streaming Read Replicas
- **The Insight:** 85%–95% of database operations in standard SaaS/e-commerce are reads (fetching feeds, browsing products, checking order status).
- **Pattern:** Deploy PostgreSQL Streaming Replication with 1 Primary (accepts WRITES) and 1–2 Read Replicas (accept READS). Configure connection poolers or ORMs to route `SELECT` queries to replicas, freeing primary IOPS for ACID write transactions.

### 4. Dedicated Full-Text Search: Meilisearch / Typesense vs. Elasticsearch
- **The Problem:** Running `WHERE name ILIKE '%query%'` or complex multi-column regex across millions of rows causes sequential table scans, locking DB CPUs.
- **Pattern:** Sync product/catalog data to lightweight search engines (Meilisearch or Typesense). Delivers instant typo-tolerant search, facet filtering, and rank ordering with sub-10ms latency using minimal RAM.

### 5. Durable Execution & Workflows: Temporal.io / Inngest / Trigger.dev
- **The Problem:** Multi-step business processes spanning minutes, hours, or days (e.g., *Charge card -> reserve inventory -> wait 10m for driver acceptance -> re-assign if declined -> cancel & refund if no driver in 30m*). Writing this with cron jobs and DB status flags creates fragile state machines that break on container restarts.
- **Pattern:** Use durable workflow engines (Temporal, Inngest, or Trigger.dev) where function state is persisted across server reboots, retries are automatic, and delays/sleeps are first-class language primitives.

### 6. APM & Centralized Metrics: Prometheus + Grafana (or Datadog)
- **Metrics:** Prometheus scrapes system metrics (CPU, RAM, DB connection pool wait times) and HTTP p95/p99 latency, visualized on Grafana dashboards.
- **Log Aggregation:** Stream container logs via Vector or Promtail to Grafana Loki, allowing instant cross-service log queries during outages.

### 7. Edge Security & Web Application Firewall (WAF): Cloudflare WAF
- **Pattern:** Place Cloudflare WAF in front of domain DNS. Enforce IP-based rate limiting on sensitive routes (`/api/v1/auth/*`), DDoS mitigation, and Turnstile bot challenges before malicious traffic reaches your origin servers.

### 8. Continuous Backups & Point-in-Time Recovery (PITR): pgBackRest / WAL-G
- **Rule:** Daily `pg_dump` is insufficient for production databases (takes hours, locks tables, and losing a DB at 4 PM causes 16 hours of lost customer data).
- **Pattern:** Continuously stream Write-Ahead Logs (WAL) to S3/Cloudflare R2 using `pgBackRest` or `WAL-G`. Allows restoring the database to the exact second prior to an accidental `DROP TABLE` or corrupt migration.
