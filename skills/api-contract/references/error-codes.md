# Standard Error Codes Catalog

| HTTP Status | `error.code` | When to use |
|---|---|---|
| `400` | `VALIDATION_FAILED` | Body, query, or path parameter failed schema validation |
| `400` | `BAD_REQUEST` | Malformed JSON or invalid request syntax |
| `401` | `UNAUTHORIZED` | Missing, expired, or invalid auth token |
| `403` | `FORBIDDEN` | Authenticated user lacks required role/permission |
| `404` | `RESOURCE_NOT_FOUND` | Requested entity ID/slug does not exist |
| `409` | `CONFLICT` | Unique constraint violation (e.g., duplicate email/slug) or invalid state transition |
| `422` | `UNPROCESSABLE_ENTITY` | Business rule violation (e.g., insufficient balance, out of stock) |
| `429` | `RATE_LIMITED` | Too many requests from client |
| `500` | `INTERNAL_ERROR` | Unexpected server fault (log full error internally, return generic message) |
