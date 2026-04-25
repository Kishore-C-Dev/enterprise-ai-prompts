---
applyTo: "**/*.cs,**/*.csproj,**/*.sln,appsettings*.json"
description: ".NET and ASP.NET Core review standards — DI, async/await, EF Core, validation, error handling, appsettings safety, and testing"
---

# .NET and ASP.NET Core Instructions

## Language and Project Setup

- Enable nullable reference types: `<Nullable>enable</Nullable>` in all `.csproj` files — reduces null-dereference risk at compile time.
- Target .NET 8 or .NET 9 LTS. Flag use of deprecated or removed APIs.
- Use C# 10+ features where they improve clarity:
  - Records for immutable DTOs and value objects.
  - Pattern matching and switch expressions over long if/else chains.
  - `required` properties on DTOs over constructors with many parameters.
  - File-scoped namespaces to reduce indentation.

## Architecture

- **Controllers / minimal API handlers** are thin: validate input, call one service method, return a result. No business logic.
- **Services** own use-case orchestration and business rules. No direct `DbContext` calls from controllers.
- **Repositories / data access** are focused on persistence. No business logic.
- Use `[ApiController]` on controllers — it enables automatic model validation and standardised 400 responses.

## Dependency Injection

- Use **constructor injection**. Never use the service locator pattern (`IServiceProvider.GetService<T>()` or `HttpContext.RequestServices`) in business logic — it hides dependencies and makes testing harder.
- Register services with the correct lifetime (`Singleton`, `Scoped`, `Transient`). Injecting a `Scoped` service into a `Singleton` is a runtime error — flag it.
- Avoid `static` service helpers or `ServiceLocator` wrappers.

## Async / Await

- Never block async code synchronously: `.Result`, `.Wait()`, and `.GetAwaiter().GetResult()` cause deadlocks in ASP.NET Core — these are **Blocker** findings.
- Always use `async`/`await` all the way through the call chain; never mix async and sync at the same level.
- Propagate `CancellationToken` through all async method signatures and pass it to EF Core, `HttpClient`, and other async APIs.
- Prefer `Task.WhenAll` for parallel independent operations over sequential `await` chains.

## Request Validation

- Use `[ApiController]` automatic validation for basic rules; use FluentValidation for complex domain rules.
- Always check `ModelState.IsValid` explicitly if not using `[ApiController]`.
- Return `400 Bad Request` with `ValidationProblemDetails` for validation failures.
- Do not trust client-supplied IDs or correlation keys without validating they belong to the authenticated user's scope.

## Response Types and HTTP Status Codes

- Use `ProblemDetails` or `ValidationProblemDetails` for all error responses — consistent with RFC 7807.
- Minimal API: use `Results.Problem`, `Results.ValidationProblem`, `Results.Ok`, `Results.Created`, `Results.NoContent`.
- Controller: return `IActionResult` or typed `ActionResult<T>` — never return raw objects from error paths.

| Scenario | Code |
|---|---|
| Successful GET / update | 200 OK |
| Successful creation | 201 Created |
| No response body | 204 No Content |
| Validation failure | 400 Bad Request |
| Unauthenticated | 401 Unauthorized |
| Authenticated but not authorised | 403 Forbidden |
| Resource not found | 404 Not Found |
| Conflict | 409 Conflict |
| Unexpected error | 500 Internal Server Error |

## DTOs and Entities

- Do not expose EF Core entities as API response or request types — use dedicated DTO/view model classes.
  - Entity changes (new columns, navigation properties, audit fields) must not silently alter the API contract.
  - Circular navigation properties cause infinite serialisation loops.
- Map between entities and DTOs in the service layer using AutoMapper projections or explicit mapping.

## EF Core

- Use `AsNoTracking()` for all read-only queries — avoids unnecessary change-tracking overhead.
- Avoid unbounded `Include()` chains that load entire object graphs; use `Select` projections instead.
- Avoid N+1 queries — use `Include` or `Join` for related data needed in a single pass. This is a **Major** finding.
- Pass `CancellationToken` to all EF Core async methods (`ToListAsync`, `FirstOrDefaultAsync`, `SaveChangesAsync`).
- Use `IDbContextFactory<T>` in background services and singletons — `DbContext` is `Scoped` and must not be shared across threads.
- Use `ExecuteSqlRawAsync` only with parameterised placeholders — never with string interpolation from user input.

## Transactions

- Wrap multi-step writes in an explicit transaction via `IDbContextFactory<T>` or `IDbContextTransaction`.
- Do not rely on implicit SaveChanges transactions spanning unrelated operations.
- Handle `DbUpdateConcurrencyException` at the service boundary and translate to a meaningful response (409 or retry logic).

## appsettings Safety

- Never store secrets, connection string passwords, API keys, or tokens in `appsettings.json` or `appsettings.*.json` committed to source.
- Use `dotnet user-secrets` for local development secrets.
- Use Azure Key Vault, AWS Secrets Manager, or environment variables in all non-local environments.
- Review `appsettings.json` for: `"DetailedErrors": true`, `"DeveloperExceptionPage"` enabled outside Development, hardcoded passwords, wildcard CORS origins.

## Logging

- Use `ILogger<T>` with structured message templates: `_logger.LogInformation("Order {OrderId} processed in {ElapsedMs}ms", orderId, elapsed)`.
- Never use string interpolation in log messages — it bypasses structured logging and prevents field extraction.
- Never log: secrets, tokens, authorization headers, PII, SSNs, card numbers, full request/response payloads.
- Configure Serilog or NLog with JSON output for production; include `TraceId`, `SpanId`, and `CorrelationId` enrichers.

## Exception Handling

- Use global exception handling middleware: `UseExceptionHandler` (production) or `UseDeveloperExceptionPage` (development only).
- Implement `IExceptionHandler` (ASP.NET Core 8+) for structured ProblemDetails responses on unhandled exceptions.
- Never return stack traces, EF Core exception details, or internal class names to API consumers.

## Unit and Integration Testing

- Use **xUnit** for all tests (`[Fact]`, `[Theory]` + `[InlineData]` / `[MemberData]`).
- Use **NSubstitute** or **Moq** for mocking in unit tests; NSubstitute is preferred for its cleaner syntax.
- Use **FluentAssertions** for readable, specific assertions.
- Use `WebApplicationFactory<TEntryPoint>` for integration tests of ASP.NET Core endpoints; inject test doubles via `ConfigureTestServices`.
- Use **Testcontainers** (.NET) for realistic DB and infrastructure tests; use `Respawn` to reset DB state between tests.
- Name projects `<ProjectName>.Tests` (unit) and `<ProjectName>.IntegrationTests`; name classes `<ClassName>Tests`.
