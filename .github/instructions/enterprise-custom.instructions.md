---
applyTo: "**"
description: "Enterprise-specific custom standards — fill in this file with your organisation's coding guidelines, security policies, and compliance requirements"
---

# Enterprise Custom Instructions

> **Instructions for maintainers:** Replace every `# TODO: [ENTERPRISE]` block below with your organisation's standards. Remove sections that are not applicable. Keep this file concise — Copilot loads it on every interaction.

---

## Coding Standards and Approved Libraries

<!-- # TODO: [ENTERPRISE] Add your organisation's approved languages, frameworks, and library versions. Example:
- Approved Java version: Java 21 LTS
- Approved Spring Boot version: 3.3.x
- Approved Node.js LTS: 22.x
- Approved .NET version: .NET 9
- Banned libraries: Log4j 1.x, commons-collections <3.2.2, lodash <4.17.21
- Mandatory code formatter: Checkstyle (Java), Prettier (Node.js), dotnet-format (.NET)
- Mandatory static analysis: SonarQube, Semgrep, Snyk
-->

## Internal Security Policies

<!-- # TODO: [ENTERPRISE] Add your security classification rules and cipher requirements. Example:
- All data classified as Confidential or higher must be encrypted at rest (AES-256) and in transit (TLS 1.2+).
- TLS 1.0 and 1.1 are prohibited on all endpoints.
- Approved JWT signing algorithms: RS256, ES256. HS256 is not permitted for inter-service tokens.
- All secrets must be stored in [Vault / AWS Secrets Manager / Azure Key Vault] — no plaintext secrets in config files.
- SIEM integration: all authentication events, authorisation failures, and data access for Confidential data must be forwarded to [Splunk / Elastic SIEM].
- Penetration test sign-off required before go-live for any internet-facing API.
-->

## Cloud and Infrastructure

<!-- # TODO: [ENTERPRISE] Add your approved cloud providers, IaC tools, and deployment conventions. Example:
- Approved cloud: AWS (primary), Azure (secondary).
- IaC: Terraform modules from the internal platform team's registry.
- Container base images: use only images from the internal approved registry (registry.internal.example.com).
- Kubernetes namespace conventions: <team>-<env> (e.g., payments-prod, payments-staging).
- Do not create IAM roles or policies outside the platform team's Terraform modules.
-->

## Service Mesh and API Gateway

<!-- # TODO: [ENTERPRISE] Describe your internal networking conventions. Example:
- All inter-service calls must go through the internal service mesh (Istio / Linkerd).
- External APIs must be published through the API Gateway (Kong / AWS API Gateway).
- mTLS is mandatory for all east-west (service-to-service) traffic in production.
- Service registration: use the internal service catalogue (Backstage) for all new services.
-->

## Compliance Requirements

<!-- # TODO: [ENTERPRISE] List your active compliance frameworks and what they require. Example:
- PCI-DSS: Any code handling card data must be reviewed by the security team before merge.
- GDPR / Data Privacy: PII fields must be annotated with @PersonalData; retention and deletion flows must be documented.
- SOC 2 Type II: All infrastructure changes must have an associated change ticket in ServiceNow.
- HIPAA: PHI must not leave the approved data residency region; log all access to PHI records.
-->

## CI/CD and Quality Gates

<!-- # TODO: [ENTERPRISE] Define your pipeline requirements. Example:
- All pipelines must run unit tests, integration tests, SAST (Semgrep), SCA (Snyk), and container scanning (Trivy).
- Minimum code coverage threshold: 80% line coverage on changed files.
- No merge to main without a passing pipeline and at least 1 approved code review.
- Deployment to production requires a change record in ServiceNow and approval from the on-call engineer.
- Feature flags: use LaunchDarkly / internal feature flag service for all production rollouts of significant features.
-->

## Observability Stack

<!-- # TODO: [ENTERPRISE] Specify your approved logging, metrics, and tracing infrastructure. Example:
- Log shipping: Fluent Bit → Splunk (production), Elasticsearch (non-production).
- Metrics: Prometheus scraped by Thanos; dashboards in Grafana. Alert routing via PagerDuty.
- Distributed tracing: Jaeger / Datadog APM / Dynatrace. Use OTLP exporter.
- Log retention: 90 days hot, 1 year cold (S3 Glacier).
- Mandatory log fields: service, env, version, traceId, spanId, correlationId.
-->

## On-Call and Incident Runbooks

<!-- # TODO: [ENTERPRISE] Link to your runbook templates and incident process. Example:
- Runbook location: Confluence space "Engineering Runbooks" — create a page for every new service.
- Incident severity levels: SEV1 (customer-impacting outage), SEV2 (degraded), SEV3 (minor).
- On-call rotation: PagerDuty schedule "Backend On-Call"; SEV1 pages immediately, SEV2 within 15 min.
- Post-incident review required for all SEV1 and SEV2 incidents within 5 business days.
- New alerts must have a corresponding runbook entry before going to production.
-->
