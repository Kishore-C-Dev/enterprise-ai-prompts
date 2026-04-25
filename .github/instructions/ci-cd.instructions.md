---
applyTo: ".github/workflows/**/*.yml,.github/workflows/**/*.yaml,Jenkinsfile,Dockerfile,docker-compose*.yml,docker-compose*.yaml,helm/**/*.yaml,k8s/**/*.yaml,**/*.tf"
description: "CI/CD, Docker, Kubernetes, and deployment safety — secrets, container security, quality gates, rollout safety, and least privilege"
---

# CI/CD, Container, and Deployment Instructions

## Secrets and Credentials — Blocker

- Never embed secrets, API keys, passwords, tokens, or certificates directly in workflow files, Dockerfiles, `docker-compose` files, Helm values, Kubernetes manifests, or Terraform files.
- Use CI/CD secret variables (GitHub Actions Secrets, Jenkins Credentials, GitLab CI Variables) and reference them as environment variables.
- Use a secrets manager (Vault, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager) for runtime secret injection — not mounted files with embedded plaintext.
- Flag any `echo "$SECRET"` or similar patterns that could leak secrets to logs.
- Never bake secrets into Docker image layers via `RUN`, `ENV`, or `COPY` — they persist in the image history and are extractable.

## Container Security

- Avoid `privileged: true` in Kubernetes pod specs and `docker-compose` unless explicitly justified with a comment explaining why it is necessary.
- Run containers as a non-root user: use `USER <uid>` in Dockerfiles and `runAsNonRoot: true` in Kubernetes `securityContext`.
- Use minimal base images (distroless, Alpine, or slim variants) to reduce attack surface.
- Pin base image tags to specific versions or digests — `FROM node:20-alpine` is better than `FROM node:latest`; a digest (`@sha256:...`) is better still.
- Drop unnecessary Linux capabilities: `capabilities: drop: ["ALL"]` and add back only what is required.
- Set `readOnlyRootFilesystem: true` in Kubernetes `securityContext` where the container does not need to write to the filesystem.

## Dependency and Action Version Pinning

- GitHub Actions: pin actions to a specific release tag (`actions/checkout@v4`) — never `@main` or `@master` (supply-chain risk).
- Package lock files (`package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `Gemfile.lock`) must be committed and used in CI — never run `npm install` without the lockfile.
- Terraform: pin provider versions in `required_providers` with `~>` or exact versions; use remote state with locking.

## Quality Gates — Must Not Be Bypassed

Every pipeline must run, in this order:

1. **Unit tests** — must pass before integration tests.
2. **Integration tests** — must pass before deployment steps.
3. **SAST scan** (Semgrep, SonarQube, CodeQL, or equivalent).
4. **Dependency vulnerability scan** (`npm audit`, `trivy`, `snyk`, `mvn dependency-check`, `dotnet list package --vulnerable`).
5. **Container image scan** (Trivy, Grype, or equivalent) for any Docker image build.

- Never disable checks with `--no-verify`, `continue-on-error: true` on security scans, or `--skip-tests` flags without explicit documented approval.
- A failing pipeline must block the deployment step — do not use `|| true` or `exit 0` to hide failures.

## Deployment Safety

- All production deployments must support **zero-downtime rollout**: rolling updates, blue/green, or canary deployment.
- Use **feature flags** for significant new functionality — decouple deploy from release.
- Every deployment must have a documented or automated **rollback path** — a one-command or one-click rollback to the previous version.
- Deployments to production require a passing pipeline, peer review approval, and (where required by policy) a change record.
- Avoid applying `terraform apply` or equivalent destructive IaC commands directly — gate them behind a CI approval step.

## Kubernetes Resource Configuration

- Set explicit CPU and memory `requests` and `limits` on all workloads — missing limits is a **Major** finding.
- **Liveness probe**: must reflect whether the process is running and not deadlocked. Do not use a liveness probe that hits the DB — it will kill healthy pods when the DB is slow.
- **Readiness probe**: must reflect whether the service can handle traffic, including dependency health. Use it to gate traffic routing.
- **Startup probe**: use for services with slow startup to avoid premature liveness failures.
- Configure `terminationGracePeriodSeconds` to allow in-flight requests to complete.
- Set `PodDisruptionBudget` for critical services to avoid zero-instance scenarios during rolling updates.

## IAM and Least Privilege

- Service accounts, IAM roles, and cloud permissions must follow least privilege:
  - No `"Action": "*"` or `"Resource": "*"` in IAM policies — flag these as **Blocker**.
  - No broad `cluster-admin` ClusterRoleBindings in Kubernetes.
  - No `admin` service accounts for application workloads.
- Use workload identity / IRSA / pod identity for cloud API access — never mount long-lived AWS/GCP/Azure credentials as environment variables or files.

## Environment Configuration Safety

- Review environment-specific configuration carefully — production settings must not be hardcoded in source.
- Separate dev/staging/prod configuration clearly; never copy production credentials into lower environments.
- Helm values files for production (`values-prod.yaml`) should not contain plaintext secrets — reference external secrets operator or vault agent.

## Terraform

- `terraform plan` output must be reviewed and approved before `terraform apply` runs.
- Use remote state with locking (S3 + DynamoDB, Terraform Cloud, etc.) — never use local state for shared infrastructure.
- Avoid `terraform destroy` in automated pipelines without an explicit human approval gate.
- Tag all cloud resources with environment, owner, and service name for cost allocation and incident response.
