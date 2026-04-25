---
applyTo: "**/*.{java,yml,yaml,properties,xml}"
description: "Use when writing or reviewing Java Spring Boot code — covers architecture, configuration, transactions, resilience, and API design"
---

# Spring Boot Instructions

## Application Architecture

- Keep controllers thin: parse input, call one service method, return response. No business logic in controllers.
- Keep services focused on use cases and orchestration. No persistence calls directly in controllers.
- Keep repositories/DAO classes focused on data access. No business logic in repositories.
- Avoid circular dependencies between Spring beans.
- Use constructor injection; avoid `@Autowired` field injection. Mark constructors with `@RequiredArgsConstructor` (Lombok) or write them explicitly.
- Prefer `@ConfigurationProperties` over `@Value` for grouped configuration; bind to immutable records or classes with validation.

## Configuration

- Never hardcode URLs, credentials, passwords, queue names, topic names, timeouts, or environment-specific values in code or `application.yml` committed to source.
- Externalise all secrets to Vault, AWS Secrets Manager, Azure Key Vault, or equivalent. Use Spring Cloud Config or environment variables at runtime.
- Use Spring profiles (`application-{profile}.yml`) to separate environment-specific configuration; name profiles clearly (`dev`, `staging`, `prod`).
- Set `spring.application.name` explicitly — it is used in distributed tracing, service discovery, and log correlation.

## Transactions

- Prefer explicit `@Transactional` boundaries on service methods over controller-level or repository-level transactions.
- Use `@Transactional(readOnly = true)` for read-only service methods.
- Avoid making remote HTTP or messaging calls inside `@Transactional` methods unless the remote call is part of the intended unit of work and the failure semantics are understood.
- Do not span a transaction across user-facing waits (e.g., opening a transaction, prompting a user, then committing).
- Handle `OptimisticLockingFailureException` and `DataIntegrityViolationException` at the service boundary; translate to appropriate domain exceptions.

## API Design

- Validate all request payloads using Bean Validation (`@Valid`, `@NotNull`, `@Size`, `@Pattern`, etc.) at the controller level.
- Return consistent error responses using a `@ControllerAdvice` + `@ExceptionHandler`; map domain exceptions to HTTP status codes.
- Do not expose stack traces, SQL details, internal class names, or infrastructure error messages to API consumers. Set `server.error.include-stacktrace=never`.
- Use appropriate HTTP status codes: `200 OK`, `201 Created`, `204 No Content`, `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `409 Conflict`, `422 Unprocessable Entity`, `500 Internal Server Error`.
- Maintain backward compatibility for public APIs. Additive changes only unless a breaking change is explicitly versioned.
- Keep DTOs separate from JPA entities unless the project already intentionally combines them and the tradeoffs are accepted.

## Resilience

- Configure connection, read, write, and overall timeouts for all HTTP clients (`RestTemplate`, `WebClient`, `FeignClient`), database connections, and messaging clients.
- Use `Resilience4j` for circuit breakers, bulkheads, and rate limiters; configure fallback behaviour for critical dependencies.
- Apply retries only to transient, idempotent operations; use exponential backoff with jitter; set a maximum retry count.
- Ensure message consumers and scheduled jobs are safe to retry and idempotent where messages may be delivered more than once.
- Avoid blocking virtual threads or reactive pipelines on slow synchronous dependencies.

## Actuator and Operations

- Secure all Actuator endpoints with Spring Security; restrict `/actuator/env`, `/actuator/heapdump`, and `/actuator/threaddump` to internal/admin roles.
- Expose `/actuator/health` and `/actuator/info` for liveness/readiness probes; use health groups for Kubernetes probes.
- Include custom `HealthIndicator` implementations for critical downstream dependencies.
- Set `management.endpoints.web.exposure.include` explicitly — do not use `*` in production.
