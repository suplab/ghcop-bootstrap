# eeik-managed pack=java
# REST API Standard

**Pack:** java-pack | **Version:** 1.0

---

## URI Design

- Use nouns, not verbs: `/orders` not `/createOrder`
- Kebab-case for multi-word resources: `/line-items` not `/lineItems`
- Nest to show ownership: `/orders/{orderId}/line-items`
- Max 2 levels of nesting — deeper = design smell
- API versioning in path: `/v1/orders`, `/v2/orders`

## HTTP Method Semantics

| Method | Use | Idempotent |
|--------|-----|-----------|
| GET | Read (no side effects) | Yes |
| POST | Create | No |
| PUT | Replace entire resource | Yes |
| PATCH | Partial update | No |
| DELETE | Remove | Yes |

## Response Shapes

### Success
```json
{
  "orderId": "uuid",
  "status": "CONFIRMED",
  "createdAt": "2024-01-15T10:30:00Z"
}
```

### Error (RFC 7807 ProblemDetail)
```json
{
  "type": "https://api.example.com/errors/order-not-found",
  "title": "Order Not Found",
  "status": 404,
  "detail": "No order found with ID: 550e8400-e29b-41d4-a716-446655440000",
  "instance": "/v1/orders/550e8400-e29b-41d4-a716-446655440000",
  "orderId": "550e8400-e29b-41d4-a716-446655440000"
}
```

## Authentication & Authorisation

All APIs are secured. Choose the pattern that matches your integration type:

| Integration Type | Auth Mechanism | Token Location |
|---|---|---|
| User-facing SPA / mobile | OAuth2 Authorization Code + PKCE | `Authorization: Bearer <jwt>` |
| Service-to-service (same org) | OAuth2 Client Credentials | `Authorization: Bearer <jwt>` |
| Service-to-service (AWS) | IAM SigV4 via task role | AWS SDK signs request |
| External partner API | API Key (short-lived, rotated monthly) | `X-Api-Key: <key>` header |

JWT validation rules:
- Verify signature using the JWKS endpoint of the issuer — never trust unverified JWTs
- Validate `exp`, `iss`, `aud` claims on every request
- Extract user identity from `sub` claim; extract roles/permissions from custom claims
- Never log the raw JWT token

---

## Versioning Strategy

- Version via URI path: `/v1/orders`, `/v2/orders`
- Never version via `Accept` header or query parameter — URI versioning is explicit and cacheable
- A new version is required for any breaking change:
  - Removing or renaming a field
  - Changing a field's type or semantics
  - Removing an endpoint
  - Changing HTTP status codes for existing scenarios
- Maintain at least the previous version for 90 days after releasing a new version
- Deprecate old versions with `Deprecation` and `Sunset` response headers:

```
Deprecation: Sun, 01 Jun 2025 00:00:00 GMT
Sunset: Sun, 01 Sep 2025 00:00:00 GMT
Link: <https://api.example.com/v2/orders>; rel="successor-version"
```

---

## Standard Error Code Registry

All domain errors must use a registered error code. Add new codes to `docs/api/error-codes.md`.

| HTTP Status | Error Code | When to Use |
|---|---|---|
| 400 | `VALIDATION_ERROR` | Request fails bean validation |
| 400 | `INVALID_FORMAT` | Field value format is incorrect (e.g. bad UUID) |
| 400 | `MISSING_REQUIRED_FIELD` | Required field absent in request |
| 401 | `AUTHENTICATION_REQUIRED` | No valid token provided |
| 401 | `TOKEN_EXPIRED` | JWT `exp` claim in the past |
| 403 | `INSUFFICIENT_PERMISSIONS` | Authenticated but not authorised |
| 404 | `RESOURCE_NOT_FOUND` | Entity does not exist |
| 404 | `ENDPOINT_NOT_FOUND` | URL path not mapped |
| 409 | `DUPLICATE_RESOURCE` | Create would violate unique constraint |
| 409 | `STATE_CONFLICT` | Operation invalid for current entity state |
| 422 | `BUSINESS_RULE_VIOLATION` | Fails a domain business rule |
| 429 | `RATE_LIMIT_EXCEEDED` | Too many requests; include `Retry-After` header |
| 500 | `INTERNAL_ERROR` | Unexpected server error; do not expose details |
| 503 | `SERVICE_UNAVAILABLE` | Dependency unavailable; circuit breaker open |

Error response always uses RFC 7807 `ProblemDetail`:

```json
{
  "type": "https://api.example.com/errors/validation-error",
  "title": "Validation Error",
  "status": 400,
  "detail": "Field 'customerId' must be a valid UUID",
  "instance": "/v1/orders",
  "errorCode": "VALIDATION_ERROR",
  "fields": [
    { "field": "customerId", "message": "must be a valid UUID", "rejectedValue": "not-a-uuid" }
  ]
}
```

---

## OpenAPI Documentation

Every endpoint requires:
- `@Operation(summary = "...", description = "...")`
- `@ApiResponse` for each possible status code (including 4xx and 5xx)
- `@Schema` on all request/response DTOs
- `@Parameter` on all path/query parameters
- `@SecurityRequirement` referencing the security scheme

Generate and publish the spec at `/v3/api-docs` (JSON) and `/swagger-ui.html` (UI). The spec is the contract — do not modify it manually.

---

## Pagination

```json
GET /v1/orders?page=0&size=20&sort=createdAt,desc

{
  "content": [...],
  "page": { "number": 0, "size": 20, "totalElements": 143, "totalPages": 8 }
}
```

Use Spring Data `Pageable` — never manual offset/limit SQL.

Maximum page size: 100 items. Reject requests with `size > 100` with `400 VALIDATION_ERROR`.

---

## Rate Limiting

All public and partner-facing APIs must implement rate limiting at the API Gateway layer.

| API Tier | Rate Limit | Burst Limit |
|---|---|---|
| Public (unauthenticated) | 10 req/s per IP | 50 |
| Authenticated user | 100 req/s per user | 200 |
| Service-to-service | 500 req/s per client | 1000 |
| Partner API | Per SLA in partner agreement | Per SLA |

Return `429 Too Many Requests` with `Retry-After: <seconds>` header when limit exceeded.
