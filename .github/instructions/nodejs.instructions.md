---
applyTo: "**/*.js,**/*.ts,**/*.tsx,package.json,package-lock.json,yarn.lock,pnpm-lock.yaml,tsconfig.json"
description: "Node.js and TypeScript review standards — type safety, validation, async safety, event loop, error handling, and testing"
---

# Node.js and TypeScript Instructions

## TypeScript

- Enable `"strict": true` in `tsconfig.json` — this activates `strictNullChecks`, `noImplicitAny`, and related guards.
- Avoid `any` unless there is no alternative; justify it with an inline comment explaining why. Use `unknown` and narrow types instead.
- Use explicit return types on public functions and exported symbols — improves readability and catches return-type regressions.
- Prefer `interface` for extensible object shapes and `type` for unions, intersections, and computed types.
- Use `readonly` on properties that should not be mutated after construction.

## Input Validation

- Validate all inputs at API boundaries (HTTP controllers, message consumers, scheduled job triggers) before processing.
- Use a project-approved validation library: `zod`, `joi`, `class-validator` (NestJS), or `express-validator`.
- Never trust `req.body`, `req.query`, or `req.params` directly — treat them as untrusted until validated and typed.
- Return `400 Bad Request` with a structured validation error body — not a raw thrown error.

## Async Safety

- Use `async/await` for all asynchronous code. Avoid raw `.then()/.catch()` chains in complex logic.
- Every `async` function must either be `await`ed, returned, or have its rejection handled — unhandled rejections crash the process.
- Never `.catch(err => {})` an error silently — either handle it, log it, or rethrow it.
- Use `Promise.all` or `Promise.allSettled` for parallel independent async operations.
- Avoid `new Promise()` wrappers around already-promisified APIs — use `util.promisify` or the native async API.

## Event Loop

- Never block the event loop in request handlers:
  - No `fs.readFileSync`, `fs.writeFileSync`, or other sync filesystem calls.
  - No `child_process.execSync` or `spawnSync`.
  - No CPU-bound loops (image processing, crypto, large JSON parsing) on the main thread — offload to `worker_threads`.
- Avoid synchronous network operations.

## Error Handling

- Use a centralised error handler:
  - **Express**: `app.use((err, req, res, next) => { ... })` — all errors must reach this middleware.
  - **NestJS**: use `ExceptionFilter` and `@Catch()` decorators.
  - **Fastify**: use `fastify.setErrorHandler`.
- Never handle errors inline per route — it leads to inconsistent responses.
- Install `express-async-errors` (Express) or use NestJS guards to ensure async route errors propagate to the error handler.
- Never return stack traces, internal module paths, or DB error details in API responses.

## Route Handlers and Services

- Route/controller handlers must be thin: call a service, return the result. No business logic inline.
- Business logic belongs in service classes.
- DB access belongs in repository classes or data-access modules.
- NestJS: use `Guards` for authentication/authorisation, `Pipes` for validation, `Interceptors` for cross-cutting concerns (logging, timing).

## Environment Variables and Configuration

- Access `process.env` only through a validated config module (e.g., `envalid`, `zod`-validated config). Never access `process.env.MY_VAR` ad-hoc in business logic.
- Validate required env vars at startup — fail fast with a clear error rather than discovering a missing var at runtime.
- Never log or expose `process.env` contents — they contain secrets.

## Sensitive Data

- Never log: secrets, tokens, authorization headers, passwords, PII, SSNs, account numbers, card numbers.
- Redact sensitive fields in logging serializers before writing to the log sink.
- Set `httpOnly`, `secure`, and `sameSite=strict|lax` on all session and auth cookies.

## Dependency Management

- Review `package.json` changes carefully:
  - Justify every new dependency — prefer fewer, well-maintained packages.
  - Pin exact versions or use lockfile (`package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`) — never use `*` or `latest`.
  - Run `npm audit` / `yarn audit` / `pnpm audit` and flag critical or high CVEs as **Blocker** findings.
- Avoid deprecated or unmaintained packages.

## Testing

- **Unit tests**: Jest or Vitest. Name files `*.test.ts`. Use `jest.mock()` or `vi.mock()` for module-level mocks.
- **API integration tests**: Supertest against the `app` instance (do not start a real server).
- **External HTTP stubs**: `nock` or Mock Service Worker (MSW).
- **DB/messaging integration**: Testcontainers for realistic infrastructure behavior.
- Test naming: describe behavior — `should return 404 when order does not exist`, not `testGetOrder`.
- Use `expect.assertions(n)` in async tests to catch unhandled promise resolutions or missed assertions.
- Avoid `setTimeout`/`setInterval` in tests — use `jest.useFakeTimers()` / Vitest fake timers.
