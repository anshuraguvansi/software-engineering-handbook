# API Error Handling

Error handling is a contract, not an afterthought.

A strong error strategy helps clients recover safely, helps operators debug quickly, and prevents sensitive data leakage.

## 1. Core Principles

1. Use semantic HTTP status codes.
2. Return a consistent machine-readable error shape.
3. Separate client-facing messages from internal diagnostics.
4. Include trace correlation identifiers.
5. Keep retry behavior explicit and predictable.

## 2. Standard Error Format (Problem Details)

Use Problem Details JSON (`application/problem+json`), standardized in RFC 7807 and updated by RFC 9457.

Base fields:

- `type`: URI identifying error category
- `title`: short summary
- `status`: HTTP status code
- `detail`: human-readable explanation
- `instance`: specific request or resource reference

Recommended extension fields:

- `code`: stable machine token (for example, `validation_failed`)
- `trace_id`: request correlation id
- `invalid_params`: field-level validation failures
- `retryable`: boolean retry hint

Example:

```json
{
  "type": "https://developer.example.com/problems/validation-failed",
  "title": "Validation failed",
  "status": 422,
  "detail": "One or more input fields are invalid.",
  "instance": "/v1/users/register",
  "code": "validation_failed",
  "invalid_params": [
    { "name": "email", "reason": "must be a valid email address" },
    { "name": "password", "reason": "must include uppercase and number" }
  ],
  "trace_id": "01JAM2D9V9C8XQ3C8Q5M3N1G7T",
  "retryable": false
}
```

## 3. Status Code Mapping

Use this baseline mapping across services.

| Category | Status | Typical Code | Notes |
|---|---|---|---|
| Malformed request syntax | 400 Bad Request | `malformed_json` | Parsing or protocol-level issue |
| Authentication failure | 401 Unauthorized | `invalid_access_token` | Include `WWW-Authenticate` when relevant |
| Authorization failure | 403 Forbidden | `insufficient_permissions` | Authenticated but not allowed |
| Resource missing | 404 Not Found | `resource_not_found` | Unknown id or hidden resource policy |
| Method not allowed | 405 Method Not Allowed | `method_not_allowed` | Include `Allow` header |
| Conflict/state violation | 409 Conflict | `resource_conflict` | Duplicate key, state transitions, idempotency mismatch |
| Precondition failure | 412 Precondition Failed | `precondition_failed` | ETag/If-Match conditional write failure |
| Semantic validation error | 422 Unprocessable Entity | `validation_failed` | Well-formed request, invalid domain rules |
| Too many requests | 429 Too Many Requests | `rate_limit_exceeded` | Include `Retry-After` and rate-limit headers |
| Server fault | 500 Internal Server Error | `internal_server_error` | Unexpected fault |
| Upstream dependency issue | 502/503/504 | `upstream_failure` | Use precise gateway/dependency semantics |

## 4. 400 vs 422 Rule of Thumb

- `400`: request cannot be parsed/interpreted at protocol level.
- `422`: request is syntactically valid but fails business or domain validation.

Document this split and apply it consistently.

## 5. Error Code Design

Machine codes should be:

1. Stable across releases
2. Predictable naming style (`snake_case`)
3. Specific enough for UI behavior and automation

Suggested taxonomy:

- `auth.invalid_access_token`
- `user.email_already_exists`
- `payment.card_declined`
- `request.validation_failed`

Avoid changing codes after clients depend on them.

## 6. Global Error Interceptor Pattern

Do not build ad-hoc `try/catch` response formatting in each controller.

Use centralized middleware/interceptor that:

1. Maps known domain exceptions to status + error code
2. Converts validation library output to `invalid_params`
3. Redacts sensitive details
4. Injects `trace_id`
5. Emits structured logs and metrics

This guarantees uniform behavior across services.

## 7. Security and Privacy Rules

Never expose in API responses:

- SQL queries or stack traces
- Internal hostnames, file paths, memory addresses
- Secrets, tokens, private keys
- Full PII payload echoes

Public response should be minimal and safe. Internal logs can contain deeper diagnostics under access control.

## 8. Retry Semantics

Clients need clear retry signals.

Guidelines:

1. Mark retryability (`retryable` field and/or docs).
2. Include `Retry-After` for 429 and selected 503 responses.
3. Do not suggest retries for deterministic client errors (400/401/403/404/422).
4. For idempotent operations, retries are safer; for non-idempotent operations, pair with idempotency keys.

## 9. Headers That Improve Error Contracts

- `Content-Type: application/problem+json`
- `X-Request-Id` or trace header
- `Retry-After` for backoff guidance
- `WWW-Authenticate` for 401
- `Allow` for 405
- `RateLimit-Limit`, `RateLimit-Remaining`, `RateLimit-Reset` (or your chosen standard)

## 10. Localization Strategy

Keep `code` stable and language-neutral. Localize user-facing text at client side or via locale-aware layers.

- `detail` can be default English for debugging
- UI should primarily map from `code`

## 11. Observability and Operations

Track and alert on:

- error_rate by status family (4xx/5xx)
- top error codes
- retryable error volume
- 429 rate and burst behavior
- 5xx by dependency and endpoint

Log structure should include:

- trace_id
- request_id
- endpoint
- method
- status
- code
- tenant/user scope (redacted policy-compliant)

## 12. Example Responses

Validation error:

```http
HTTP/1.1 422 Unprocessable Entity
Content-Type: application/problem+json
X-Request-Id: 01JAM2D9V9C8XQ3C8Q5M3N1G7T

{
  "type": "https://developer.example.com/problems/validation-failed",
  "title": "Validation failed",
  "status": 422,
  "code": "request.validation_failed",
  "detail": "One or more input fields are invalid.",
  "invalid_params": [
    { "name": "email", "reason": "must be a valid email address" }
  ],
  "trace_id": "01JAM2D9V9C8XQ3C8Q5M3N1G7T",
  "retryable": false
}
```

Rate-limit error:

```http
HTTP/1.1 429 Too Many Requests
Content-Type: application/problem+json
Retry-After: 30
RateLimit-Limit: 120
RateLimit-Remaining: 0
RateLimit-Reset: 30

{
  "type": "https://developer.example.com/problems/rate-limit",
  "title": "Too Many Requests",
  "status": 429,
  "code": "request.rate_limit_exceeded",
  "detail": "Request quota exceeded. Retry after 30 seconds.",
  "trace_id": "01JAM2GH2KV3R4H6W9V6D8A2EZ",
  "retryable": true
}
```

## 13. Common Mistakes

1. Returning 200 with error payloads
2. Inconsistent schemas between services
3. Unstable error codes changing every release
4. Exposing stack traces publicly
5. Treating all 4xx as retryable
6. Missing correlation id in error responses
7. No distinction between 400 and 422

## 14. Production Checklist

1. Uniform Problem Details schema enforced
2. Global interceptor implemented in every service
3. Status-code mapping standards documented
4. Error codes reviewed for stability and taxonomy
5. Sensitive data redaction verified
6. Retry headers and semantics tested
7. Monitoring dashboards and alerts configured
8. Contract tests validate error payload shape

---

Great API error handling is predictable, secure, and observable. If clients and operators can both act quickly on failures, your platform becomes far easier to integrate and operate.