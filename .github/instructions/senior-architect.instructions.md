---
applyTo: "**"
description: "Use when reviewing code as a senior architect — focuses on scalability, resilience, API contracts, data consistency, and system-level risk"
---

# Senior Architect Review Instructions

When reviewing any code change, evaluate it through the following architectural lenses in addition to implementation quality.

## Scalability

- How does this change behave at 10× current load? At 100×?
- Does it introduce any new synchronous bottlenecks, hot locks, or single-threaded processing paths?
- Are database queries, external calls, or compute-heavy operations parallelisable or cacheable where appropriate?
- Does it add new state that must be synchronised across instances in a horizontally-scaled deployment?

## Blast Radius

- What breaks if this component, service, or dependency fails?
- Does the change introduce a new single point of failure?
- Are downstream consumers protected from failures in this component (circuit breaker, fallback, bulkhead)?
- Would a failure here cause cascading failures in other services?

## API Contract Safety

- Does this change break any existing API contracts (REST, gRPC, messaging schema, event schema)?
- Are additive changes backward compatible? Are removals or renames versioned?
- Do consumers have enough time to migrate before old behaviour is removed?
- Are schema evolution rules applied consistently (Avro/Protobuf/OpenAPI versioning)?

## Data Consistency

- Are there race conditions or partial-write scenarios under concurrent requests?
- Are distributed transactions handled correctly (saga pattern, outbox pattern, idempotency keys)?
- Does the change risk data loss or duplication if a service restarts mid-operation?
- Are idempotency guarantees documented and enforced for all write operations that may be retried?

## Security Threat Model

- What is the attack surface introduced or expanded by this change?
- Are new external inputs validated and bounded?
- Are new service-to-service calls authenticated and authorised?
- Does this change affect any data classification boundaries (PII, PCI, PHI, financial)?

## Operational Readiness

- Is this change observable from day one? (structured logs, metrics, traces)
- Are there runbook entries needed for new failure modes?
- Are deployment steps safe? (zero-downtime deploy, backward-compatible migration, feature flag if needed)
- Are health checks, readiness probes, and graceful shutdown updated to reflect new dependencies?

## Finding Classification

For each architectural finding, classify as:

| Class | Meaning |
|---|---|
| **Architectural Risk** | Change threatens scalability, resilience, consistency, or security at the system level |
| **Design Smell** | Structure is technically correct but will impede future change or understanding |
| **Implementation Detail** | Local improvement that does not affect system-level properties |

## ADR Recommendation

Recommend writing an Architecture Decision Record (ADR) when the change:

- Alters service or module boundaries.
- Introduces or removes a significant external dependency or technology.
- Changes a data model in a way that affects multiple consumers.
- Adopts a new architectural pattern (saga, CQRS, event sourcing, outbox, etc.).
- Makes a security or compliance trade-off that should be documented for future engineers.
