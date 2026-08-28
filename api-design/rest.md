# REST Deep Dive

REST (Representational State Transfer) is an architectural style for distributed systems. In web APIs, it is typically implemented over HTTP with resource-oriented URIs, standard methods, and self-descriptive messages.

This guide covers REST from first principles to production-level design decisions.

## 1. What REST Is (and Is Not)

REST is not just "JSON over HTTP". A system is closer to REST when it follows these constraints:

1. Client-server separation
2. Stateless communication
3. Cacheable responses
4. Uniform interface
5. Layered system
6. Code on demand (optional)

In practice, most production APIs are "REST-like" rather than strictly RESTful, especially around HATEOAS adoption.

## 2. Core Constraints

### Statelessness

Each request must contain enough context for the server to process it. The server should not rely on in-memory client session state between requests.

### Uniform Interface

A stable contract based on:

- Resource identification via URI
- Manipulation via representations (JSON, XML, etc.)
- Self-descriptive messages (headers, media types, status codes)
- Hypermedia controls (less common in many modern APIs)

### Cacheability

Responses should explicitly define cache behavior using headers such as:

- `Cache-Control`
- `ETag`
- `Last-Modified`

### Layered System

Clients should not need to know whether they are talking to origin servers, API gateways, or edge caches.

## 3. Resource Modeling and URI Design

Model nouns (resources), not verbs (actions).

Good:

- `/v1/users`
- `/v1/users/{userId}`
- `/v1/companies/{companyId}/employees/{employeeId}`

Avoid:

- `/v1/createUser`
- `/v1/getUserById`

Rules of thumb:

1. Use plural nouns consistently.
2. Keep hierarchy meaningful. Nest only when child lifecycle depends on parent.
3. Keep URIs stable; evolve representations and behavior via versioning.

## 4. HTTP Methods: Semantics, Safety, Idempotency

Safety means the method does not change server state.
Idempotency means repeating the same request yields the same intended effect as a single request.

| Method | Safe | Idempotent | Typical Use |
|---|---|---|---|
| GET | Yes | Yes | Read resource(s) |
| HEAD | Yes | Yes | Read metadata only |
| OPTIONS | Yes | Yes | Discover allowed methods/capabilities |
| POST | No | No | Create subordinate resource, trigger processing |
| PUT | No | Yes | Replace resource (or create at known URI) |
| PATCH | No | Depends | Partial update |
| DELETE | No | Yes | Remove resource |

Notes:

- `POST` is not always a DB INSERT; it can trigger workflows/jobs.
- `PUT` is idempotent by semantics, but only if implementation respects replacement behavior.
- `PATCH` can be idempotent when applying absolute updates (for example, `status = "active"`) and non-idempotent for relative operations (for example, `balance += 10`).

## 5. PUT vs PATCH

### PUT (full replacement semantics)

- Client sends the full desired state of the resource representation.
- Omitted fields may be reset depending on server contract.

### PATCH (partial modification semantics)

- Client sends only fields/operations to change.
- Supports patch document formats such as:
  - JSON Merge Patch: `application/merge-patch+json`
  - JSON Patch: `application/json-patch+json`

Pick one patch format and document it clearly.

## 6. Path, Query, and Body Parameters

### Path Parameters

Use for identity and required hierarchy.

Example:

- `GET /v1/users/{userId}`

### Query Parameters

Use for collection shaping (filter, sort, paginate, search).

Examples:

- `GET /v1/users?status=active`
- `GET /v1/users?sort=-createdAt`
- `GET /v1/users?page=2&pageSize=25`
- `GET /v1/users?q=johndoe`

### Request Body

Use for structured input to `POST`, `PUT`, and `PATCH`.

While HTTP allows bodies on other methods in theory, many proxies and intermediaries behave inconsistently. Avoid bodies in `GET` and usually in `DELETE` unless you fully control all intermediaries.

## 7. Content Negotiation

Use headers to separate resource from representation:

- `Accept`: what response media type the client can consume
- `Content-Type`: media type of the request body

Common media types:

- `application/json`
- `application/xml`
- `application/problem+json` (errors)

Response behavior:

1. If none of the requested `Accept` types are supported, return `406 Not Acceptable`.
2. If request body media type is unsupported, return `415 Unsupported Media Type`.

## 8. Status Codes That Matter

- `200 OK`: successful read/update with body
- `201 Created`: successful create, include `Location` header
- `202 Accepted`: async processing started
- `204 No Content`: successful operation with no body
- `304 Not Modified`: cache validation hit
- `400 Bad Request`: malformed request
- `401 Unauthorized`: missing/invalid authentication
- `403 Forbidden`: authenticated but not allowed
- `404 Not Found`: resource absent
- `409 Conflict`: state/version conflict
- `412 Precondition Failed`: failed conditional request (`If-Match`, etc.)
- `422 Unprocessable Entity`: semantic validation failure
- `429 Too Many Requests`: rate limit exceeded
- `500 Internal Server Error`: unexpected server fault
- `503 Service Unavailable`: temporary outage/overload

## 9. Caching and Conditional Requests

For read-heavy APIs, caching is a major performance lever.

Use:

- `Cache-Control` for cache policy (`max-age`, `no-store`, etc.)
- `ETag` with `If-None-Match` for conditional GET
- `Last-Modified` with `If-Modified-Since` as a simpler validator

Concurrency-safe updates:

- Return `ETag` on reads.
- Require `If-Match` on update/delete to prevent lost updates.
- Return `412 Precondition Failed` when tags do not match.

## 10. Idempotency in Real Systems

Retries happen due to timeouts, network glitches, and client restarts.

For non-idempotent creates/commands (`POST`), use idempotency keys:

- Client sends `Idempotency-Key` header.
- Server stores request fingerprint + result for a key window.
- Repeated key returns original response instead of duplicating side effects.

## 11. Error Contract

Use a consistent machine-readable error shape, preferably RFC 7807 (`application/problem+json`).

Example:

```json
{
  "type": "https://api.example.com/problems/validation-error",
  "title": "Validation failed",
  "status": 422,
  "detail": "email must be a valid address",
  "instance": "/v1/users",
  "errors": {
    "email": ["invalid format"]
  }
}
```

Benefits:

- Predictable client behavior
- Better observability and debugging
- Easier API governance

## 12. Pagination, Filtering, and Sorting Patterns

Recommended baseline:

- Pagination: `page` + `pageSize` or cursor-based pagination
- Filtering: exact-match fields (`status=active`)
- Sorting: allow-list sort fields (`sort=-createdAt,name`)

For large/changing datasets, prefer cursor pagination over offset pagination for consistency and performance.

## 13. Versioning Strategy

Common approaches:

1. URI versioning: `/v1/...` (simple and explicit)
2. Header/media-type versioning (clean URIs, more complex tooling)

Guidelines:

- Treat breaking changes as a new version.
- Keep old versions supported with a deprecation timeline.
- Communicate sunset dates and migration paths.

## 14. Security Baseline

1. Use HTTPS everywhere.
2. Use OAuth2/OIDC or strong token-based auth.
3. Enforce authorization per resource/action.
4. Validate and sanitize all inputs.
5. Never expose internal DB fields directly; map through DTOs.
6. Apply rate limits and abuse controls.
7. Avoid leaking internals in errors and logs.

## 15. Production Checklist

Use this quick review before launch:

1. Resource URIs are noun-based, plural, and consistent.
2. Method semantics match intent (`GET` read-only, idempotency respected).
3. Status codes are specific and accurate.
4. Error format is standardized (`application/problem+json`).
5. Caching strategy is explicit (`Cache-Control`, `ETag`).
6. Concurrency controls are in place (`If-Match` where needed).
7. Retry safety exists for `POST` (idempotency keys).
8. Pagination/filter/sort are documented and bounded.
9. AuthN/AuthZ and rate limiting are enforced.
10. Observability includes request IDs, latency, error rate, saturation.
11. Timeouts, circuit breakers, and backpressure are configured.

## 16. Practical Reference Endpoints

```http
GET    /v1/users
POST   /v1/users
GET    /v1/users/{userId}
PUT    /v1/users/{userId}
PATCH  /v1/users/{userId}
DELETE /v1/users/{userId}
```

Create example:

```http
POST /v1/users
Content-Type: application/json
Idempotency-Key: 84d4e2df-4d95-4f5e-a6e9-2462a73b6f2a

{
  "name": "John Doe",
  "email": "john@example.com"
}
```

```http
HTTP/1.1 201 Created
Location: /v1/users/123
Content-Type: application/json

{
  "id": "123",
  "name": "John Doe",
  "email": "john@example.com"
}
```

---

If you follow the semantics above, your API will be easier to scale, cache, secure, and evolve without surprising clients.