---
applyTo: "**/*Controller*.java,**/*Resource*.java,**/*Endpoint*.java,**/*Controller.cs,**/*.controller.ts,**/*.routes.ts,openapi*.yaml,openapi*.yml,swagger*.yaml,swagger*.yml"
description: "REST API and contract standards — backward compatibility, versioning, DTOs, status codes, pagination, idempotency, and OpenAPI"
---

# API Standards

## Backward Compatibility

- Additive changes are safe: new optional fields, new endpoints, new optional query parameters.
- The following are **Blocker** breaking changes without versioning:
  - Removing or renaming a field in a response body.
  - Changing a field's type (e.g., `string` → `integer`).
  - Making a previously optional request field required.
  - Removing or renaming an endpoint.
  - Changing an endpoint's HTTP method.
  - Changing the meaning of a status code returned for an existing scenario.
- Deprecate before removing: mark deprecated fields/endpoints in OpenAPI with `deprecated: true`, communicate a migration timeline, then remove in a future version.

## Versioning

- Version APIs via URL path (`/api/v2/orders`) rather than headers, unless the project has an established convention.
- Introduce a new version when making breaking changes — do not break existing consumers silently.
- Keep old versions active long enough for consumers to migrate; document the sunset date.

## Request and Response DTOs

- Never expose persistence entities (JPA entities, EF Core models, Mongoose documents) as API types.
  - Entity-to-API coupling means internal schema changes silently break the API contract.
  - Lazy-loaded collections and audit fields must not appear in public responses.
- Use dedicated, named request DTOs and response DTOs.
- DTOs should be flat and serialisation-friendly — avoid deep nested structures where possible.

## HTTP Status Codes

Use semantically correct status codes consistently:

| Scenario | Code |
|---|---|
| Success (with response body) | 200 OK |
| Successful creation | 201 Created (+ `Location` header pointing to the new resource) |
| Success (no response body) | 204 No Content |
| Invalid request / validation failure | 400 Bad Request |
| Missing or invalid authentication | 401 Unauthorized |
| Authenticated but not authorised | 403 Forbidden |
| Resource not found | 404 Not Found |
| Conflict (duplicate, stale version) | 409 Conflict |
| Business rule violation (valid format, invalid state) | 422 Unprocessable Entity |
| Rate limit exceeded | 429 Too Many Requests |
| Unexpected server error | 500 Internal Server Error |
| Downstream dependency unavailable | 503 Service Unavailable |

## Error Response Format

- Use a consistent error format across all endpoints — RFC 7807 Problem Details is the recommended standard:

```json
{
  "type": "https://example.com/problems/order-not-found",
  "title": "Order not found",
  "status": 404,
  "detail": "No order with ID 12345 exists.",
  "instance": "/api/v1/orders/12345"
}
```

- Never include: stack traces, SQL error messages, DB constraint names, internal class names, or infrastructure details.
- Validation errors should list all failing fields, not just the first.

## Input Validation

- Validate all inputs at the API boundary before any business logic executes.
- Return `400` with field-level validation errors for malformed requests.
- Validate path variables, query parameters, and headers — not just the request body.

## Pagination

- All list endpoints that can return more than a fixed small number of items must be paginated.
- Prefer offset/page-based pagination (`?page=0&size=20`) for simple use cases.
- Prefer cursor/keyset pagination for large datasets or real-time feeds (consistent performance, no skipped rows).
- Include pagination metadata in the response: `totalElements`, `totalPages`, `page`, `size`, or a `nextCursor`.
- Never return unbounded lists — this is a **Major** finding.

## Idempotency

- POST/PATCH operations that may be retried by clients or infrastructure must either be naturally idempotent or support an idempotency key header (e.g., `Idempotency-Key`).
- PUT operations should be naturally idempotent (same input = same state, always).
- Document idempotency guarantees in the OpenAPI spec.

## OpenAPI / Swagger

- Update the OpenAPI/Swagger specification for every change to: endpoint paths, HTTP methods, request body shapes, response shapes, status codes, or authentication requirements.
- Keep schema `$ref` definitions DRY — reuse named schemas for shared request/response types.
- Mark deprecated fields and endpoints with `deprecated: true`.
- Include examples in the spec for request bodies and key response shapes.

## Information Leakage

- Do not return internal details in any response field or header:
  - No stack traces.
  - No SQL or DB error messages.
  - No internal service names, hostnames, or IP addresses.
  - No framework version headers (remove `X-Powered-By`, `Server` headers in production).
- Rate limiting and throttling headers (`X-RateLimit-Limit`, `Retry-After`) should be documented if the API applies them.
