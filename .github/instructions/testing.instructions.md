---
applyTo: "**/*.{java,ts,js,spec.ts,spec.js,test.ts,test.js,cs,Tests.cs}"
description: "Use when writing or reviewing unit tests, integration tests, or test infrastructure for Java, Node.js, and .NET"
---

# Testing Instructions

## All Stacks

- Use behaviour-focused test names that describe what the system does, not how it is implemented. Prefer `should_return_404_when_order_not_found` over `testGetOrder`.
- Follow Arrange → Act → Assert (AAA) structure with a blank line between each phase.
- Test meaningful behaviour: public contracts, domain rules, edge cases, failure modes. Do not test private implementation details.
- Cover: happy path, validation failures, authorisation failures, null/empty inputs, dependency timeouts, dependency errors, and boundary values.
- Do not sleep in tests (`Thread.sleep`, `setTimeout`); use virtual time, fake clocks, or deterministic awaits.
- Do not write order-dependent tests; each test must be runnable in isolation.
- Keep fixtures small, readable, and specific to the scenario; avoid shared mutable test state.
- Verify important side effects: emitted events, DB writes, outbound HTTP calls. Only verify logs and metrics when they are part of the observable contract.
- Use mocks for external dependencies in unit tests; use real (or containerised) dependencies in integration tests.

## Java

- Use JUnit 5 (`@Test`, `@ParameterizedTest`, `@ExtendWith`) and Mockito for unit tests.
- Use Spring Boot test slices for focused integration tests:
  - `@WebMvcTest` for controller layer (with mocked services).
  - `@DataJpaTest` for repository layer (with in-memory or Testcontainers DB).
  - `@SpringBootTest` for full context integration tests; minimise usage to reduce startup cost.
- Use Testcontainers for database, messaging (Kafka, RabbitMQ), and cache (Redis) integration tests.
- Use `MockMvc` or `WebTestClient` for HTTP-layer tests; assert status codes, response bodies, and headers.
- Use `@MockBean` sparingly in `@SpringBootTest` — prefer `@WebMvcTest` with `@MockBean` or fully wired integration tests.
- Name test classes `<ClassName>Test` for unit tests and `<ClassName>IT` for integration tests.

## Node.js

- Use Jest or Vitest for unit and integration tests.
- Use Supertest for HTTP endpoint tests; do not start a real server — use the `app` instance directly.
- Use `nock` or Mock Service Worker (MSW) to stub outbound HTTP calls in integration tests.
- Avoid `setTimeout` / `setInterval` in tests; use `jest.useFakeTimers()` / Vitest fake timers for time-dependent logic.
- Use `jest.mock()` for module-level mocks in unit tests; prefer dependency injection patterns that make substitution natural.
- Test async code with `async/await` and `expect.assertions(n)` to catch unhandled rejections or missed assertions.
- Name test files `*.test.ts` (unit) or `*.spec.ts` (integration/contract) and co-locate with source files or a dedicated `__tests__/` directory.

## .NET

- Use xUnit for all tests; use `[Fact]` for single-scenario tests and `[Theory]` with `[InlineData]` or `[MemberData]` for parameterised tests.
- Use NSubstitute or Moq for mocks in unit tests; prefer `NSubstitute` for its cleaner syntax.
- Use `WebApplicationFactory<TEntryPoint>` for integration tests of ASP.NET Core endpoints; inject test doubles via `ConfigureTestServices`.
- Use Testcontainers (.NET) or `Respawn` to manage database state between integration tests; never rely on test execution order for DB cleanup.
- Use `FluentAssertions` for readable, descriptive assertions.
- Name test projects `<ProjectName>.Tests` (unit) and `<ProjectName>.IntegrationTests`; name test classes `<ClassName>Tests`.
- Use `CancellationToken.None` in tests unless testing cancellation behaviour explicitly.
