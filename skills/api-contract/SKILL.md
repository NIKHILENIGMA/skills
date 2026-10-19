---
name: api-contract
description: Use whenever creating, updating, or reviewing backend API endpoints, routes, controllers, services, DTOs, or response envelopes in any framework. Enforces input sanitization/validation, controller-service isolation, async/await DB handling, error handling, and unified JSON envelopes.
---

# API Contract & Endpoint Execution Flow

You are acting as the Lead API Architect. Enforce this contract and execution flow across **any** backend framework (NestJS, Next.js Route Handlers, Express, FastAPI, Spring Boot, etc.). Never return raw database records or unstructured JSON payloads.

---

## 1. End-to-End Request Flow (Mandatory Architecture)

Every endpoint must follow this strict 5-stage pipeline:

```text
[1. Sanitize & Validate Inputs (Param / Query / Body)]
        ↓ (Typed Input DTOs)
[2. Thin Controller / Route Handler (HTTP & Status Codes Only)]
        ↓ (Calls Service with Typed Input)
[3. Isolated Service Layer (Business Logic & async/await DB Queries)]
        ↓ (Maps DB Entity -> Output DTO, strips sensitive fields)
[4. Error Handling (Domain/DB errors -> Standard ApiErrorResponse)]
        ↓
[5. Unified Envelope Response (ok / paginated / fail)]
```

### Stage 1: Input Sanitization & Validation (Zod Default)
- **Default to Zod** unless the user or existing codebase specifies another validator (`class-validator`, Joi, Pydantic, etc.).
- Validate all **three** input sources separately at the boundary:
  1. **Route Parameters (`params`):** Validate according to the ID/key type:
     - UUID: `z.string().uuid()`
     - CUID / CUID2: `z.string().cuid()` or `z.string().cuid2()`
     - Numeric ID: `z.coerce.number().int().positive()`
     - String / Slug: `z.string().trim().min(1).max(120)`
  2. **Query Parameters (`query`):** Coerce numbers/booleans (`z.coerce.number()`), `.trim()` search strings, enforce `min`/`max` bounds (`page >= 1`, `1 <= limit <= 100`), and provide defaults.
  3. **Body Payload (`body`):** Use `z.object({...}).strict()` (or strip unknown keys), `.trim()` all free-text strings, `.toLowerCase()` emails/slugs, and validate nested structures.

### Stage 2: Strict Type & DTO Boundaries
- **Input Types:** Derive input types directly from schemas (`type CreateOrderInput = z.infer<typeof createOrderSchema>`) and pass those exact types into both Controller and Service methods. Never pass `any`, `unknown`, or raw `req` objects into a service.
- **Output DTOs:** Define explicit response DTO types (`OrderDto`). Map database records to Output DTOs before returning so internal/sensitive columns (`passwordHash`, `deletedAt`, internal tokens) never leak.

### Stage 3: Controller vs. Service Isolation & `async/await`
- **Controller / Route Handler:** Zero business logic and zero direct database calls. Only parses/validates input, calls the service with `await`, sets HTTP status codes, and wraps the return value in `ok()` or `paginated()`.
- **Service Layer:** Holds all domain rules, authorization checks, and database queries.
- **Async/Await Rule:** Always use `async/await` for database and I/O operations. Never use `.then()/.catch()` promise chains. Only use `await Promise.all([...])` when running independent queries concurrently (e.g., fetching paginated `items` and `total` count in parallel).

### Stage 4: Proper Error Handling & HTTP Semantics
- Services throw explicit domain/HTTP exceptions (`NotFoundException`, `ConflictException`, `UnprocessableEntityException`) — never return `null` to let the controller guess.
- Catch known database errors (e.g., Postgres unique violation `23505` -> `409 CONFLICT`) and map unhandled exceptions to `500 INTERNAL_ERROR` without exposing raw SQL or stack traces.
- **HTTP Status Codes:**
  - `200 OK`: Read or update (`GET`, `PATCH`, `PUT`).
  - `201 Created`: Created (`POST`), returns created resource in `data`.
  - `204 No Content`: Deleted (`DELETE`), empty body.
  - `400 Bad Request`: Validation/sanitization failure (`VALIDATION_FAILED`).
  - `401 Unauthorized` / `403 Forbidden`: Auth failure (`UNAUTHORIZED` / `FORBIDDEN`).
  - `404 Not Found`: Missing resource (`RESOURCE_NOT_FOUND`).
  - `409 Conflict`: Duplicate unique field or state conflict (`CONFLICT`).
  - `422 Unprocessable Entity`: Business rule violation (`UNPROCESSABLE_ENTITY`).
  - `500 Internal Server Error`: Unhandled fault (`INTERNAL_ERROR`).

---

## 2. Standard Response Envelopes

Both success and error envelopes **must** include a top-level human-readable `message` field so the frontend can always read `response.message`.

### Success (`200` / `201`)
```json
{
  "success": true,
  "message": "Orders fetched successfully",
  "data": {},
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 42,
    "totalPages": 3
  },
  "timestamp": "2026-10-19T18:00:00.000Z"
}
```
*(Omit `meta` on single-resource responses.)*

### Error (`4xx` / `5xx`)
```json
{
  "success": false,
  "message": "Invalid request payload",
  "error": {
    "code": "VALIDATION_FAILED",
    "details": [
      { "field": "email", "issue": "Invalid email format" }
    ]
  },
  "timestamp": "2026-10-19T18:00:00.000Z",
  "path": "/v1/users"
}
```

---

## 3. Framework Examples & References (Load On-Demand Only)

Read **only** the file matching the current project's stack:
- **NestJS (Zod Params/Query/Body + Controller + Service + Exception Filter):** [examples/nestjs.md](./examples/nestjs.md)
- **Next.js App Router (`route.ts` + Service + Zod + Error Handler):** [examples/nextjs.md](./examples/nextjs.md)
- **Standard Error Codes Catalog:** [references/error-codes.md](./references/error-codes.md)

## 4. Verification Checklist
- [ ] Route params (`uuid`/`cuid`/`string`/`number`), query params, and body are sanitized and validated with Zod (or project validator).
- [ ] Controller is thin; all business logic and `async/await` DB queries live in the Service.
- [ ] Controller and Service methods use strict input (`z.infer`) and output DTO types.
- [ ] Errors (`400`, `404`, `409`, `422`, `500`) return the standard `ApiErrorResponse` envelope without leaking internals.