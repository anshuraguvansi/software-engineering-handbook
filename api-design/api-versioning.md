# API Versioning

API versioning is the discipline of evolving contracts without breaking existing consumers.

A good versioning strategy does three things well:

1. Preserves backward compatibility where possible
2. Makes breaking changes explicit and predictable
3. Provides a clear deprecation and migration path

## 1. What Counts as a Breaking Change

A breaking change is any change that can cause an existing client integration to fail without client-side updates.

### Usually Non-Breaking

- Adding a new endpoint
- Adding an optional request field
- Adding a new response field when clients ignore unknown fields
- Adding new enum values only when consumers are designed to handle unknown values safely

### Usually Breaking

- Removing or renaming request/response fields
- Changing field types (for example, integer to string)
- Making an optional input required
- Tightening validation in a way that rejects previously valid requests
- Changing authentication/authorization requirements
- Removing an endpoint or HTTP method
- Changing response shape or semantics in incompatible ways

### Context-Dependent (Treat Carefully)

- Changing success code from 200 to 204
- Reordering arrays when clients relied on order
- Changing default sort/pagination behavior
- Switching nullability semantics

These can be safe for robust clients but are often breaking in real integrations.

## 2. Compatibility Rules for Contract Evolution

Use these rules before deciding whether to bump versions.

### Request Compatibility

- Adding optional request fields is backward-compatible
- Removing fields is breaking
- Making validation stricter is often breaking

### Response Compatibility

- Adding fields is usually backward-compatible
- Removing fields is breaking
- Changing field meaning is breaking even if the type stays the same

### Behavior Compatibility

- Keep side effects, ordering, and pagination semantics stable
- If behavior changes materially, treat it as breaking

## 3. Versioning Strategies

There is no single universal strategy. Pick one default and use it consistently.

### A. URI Path Versioning

Example:

```http
GET /v1/users/123
```

Pros:

- Highly visible and easy to test
- Straightforward routing and observability
- Works cleanly with most caching layers

Cons:

- Puts representation version in URI
- Can duplicate route surface across versions

### B. Custom Header Versioning

Example:

```http
GET /users/123
X-API-Version: 2
```

Pros:

- Keeps URLs stable
- Allows flexible version selection

Cons:

- Harder to test from basic browser workflows
- Requires cache correctness with `Vary: X-API-Version`

### C. Media-Type Versioning (Accept Header)

Example:

```http
GET /users/123
Accept: application/vnd.example.user.v2+json
```

Pros:

- Clean URI model
- Aligns with representation negotiation

Cons:

- Higher implementation and consumer complexity
- Requires cache correctness with `Vary: Accept`

### D. Query Parameter Versioning

Example:

```http
GET /users/123?version=2
```

Pros:

- Easy to roll out quickly
- Simple for manual testing

Cons:

- Easier to misuse or omit
- Cache behavior depends on proxy and CDN configuration

## 4. Recommendation for Most Teams

For most production systems, default to URI path versioning as your primary strategy:

- `/v1/...`, `/v2/...`

Reason:

- Best operational clarity for clients, gateway routing, and observability
- Lowest accidental complexity for most engineering teams

If your platform has strong API gateway and governance tooling, header-based strategies can also work well.

## 5. Version Scope and Naming

Define what a version applies to:

1. Whole API surface (common default)
2. Domain or bounded context
3. Individual resource families

Avoid mixed policies unless governance is mature. Inconsistent scoping creates confusion.

Naming options:

- Integer major versions: `v1`, `v2`
- Date-based versions for public APIs: `2026-08-01`

Choose one convention and document it.

## 6. Deprecation and Sunset Lifecycle

Versioning without retirement planning creates permanent maintenance burden.

Use explicit lifecycle signals:

- `Deprecation` response header to indicate deprecated behavior
- `Sunset` response header (RFC 8594) to indicate end-of-support date
- `Link` header with migration docs (`rel="successor-version"` or docs URL)

Example:

```http
HTTP/1.1 200 OK
Content-Type: application/json
Deprecation: true
Sunset: Wed, 11 Nov 2026 23:59:59 GMT
Link: <https://developer.example.com/apis/users/v2-migration>; rel="successor-version"
```

Good practice:

1. Announce deprecation early
2. Provide migration guides and SDK updates
3. Monitor usage of deprecated versions
4. Enforce sunset only after published notice windows

## 7. Migration Playbook for Breaking Changes

When introducing v2:

1. Run v1 and v2 in parallel
2. Publish a change log with concrete before/after examples
3. Add SDK and contract-test support for v2
4. Offer dual-write or compatibility adapters when feasible
5. Track adoption metrics and top client blockers
6. Sunset v1 after objective usage thresholds are reached

## 8. Caching and Gateways

Versioning decisions must account for cache keys and intermediaries.

Checklist:

- Path versioning: route naturally segregated by URI
- Header/media-type versioning: configure `Vary` headers correctly
- Query versioning: verify cache-key policy includes version parameter
- Keep version dimensions visible in logs and metrics labels

## 9. Governance Rules You Should Enforce

1. Every breaking change requires a new major version
2. Every change must include compatibility classification
3. Every major version requires a migration guide
4. Every deprecated version requires a sunset date
5. Every release requires contract-test validation against supported versions

## 10. Practical Decision Matrix

| Strategy | Cache Friendliness | Developer Experience | Complexity | Typical Fit |
|---|---|---|---|---|
| URI Path | High | High | Low | Most teams |
| Custom Header | Medium (needs `Vary`) | Medium | Medium | Internal platforms |
| Accept Header | Medium (needs `Vary`) | Low-Medium | High | Mature API governance |
| Query Param | Medium (cache-config sensitive) | Medium-High | Low | Transitional/legacy setups |

## 11. Production Checklist

1. Breaking vs non-breaking policy is documented
2. Versioning strategy is consistent across the API portfolio
3. Deprecation and sunset headers are implemented
4. Migration docs exist for each breaking release
5. Usage telemetry by version is visible in dashboards
6. Contract tests run for all active versions
7. Cache and gateway policies are verified for selected strategy

---

A strong API versioning strategy is less about picking a fashionable mechanism and more about predictable evolution, explicit communication, and operational discipline.