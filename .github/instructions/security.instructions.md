---
applyTo: "**/*.{java,ts,js,mjs,cjs,cs,csproj,yml,yaml,json,properties,xml}"
description: "Use when writing or reviewing security-sensitive code — covers OWASP Top 10, secrets, injection, auth, and TLS for Java, Node.js, and .NET"
---

# Security Instructions

## All Stacks

- Never add secrets, credentials, tokens, certificates, private keys, or API keys to source files or test fixtures.
- Do not log sensitive data: passwords, tokens, cookies, authorization headers, session IDs, SSNs, account numbers, card numbers (PAN), PHI, PII, or sensitive customer payloads.
- Validate and sanitise all inputs at trust boundaries before using them in SQL, LDAP, XPath, shell commands, file paths, URLs, templates, or log output.
- Use parameterised queries or ORM-safe query builders — never concatenate user input into query strings.
- Prevent path traversal: validate and normalise file paths before file access; reject `..` sequences.
- Avoid injection in OS commands: prefer library calls over `exec`/`Runtime.exec`/`Process.Start`; never pass unsanitised user input to shell.
- Enforce authorisation checks before reading or modifying protected resources — authentication is not sufficient.
- Use least-privilege for service accounts, database users, message queues, topics, cloud roles, and storage buckets.
- Apply HTTPS/TLS for all external communication; reject invalid or self-signed certificates in production.
- Set secure defaults for CORS (explicit allow-list), CSRF protection, HTTP security headers (`Strict-Transport-Security`, `X-Content-Type-Options`, `X-Frame-Options`, `Content-Security-Policy`).
- Hash passwords with bcrypt, scrypt, or Argon2 — never MD5, SHA-1, or plain SHA-256 for passwords.
- Use constant-time comparison for secrets and tokens to prevent timing attacks.
- Avoid unsafe deserialisation of untrusted payloads.
- Flag missing authentication and authorisation as **Critical** severity.

## Java

- Avoid Java native deserialisation (`ObjectInputStream`) with untrusted data; prefer JSON/protobuf with explicit schema.
- Do not expose Spring Boot Actuator endpoints (`/actuator/**`) without authentication; restrict sensitive endpoints (`/env`, `/heapdump`, `/threaddump`) entirely in production.
- Use Spring Security's `@PreAuthorize` / `@Secured` for method-level authorisation; do not rely solely on URL pattern matching.
- Prefer `@ConfigurationProperties` + Vault/Secrets Manager for credentials; never use `@Value("${password}")` bound to a plain config file committed to source.
- Set `server.error.include-stacktrace=never` and `server.error.include-message=never` in production profiles.
- Validate `@RequestBody`, `@RequestParam`, and `@PathVariable` inputs using Bean Validation (`@NotNull`, `@Size`, `@Pattern`).

## Node.js

- Use `helmet` to set HTTP security headers on all Express / Fastify / NestJS applications.
- Validate request bodies with `express-validator`, `joi`, or `zod` — never trust `req.body` directly.
- Avoid `eval()`, `new Function()`, `vm.runInNewContext()`, and `child_process.exec()` with user-controlled input.
- Verify JWT signatures server-side on every request; never trust a decoded payload without verification.
- Guard against SSRF: validate and allowlist URLs before making outbound HTTP requests on behalf of users.
- Set `httpOnly`, `secure`, and `sameSite` flags on all session and authentication cookies.
- Audit `package.json` dependencies with `npm audit` or `snyk`; avoid packages with known critical CVEs.
- Use `crypto.timingSafeEqual` for token comparisons; avoid `===` for secret comparison.

## .NET

- Use `[Authorize]` attributes and policy-based authorisation; never rely solely on route-level security.
- Use `IDataProtection` API or Azure Key Vault / AWS Secrets Manager for secrets — never store them in `appsettings.json`.
- Always check `ModelState.IsValid` (or use `[ApiController]` automatic validation) before processing requests.
- Avoid `JsonSerializer.Deserialize<dynamic>` or `JObject` with untrusted content; use strongly-typed models.
- Configure `DataProtection` key storage and lifetime explicitly; do not rely on in-memory default keys in production.
- Set `UseHsts()`, `UseHttpsRedirection()`, and configure CORS with an explicit allow-list.
- Use `IHttpClientFactory` for all outbound HTTP calls — never create raw `HttpClient` instances in dependency-injected code.
- Enable nullable reference types (`<Nullable>enable</Nullable>`) to reduce null-dereference risks.
