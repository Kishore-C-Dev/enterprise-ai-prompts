---
applyTo: "**/*.{java,ts,js,cs,yml,yaml,json,properties}"
description: "Use when writing or reviewing logging, metrics, tracing, or alerting code for Java, Node.js, and .NET"
---

# Observability Instructions

## All Stacks

- Use structured (JSON) logging in all environments — key/value pairs, not freeform strings.
- Include correlation IDs, trace IDs, request IDs, and relevant business identifiers (order ID, customer ID, transaction ID) in every log entry where safe.
- Never log secrets, credentials, tokens, session IDs, passwords, card numbers, PHI, PII, or full request/response payloads unless the payload is explicitly classified as non-sensitive.
- Log at appropriate levels:
  - `DEBUG` — developer detail useful during local development or targeted debugging.
  - `INFO` — meaningful lifecycle events: service start/stop, request handled, job completed, message processed.
  - `WARN` — recoverable issues: retried operation, degraded mode, unexpected but non-fatal state.
  - `ERROR` — failed operations requiring attention: unhandled exception, dependency unavailable, data integrity issue.
- Propagate `traceparent` / W3C Trace Context headers across all service-to-service calls.
- Instrument key operations with metrics: request latency (histogram), success/failure counts (counter), dependency call latency, retry counts, queue depth, and timeout rates.
- Ensure errors include enough context for root-cause analysis: what operation failed, what input triggered it (redacted if sensitive), what dependency was involved.
- Do not log excessive payloads that would overwhelm log storage or expose sensitive data in aggregated form.

## Java

- Use SLF4J as the logging facade; bind to Logback or Log4j2.
- Use `MDC.put(key, value)` to attach correlation IDs and request context; clear MDC at the end of each thread-bound scope with `MDC.clear()`.
- Use Micrometer for metrics: `Counter`, `Timer`, `Gauge`, `DistributionSummary`; register custom meters via `MeterRegistry`.
- Instrument Spring Boot apps with `spring-boot-starter-actuator` and expose `/actuator/metrics`, `/actuator/health`, and `/actuator/prometheus` (secured appropriately).
- Use OpenTelemetry Java agent or Micrometer Tracing for distributed tracing; propagate `traceparent` through `RestTemplate` / `WebClient` / Kafka headers.
- Log exceptions with `log.error("message", exception)` — never log `exception.getMessage()` only, as root causes are lost.

## Node.js

- Use `pino` (preferred) or `winston` for structured JSON logging; configure `serializers` to redact sensitive fields.
- Attach correlation IDs to the logger child context: `logger.child({ requestId, traceId })` and pass the child logger through the request lifecycle.
- Use `express-async-errors` or equivalent to ensure unhandled promise rejections in route handlers reach the global error middleware.
- Install and configure the OpenTelemetry Node.js SDK (`@opentelemetry/sdk-node`) for distributed tracing; propagate `traceparent` via HTTP headers.
- Expose a `/health` or `/healthz` endpoint that reports dependency status (DB, cache, message broker) for liveness and readiness probes.
- Use `prom-client` or OpenTelemetry metrics SDK to expose Prometheus-compatible metrics; instrument HTTP request duration, error rates, and external call latency.

## .NET

- Use `ILogger<T>` with structured message templates: `_logger.LogInformation("Order {OrderId} processed", orderId)` — not string interpolation.
- Configure Serilog or NLog as the logging sink with JSON output format for production; include `{TraceId}` and `{SpanId}` enrichers.
- Use `Activity` and `ActivitySource` (System.Diagnostics) for distributed tracing; propagate W3C `traceparent` headers via `HttpClient` and messaging.
- Register `IMeterFactory` and use `System.Diagnostics.Metrics` (`Meter`, `Counter<T>`, `Histogram<T>`) for custom metrics.
- Expose health checks via `AddHealthChecks()` and `MapHealthChecks("/health")`, including dependency checks (EF Core, Redis, external HTTP).
- Use `IHttpContextAccessor` to read the current trace/correlation ID and include it in downstream calls and log context.
- Configure OpenTelemetry exporters (`AddOtlpExporter`, `AddPrometheusExporter`) for metrics and traces.
