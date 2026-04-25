---
applyTo: "**/*.java,**/*.cs,**/*.js,**/*.ts,**/*.yml,**/*.yaml,**/*.json"
description: "Logging, metrics, tracing, and supportability standards — structured logs, correlation IDs, sensitive data protection, trace context, and loop noise"
---

# Logging and Observability Instructions

## Structured Logging

- Use structured (JSON) logging in all environments — key/value pairs in every log entry, never freeform string concatenation.
- Every log entry in a request context must include: `correlationId`, `traceId`, `requestId`, and relevant business identifiers (e.g., `orderId`, `customerId`, `transactionId`) where safe.
- Log at appropriate levels:
  - `DEBUG` — developer detail; useful during local debugging or targeted tracing. Do not leave DEBUG statements in merged code that fire on every request.
  - `INFO` — meaningful lifecycle events: service started, request processed, job completed, message consumed.
  - `WARN` — recoverable issues: retried operation, degraded mode, unexpected-but-non-fatal state.
  - `ERROR` — failures requiring attention: unhandled exception, dependency unavailable, data integrity issue.

## Sensitive Data — Never Log

The following must **never** appear in any log output, at any level, in any environment:

- Passwords, PINs, security questions.
- Tokens, API keys, OAuth credentials.
- Authorization or authentication headers (`Authorization`, `Cookie`, `X-API-Key`).
- Session identifiers.
- SSNs, national identity numbers, government IDs.
- Account numbers, card numbers (PAN), CVV, expiry dates.
- Customer PII: names (combined with other identifiers), addresses, email addresses, phone numbers, health data.
- Full request or response payloads unless the payload is explicitly classified as non-sensitive by the data owner.

Log the ID or a safe reference instead of the sensitive value.

## Production Path Restrictions

The following are **Blocker** findings in production code paths:

- `System.out.println` (Java) — use SLF4J/Logback instead.
- `Console.WriteLine` / `Console.Write` (C#) — use `ILogger<T>` instead.
- Raw `console.log` / `console.error` / `console.warn` (Node.js) — use `pino` or `winston` instead, unless the project explicitly uses a console transport through an approved logger wrapper.
- `e.printStackTrace()` (Java) — use `log.error("message", exception)` instead.

## Trace Context

- Propagate W3C `traceparent` and `tracestate` headers across all service-to-service HTTP calls and message queue headers (Kafka, RabbitMQ, SQS).
- Incoming trace context must be extracted and attached to the current span — do not start a new root trace if a parent trace exists.
- Include `traceId` and `spanId` in every log entry for the request or message processing context.

## Error Log Quality

Each ERROR-level log must include:

1. What operation failed (e.g., `"Failed to process payment for order {orderId}"`).
2. Which dependency was involved (DB, downstream service name, queue name).
3. What input triggered it — redacted if sensitive (e.g., `"orderId={orderId}, amount={amount}"`).
4. The full exception (not just `exception.getMessage()`) so the root cause is preserved.

Never log `"An error occurred"` without context.

## Metrics

Instrument the following with counters, histograms, or gauges as appropriate:

- Request latency (histogram per endpoint).
- Request error rate (counter, broken down by status code or error type).
- Downstream call latency and error rate (per dependency).
- Retry count and retry exhaustion events.
- Timeout count per dependency.
- Queue depth and consumer lag (for messaging systems).
- Key business events: orders placed, payments processed, jobs completed.

## Avoiding Log Noise

- Do not emit `INFO` or `DEBUG` log statements inside tight loops, high-frequency batch iterations, or per-item processing — emit a summary at the boundary instead.
- Do not log every cache hit or cache miss in hot paths — aggregate or sample.
- Do not add logging for expected, recoverable states (e.g., "User not found" for a lookup endpoint) at `WARN` or `ERROR` — use `DEBUG` or `INFO`.

## Stack-Specific Notes

### Java
- Use SLF4J + Logback or Log4j2; never `java.util.logging` in application code.
- Use `MDC.put(key, value)` for correlation context; clear with `MDC.clear()` at the end of each thread-bound scope.
- Log exceptions: `log.error("Failed to process {}", orderId, exception)` — the exception must be the last argument.

### .NET
- Use `ILogger<T>` with message templates: `_logger.LogError(ex, "Failed to process order {OrderId}", orderId)`.
- Never use string interpolation in log statements — it bypasses structured field extraction.
- Configure Serilog or NLog sinks for JSON output; include `{TraceId}` and `{SpanId}` enrichers.

### Node.js
- Use `pino` (preferred) or `winston`; configure `redact` or `serializers` to scrub sensitive fields before output.
- Create child loggers per request: `logger.child({ requestId, traceId })` and pass the child through the request lifecycle.
- Never use `console.log` in production code paths — it bypasses redaction and structured serialization.
