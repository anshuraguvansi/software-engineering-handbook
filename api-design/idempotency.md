# API Idempotency

Idempotency ensures that retrying the same operation does not create unintended side effects.

In distributed systems, retries are normal due to timeouts, connection resets, load balancers, and client restarts. Without idempotency controls, retries can produce duplicate charges, duplicate orders, and inconsistent state.

## 1. Protocol Idempotency vs Application Idempotency

HTTP defines method semantics, but business safety still depends on implementation.

| HTTP Method | Protocol-Level Idempotent | Notes |
|---|---|---|
| GET | Yes | Safe read; no mutation expected |
| HEAD | Yes | Metadata read |
| OPTIONS | Yes | Capability discovery |
| PUT | Yes | Full replacement semantics |
| DELETE | Yes | Repeating delete should not create new side effects |
| POST | No | Usually requires application-level idempotency |
| PATCH | Depends | Idempotent for absolute updates, non-idempotent for relative updates |

Important distinction:

- Protocol-level idempotency means repeating the same request has the same intended effect.
- Application-level idempotency means your business operation is duplicate-safe under retries.

## 2. PATCH Edge Case

Idempotent PATCH example:

```http
PATCH /v1/accounts/1
Content-Type: application/merge-patch+json

{ "status": "suspended" }
```

Repeating this keeps the account in the same final state.

Non-idempotent PATCH example:

```http
PATCH /v1/accounts/1
Content-Type: application/json

{ "increment_by": 10 }
```

Repeating this changes state each time.

## 3. Where Idempotency Is Required Most

Prioritize idempotency for operations that are:

1. Financial or inventory-affecting
2. Triggered from unreliable networks (mobile/web)
3. Executed through async queues or webhook retries
4. Exposed as public APIs where clients auto-retry

Typical examples:

- Payments
- Order creation
- Subscription changes
- External side-effect commands

## 4. Idempotency Key Pattern (Intercept and Replay)

For non-idempotent operations (usually POST), use an idempotency key.

Client sends:

```http
POST /v1/payments
Idempotency-Key: 2f61eb6c-4d4f-4850-89f3-2f0f38f4b6f8
Content-Type: application/json
```

Server behavior:

1. Compute request fingerprint from method + route + normalized body + tenant/user scope.
2. Attempt atomic key creation in shared storage (for example Redis SET NX with TTL).
3. If key is new, process request and store final response metadata.
4. If key already completed with same fingerprint, replay stored response.
5. If key exists but payload fingerprint differs, reject as conflict/misuse.

## 5. Storage Record Model

Persist enough data to replay safely.

Suggested record fields:

- idempotency_key
- scope (tenant/user/client)
- method
- path template
- request_fingerprint
- status (IN_PROGRESS, COMPLETED, FAILED_RETRYABLE)
- response_status_code
- response_headers (subset)
- response_body
- created_at, expires_at

## 6. Response Strategy for Duplicates

When duplicate key arrives:

1. If original request is COMPLETED and fingerprint matches, return same status/body.
2. If original request is IN_PROGRESS, return a deterministic response such as:
   - 409 Conflict, or
   - 202 Accepted with Retry-After
3. If fingerprint mismatches, return 409 Conflict (or 422 by policy) and do not process.

The key principle is deterministic behavior, not a specific mandatory status code.

## 7. Security and Abuse Controls

Do not trust key alone. Bind key to request identity and payload.

Fingerprint example:

$$
\text{fingerprint} = \text{SHA256}(\text{method} || \text{pathTemplate} || \text{canonicalBody} || \text{tenantId} || \text{userId})
$$

Guidelines:

1. Use canonical JSON when hashing to avoid field-order false mismatches.
2. Enforce key entropy (UUIDv4 or equivalent).
3. Reject overly long or malformed keys.
4. Apply TTL and cleanup policies.

## 8. TTL and Retention Windows

Choose TTL based on business retry windows and risk profile.

Typical ranges:

- Low-risk commands: minutes to hours
- Financial operations: 24 to 72 hours or longer per policy

Trade-off:

- Short TTL risks duplicate side effects after expiry.
- Long TTL increases storage costs and key cardinality.

## 9. Database Backstop (Unique Constraint Pattern)

Idempotency cache is not enough. The database should still enforce uniqueness for critical operations.

```sql
CREATE TABLE payment_transactions (
    id UUID PRIMARY KEY,
    user_id BIGINT NOT NULL,
    amount DECIMAL(12,2) NOT NULL,
    idempotency_fingerprint CHAR(64) NOT NULL UNIQUE,
    status VARCHAR(20) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);
```

```sql
INSERT INTO payment_transactions (id, user_id, amount, idempotency_fingerprint, status)
VALUES ('9b1deb4d-3b7d-4fdd-a6f0-4e8f0de8d72a', 123, 100.00, 'computed_sha256_here', 'SUCCESS');
```

This is a uniqueness guard, not full Optimistic Concurrency Control (OCC). OCC usually relies on version checks during updates.

## 10. Concurrency and Race Conditions

Handle double-submit races with atomic primitives:

1. Use atomic lock/create semantics in shared store.
2. Mark request IN_PROGRESS before side effects begin.
3. Ensure lock timeout is safe for worst-case processing.
4. Always finalize state (COMPLETED or FAILED_RETRYABLE) even on exceptions.

If process crashes mid-flight, recovery logic should prevent stuck IN_PROGRESS records.

## 11. Multi-Region and Distributed Considerations

For global systems:

1. Prefer region-local key processing where possible.
2. Ensure consistent routing for same key (sticky or hash-based).
3. If active-active writes exist, design for replication lag and conflict handling.
4. Keep idempotency scope explicit (global vs regional) per business operation.

## 12. Observability and Operations

Track these metrics:

- duplicate_request_rate
- in_progress_conflict_rate
- fingerprint_mismatch_rate
- replay_response_rate
- key_store_latency
- key_store_error_rate

Log fields:

- request_id
- idempotency_key
- fingerprint_hash_prefix
- idempotency_status
- replayed (true/false)

## 13. Common Mistakes

1. Treating POST retries as safe without idempotency keys
2. Hashing raw JSON without canonicalization
3. Keying only by idempotency key and ignoring user/tenant scope
4. Returning different responses for the same completed key
5. No DB uniqueness backstop for critical money flows
6. TTL too short for real retry behavior

## 14. Production Checklist

1. Idempotency key required on selected POST endpoints
2. Atomic key-claim mechanism implemented
3. Fingerprint includes method, route, body, and principal scope
4. Deterministic duplicate response policy documented
5. IN_PROGRESS timeout and recovery flow defined
6. Database uniqueness guard in place for critical writes
7. Metrics and alerts configured
8. Runbook documents retry semantics for clients

---

Idempotency is not optional hardening. It is a core correctness control for any API that can be retried under failure conditions.