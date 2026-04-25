---
applyTo: "**/*"
description: "Enterprise-wide code review standards — severity definitions, required review areas, output format, and merge recommendation"
---

# Company Review Standards

## Severity Definitions

| Severity | Meaning | Action |
|---|---|---|
| **Blocker** | Data loss, security breach, production outage, broken build, missing auth, or unsafe injection | Must be fixed before merge — no exceptions |
| **Major** | Likely bug, broken API contract, critical test gap, unsafe transaction or retry | Strong recommendation to fix before merge |
| **Minor** | Partial failure handling, observability gap, maintainability debt, inconsistent style | Fix at reviewer/author discretion |
| **Question** | Clarification needed before impact can be assessed | Author must respond before merge |

---

## Required Review Areas

Every code review must consider all of the following areas. Skip an area only if it is genuinely not touched by the change.

### 1. Correctness
- No bugs, silent failures, incorrect null handling, or data loss scenarios.
- Business logic matches the stated intent; edge cases and boundary conditions handled.
- Exception handling preserves root causes; no silent swallows.
- Concurrency safety: no shared mutable state, no race conditions.

### 2. Security
- No hardcoded secrets, credentials, keys, or tokens.
- No logging of sensitive data (secrets, PII, tokens, auth headers, SSNs, card numbers).
- Authorization checks present and enforced at the service layer, not just the route/URL layer.
- No injection vectors: SQL, NoSQL, LDAP, XPath, shell, template, expression language.
- No unsafe deserialization, path traversal, insecure file handling, or weak cryptography.

### 3. API Compatibility
- Backward compatibility maintained for any changed endpoints or message schemas.
- Breaking changes are explicitly versioned and consumers notified.
- Request/response DTOs used — persistence entities not exposed.
- OpenAPI/Swagger updated if API shape changes.

### 4. Reliability
- Timeouts configured for all external calls (HTTP, DB, messaging, cache).
- Retries apply only to idempotent, transient failures with backoff and a cap.
- Circuit breakers or bulkheads present for critical downstream dependencies.
- Failure modes degrade gracefully rather than cascading.

### 5. Observability
- Structured (JSON) logging with correlation IDs, trace IDs, and request IDs.
- No sensitive data in logs.
- Metrics instrumented for key business operations, errors, retries, and latencies.
- Trace context propagated across service boundaries.
- Errors include enough context for root-cause analysis without leaking internals.

### 6. Testing
- Unit tests cover business logic and validation rules.
- Integration tests cover DB, messaging, and external API boundaries.
- Tests cover: happy path, validation failure, auth failure, dependency failure, edge cases.
- Bug fixes include a regression test.
- Public API changes include contract tests where external consumers exist.

### 7. Maintainability
- Code is readable and follows existing project patterns.
- No unnecessary complexity, premature abstraction, or dead code.
- Naming is clear and consistent.
- Separation of concerns: transport, business logic, and persistence are not mixed.

### 8. Performance
- No N+1 queries or unbounded DB/API calls.
- No blocking operations in async or event-loop contexts.
- Large result sets paginated; excessive data not loaded into memory.
- No synchronous bottlenecks that would not scale horizontally.

### 9. Documentation
- README, API docs, and configuration docs updated if behavior or setup changes.
- Breaking changes documented with migration steps.
- Operational notes added for support teams if observable behavior changes.

### 10. CI/CD Safety
- No secrets in workflow files, Dockerfiles, or IaC.
- Quality gates (tests, scans) are not bypassed or disabled.
- Deployment supports zero-downtime rollout and has a rollback path.
- Container resource limits, health checks, and probes are correct.

---

## Required Output Format

For each finding, use this structure:

```
**[Severity]** — `path/to/file.ext` (line N, optional)
**Problem:** What is wrong and where.
**Why it matters:** Production impact, risk, or quality consequence.
**Suggested fix:** Concrete code change, pattern, or configuration.
```

Example:

```
**[Blocker]** — `src/main/java/com/example/OrderService.java` (line 42)
**Problem:** User-supplied `orderId` is concatenated directly into a JPQL string.
**Why it matters:** SQL/JPQL injection allows an attacker to access or delete arbitrary orders.
**Suggested fix:** Use a named parameter: `"SELECT o FROM Order o WHERE o.id = :id"` with `.setParameter("id", orderId)`.
```

---

## Merge Recommendation

End every review with one of:

| Recommendation | Meaning |
|---|---|
| **Safe to merge** | No significant issues; change is ready. |
| **Safe to merge after minor fixes** | Only Minor or Question findings; can merge once addressed. |
| **Needs changes before merge** | One or more Major findings that should be resolved first. |
| **Do not merge** | One or more Blocker findings; change must not be merged in current state. |
