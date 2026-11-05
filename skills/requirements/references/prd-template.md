# Product Requirements Document (PRD) Reference Template

Use this template to generate or update `docs/PRD.md`.

> **Output Constraints:**
> - **Target Length:** **150–250 lines** total.
> - **P0 Scope Limit:** Strictly **3–5 `P0 (MVP)` features** (or 2–3 per core surface in multi-app ecosystems). If more are requested, force a trade-off demotion to `P1`.
> - **Acceptance Criteria:** Format as concise **Gherkin-style bullets** (`Given ... When ... Then ...`), **maximum 3 bullets per feature**.
> - **Edge Cases:** Maximum **2 concrete constraints or failure states** per feature.

---

```markdown
# Product Requirements Document (PRD): <Product Name>

## 1. Executive Summary & Problem-Solution Fit
- **Problem Statement:** <Clear 1–2 sentence description of the pain point or inefficiency being solved>
- **Target Value Proposition:** <How this product solves that pain point across its user ecosystem>
- **Business Model Archetype:** <e.g., On-Demand O2O Service & Logistics, Multi-Vendor Marketplace, B2B SaaS, Consumer Utility, Internal Ops>

---

## 2. Actor x Surface Matrix & Scale Profile

### 2.1 Actor & Client Surface Matrix
| Actor / Persona | Client Surface | Primary Responsibility | Key Operational Need |
|---|---|---|---|
| **<e.g., Customer>** | <e.g., Mobile App + Web> | <e.g., Schedule pickup/delivery, track order, pay> | <Transparent pricing & real-time ETA> |
| **<e.g., Delivery Partner>** | <e.g., Mobile App (iOS/Android)> | <e.g., Route navigation, doorstep pickup/drop, COD collection> | <Fast dispatch, accurate location & daily settlement> |
| **<e.g., Admin / Operator>** | <e.g., Web Dashboard> | <e.g., Order state machine, hub processing, rider & payout ledger> | <Exception handling & financial reconciliation> |

### 2.2 Scale, Concurrency & Data Profile
- **Target Scale Tier:** <Case 1: 1–10k Users (Day 1 Baseline Production) | Case 2: 10k–100k+ Users (Hardened HA & Scale)>
- **Peak Traffic & Operational Shape:** <e.g., Morning/evening booking spikes, continuous GPS pings during active delivery, daily settlement batch>
- **Data & Compliance Sensitivity:** <e.g., Payment & wallet ledger integrity, location privacy, audit logs for cash handovers>

---

## 3. Master Feature Registry

Every feature is classified by strict priority:
- **`P0 (MVP)`**: Core functionality required to launch the end-to-end value loop (strictly capped at 3–5 features, or 2–3 per surface).
- **`P1 (Fast-Follow)`**: High-value enhancements scheduled immediately after MVP validation.
- **`P2 (Backlog / Future)`**: Desirable ideas deferred until post-launch demand is confirmed.

| Feature ID | Surface / Actor | Feature Name | Priority | User Story | Acceptance Criteria (Max 3 Gherkin Bullets) | Edge Cases & Constraints (Max 2) | Status |
|---|---|---|---|---|---|---|---|
| `FEAT-01` | `[Customer]` | <Core Feature 1> | **P0 (MVP)** | As a <actor>, I want <capability> so that <benefit>. | • **Given** <ctx>, **When** <action>, **Then** <outcome>.<br>• **Given** <ctx>, **When** <action>, **Then** <outcome>. | • <Constraint/Edge case 1><br>• <Constraint/Edge case 2> | `Planned` |
| `FEAT-02` | `[Rider / Field]` | <Core Feature 2> | **P0 (MVP)** | As a <actor>, I want <capability> so that <benefit>. | • **Given** <ctx>, **When** <action>, **Then** <outcome>.<br>• **Given** <ctx>, **When** <action>, **Then** <outcome>. | • <Offline / GPS drop handling> | `Planned` |
| `FEAT-03` | `[Admin Web]` | <Core Feature 3> | **P0 (MVP)** | As a <actor>, I want <capability> so that <benefit>. | • **Given** <ctx>, **When** <action>, **Then** <outcome>.<br>• **Given** <ctx>, **When** <action>, **Then** <outcome>. | • <Concurrent state update lock> | `Planned` |
| `FEAT-04` | `[Cross-Surface]` | <Payment & Settlement> | **P0 (MVP)** | As a <actor>, I want <capability> so that <benefit>. | • **Given** <ctx>, **When** <action>, **Then** <outcome>. | • <Idempotency / partial settlement> | `Planned` |
| `FEAT-05` | `[Customer]` | <Secondary Feature> | **P1 (Fast-Follow)** | As a <actor>, I want <capability> so that <benefit>. | • **Given** <ctx>, **When** <action>, **Then** <outcome>. | • <Timeout / fallback state> | `In Pipeline` |
| `FEAT-06` | `[Admin Web]` | <Future Extension> | **P2 (Backlog)** | As a <actor>, I want <capability> so that <benefit>. | • **Given** <ctx>, **When** <action>, **Then** <outcome>. | • TBD | `Backlog` |

---

## 4. Core End-to-End Workflows

### 4.1 Primary Value Loop (Creation -> Fulfillment -> Settlement)
1. **Initiation (`[Customer]`):** <How an order/request is created and scheduled>
2. **Dispatch & Field Action (`[Rider / Field]`):** <How assignment, location routing, and physical/service verification occur>
3. **Processing & State Machine (`[Admin Web]`):** <How the backoffice processes and updates lifecycle stages>
4. **Completion & Settlement (`[Cross-Surface]`):** <How final delivery, payment capture, and rider/vendor wallet settlement conclude>

---

## 5. Explicit Non-Goals (Out of Scope for v1)
- **No <Deferred Feature X>:** <Why it is excluded from MVP>
- **No <Deferred Surface/Integration Y>:** <Why it is deferred to P1/P2>

---

## 6. Success Metrics (KPIs) & Product Risks

### 6.1 Key Performance Indicators
- **Primary North Star Metric:** <e.g., Completed Orders with Verified Settlement per Week>
- **Operational Efficiency Target:** <e.g., On-time pickup/delivery SLA >95%, zero unsettled cash discrepancies>

### 6.2 Product & Operational Risks
| Risk / Uncertainty | Impact | Mitigation Strategy | Status |
|---|---|---|---|
| <e.g., COD cash leakage or delayed rider settlement> | `High` | <Enforce maximum cash-in-hand limit before rider can accept new orders> | `Open` |
| <e.g., Pricing discrepancy between booking estimate and hub inspection> | `Medium` | <Require customer approval notification before processing modified bill> | `Open` |
```
