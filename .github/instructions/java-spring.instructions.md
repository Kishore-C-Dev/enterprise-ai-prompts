---
applyTo: "**/*.java,pom.xml,build.gradle,build.gradle.kts,src/main/resources/**/*.yml,src/main/resources/**/*.yaml,src/main/resources/**/*.properties"
description: "Java and Spring Boot review standards — architecture, injection, transactions, validation, error handling, Optional usage, and dependency safety"
---

# Java and Spring Boot Instructions

## Language and Version

- Target Java 17 or 21 LTS. Use modern features where they improve clarity:
  - Records for immutable DTOs and value objects.
  - Sealed classes/interfaces for closed type hierarchies.
  - Text blocks for multi-line strings (SQL, JSON templates).
  - Pattern matching for `instanceof` — eliminates redundant casts.
- Do not use deprecated APIs; flag use of removed or deprecated classes from earlier Java versions.

## Architecture

- **Controllers** are thin: parse input, call one service method, return a response. No business logic.
- **Services** own use-case orchestration and business rules. No direct persistence calls from controllers.
- **Repositories/DAOs** are focused on data access. No business logic; no HTTP calls.
- Avoid circular Spring bean dependencies — they indicate a design problem.

## Dependency Injection

- Use **constructor injection only**. Field injection (`@Autowired` on fields) is a Blocker — it hides dependencies and breaks testability.
- Use `@RequiredArgsConstructor` (Lombok) or explicit constructors.
- Prefer `@ConfigurationProperties` over scattered `@Value` annotations for grouped config; bind to validated, immutable classes or records.

## Request Validation

- Validate all `@RequestBody`, `@RequestParam`, and `@PathVariable` inputs using Bean Validation (`@Valid`, `@NotNull`, `@Size`, `@Pattern`, etc.) at the controller boundary.
- Return `400 Bad Request` with a consistent error body for validation failures — never propagate `ConstraintViolationException` to the client as a raw 500.

## DTOs and Entities

- Do not expose JPA/Hibernate entities directly through public API responses. Use dedicated response DTOs.
  - Entity changes (new fields, lazy proxies, Hibernate metadata) must not silently alter the API contract.
  - Entities serialised directly can expose internal state or trigger `LazyInitializationException`.
- Map between entities and DTOs in the service layer; use MapStruct or explicit constructors — not `BeanUtils.copyProperties` on complex graphs.

## Transactions

- Place `@Transactional` on service methods, not controllers or repositories.
- Use `@Transactional(readOnly = true)` for read-only service methods.
- Do not make remote HTTP or messaging calls inside a `@Transactional` method unless the failure semantics are understood and documented.
- Handle `OptimisticLockingFailureException` and `DataIntegrityViolationException` at the service boundary; translate to domain exceptions with meaningful messages.

## Database Access

- Avoid DB calls inside loops — this is an N+1 query pattern and a **Major** finding.
- Use pagination for all unbounded queries (`Pageable`, `LIMIT`/`OFFSET`, or keyset pagination).
- Prefer `@Query` with named parameters or Spring Data method names over Criteria API for complex queries.
- Disable `spring.jpa.open-in-view=true` in all profiles — it causes lazy-load surprises and holds DB connections through the HTTP thread.

## Exception Handling

- Preserve exception root causes when wrapping: always chain the original exception (`new MyException("message", cause)`).
- Avoid `catch (Exception e) { log.error("error"); }` without rethrowing or handling — this is a silent swallow.
- Use `@ControllerAdvice` + `@ExceptionHandler` to map domain exceptions to HTTP responses consistently.
- Consider RFC 7807 Problem Details (`application/problem+json`) for JSON APIs — Spring 6 / Boot 3 supports this natively via `ProblemDetail`.
- Set `server.error.include-stacktrace=never` and `server.error.include-message=never` in production profiles.

## Optional Usage

- Use `Optional<T>` **only** as a method return type (typically repository methods).
- Never use `Optional` as a method parameter, constructor parameter, or field — it adds noise without benefit and serialises poorly.
- Avoid `Optional.get()` without `isPresent()` — use `orElse`, `orElseThrow`, `orElseGet`, or `map`/`flatMap`.

## HTTP Status Codes

| Scenario | Code |
|---|---|
| Successful GET / update | 200 OK |
| Successful creation | 201 Created + `Location` header |
| Successful delete (no body) | 204 No Content |
| Validation failure | 400 Bad Request |
| Unauthenticated | 401 Unauthorized |
| Authenticated but not authorised | 403 Forbidden |
| Resource not found | 404 Not Found |
| Conflict (duplicate, version mismatch) | 409 Conflict |
| Semantically invalid (passed validation, failed business rule) | 422 Unprocessable Entity |
| Unexpected server error | 500 Internal Server Error |

## Maven / Gradle Dependency Review

- Flag additions of transitive scope overrides, new top-level dependencies, or version downgrades.
- Verify no dependency has a known CVE at the version added (`mvn dependency-check:check` / `./gradlew dependencyCheckAnalyze`).
- Avoid `implementation 'group:artifact:+'` or SNAPSHOT versions in non-development builds.
- Keep `spring-boot-dependencies` BOM as the primary version manager — avoid overriding Spring-managed versions unless required.

## Spring Configuration Safety

- Never commit secrets, tokens, passwords, queue names, or environment-specific URLs to `application.yml` or `application.properties`.
- Externalise secrets to Vault, AWS Secrets Manager, Azure Key Vault, or environment variables at runtime.
- Use named Spring profiles (`dev`, `staging`, `prod`) to separate environment config.
- Set `management.endpoints.web.exposure.include` explicitly — never use `*` in production.
- Restrict `/actuator/env`, `/actuator/heapdump`, and `/actuator/threaddump` to internal/admin roles.
