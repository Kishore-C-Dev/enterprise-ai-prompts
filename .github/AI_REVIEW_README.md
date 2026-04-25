# AI Review Configuration — Developer Guide

This directory contains GitHub Copilot configuration files that give every developer on the team access to consistent, opinionated, enterprise-grade AI assistance for code reviews, test generation, documentation, and incident debugging.

---

## What's in This Directory

### Always-On Instructions

| File | Purpose |
|---|---|
| `copilot-instructions.md` | Repository-wide rules loaded into every Copilot interaction — defines the senior architect persona, universal standards, severity labels, and review behavior |

### File-Type-Specific Instructions (`instructions/`)

These are loaded automatically by Copilot when the files you are working on match the `applyTo` glob pattern.

| File | Applies To | Purpose |
|---|---|---|
| `company-review.instructions.md` | All files | Enterprise review standards: severity definitions, required review areas, output format, merge recommendation |
| `java-spring.instructions.md` | `.java`, `pom.xml`, `build.gradle`, Spring config files | Java 17/21 and Spring Boot standards: injection, transactions, DTOs, Optional, error handling, dependency review |
| `dotnet.instructions.md` | `.cs`, `.csproj`, `.sln`, `appsettings*.json` | ASP.NET Core DI, async/await, EF Core, validation, appsettings safety, testing |
| `nodejs.instructions.md` | `.ts`, `.js`, `.tsx`, `package.json`, lock files, `tsconfig.json` | TypeScript strict typing, validation, event loop safety, error middleware, testing |
| `security.instructions.md` | All files | OWASP Top 10 + Blocker severity checklist for secrets, injection, auth, crypto, containers |
| `api-standards.instructions.md` | Controller, route, and OpenAPI files | REST API contracts: backward compatibility, versioning, DTOs, pagination, idempotency, OpenAPI |
| `logging-observability.instructions.md` | All source and config files | Structured logging, correlation IDs, trace context, metrics, no sensitive data in logs |
| `database.instructions.md` | Repository, DAO, entity, migration, and SQL files | Parameterized queries, N+1 prevention, transaction safety, migration safety, rollback plans |
| `ci-cd.instructions.md` | Workflow, Dockerfile, Helm, K8s, and Terraform files | Secrets, container security, quality gates, zero-downtime rollout, IAM least privilege |
| `documentation.instructions.md` | Markdown and OpenAPI files | Accuracy, breaking change docs, examples, operational notes |
| `testing.instructions.md` | Test files (all stacks) | AAA style, behavior-focused names, contract tests, regression tests, Testcontainers |
| `senior-architect.instructions.md` | All files | Architecture lens: scalability, blast radius, API contracts, data consistency, ADR recommendations |
| `springboot.instructions.md` | Java/Spring files | Spring Boot architecture, configuration, transactions, resilience, Actuator |
| `observability.instructions.md` | All source and config files | SLF4J/Micrometer (Java), pino/winston (Node.js), ILogger/Serilog (.NET) — detailed stack-specific guidance |
| `enterprise-custom.instructions.md` | All files | Placeholder for your organisation's specific standards — fill in before deploying to your team |

### On-Demand Agents (`agents/`)

Agents are invoked explicitly in Copilot Chat. They are context-aware: they read your changed files, search the codebase, and apply all instruction files.

| Agent | Purpose |
|---|---|
| `full-stack-code-reviewer` | Primary PR reviewer — comprehensive review across correctness, security, tests, API compatibility, performance, observability, and docs for Java, .NET, and Node.js |
| `test-writer` | Generates unit, integration, regression, and contract tests for the detected stack |
| `security-reviewer` | Deep OWASP Top 10 security audit with Blocker/Major/Minor findings and merge recommendation |
| `documentation-writer` | Identifies and generates missing or outdated README, API, config, runbook, and migration docs |
| `pr-description-writer` | Generates a structured PR description with summary, impact, testing, risks, rollback plan, and reviewer focus areas |
| `incident-debugger` | Analyzes stack traces, logs, and recent changes to identify root cause, debugging steps, and fix options |
| `architecture-reviewer` | Principal engineer lens: scalability, blast radius, API contracts, data consistency, ADR recommendations |
| `java-senior-reviewer` | Focused deep-dive review for Java/Spring Boot changes |
| `node-senior-reviewer` | Focused deep-dive review for Node.js/TypeScript changes |
| `dotnet-senior-reviewer` | Focused deep-dive review for .NET/C# changes |
| `java-springboot-developer` | Implements Java/Spring Boot features following project conventions |
| `node-developer` | Implements Node.js/TypeScript features following project conventions |
| `dotnet-developer` | Implements .NET/C# features following project conventions |
| `readme-generator` | Generates or regenerates a structured README.md from the codebase |

### On-Demand Prompts (`prompts/`)

| Prompt | Invocation | Purpose |
|---|---|---|
| `sequence-diagram` | `/sequence-diagram <flow>` | Generates a Mermaid sequence diagram for a feature flow or call chain |
| `dataflow-diagram` | `/dataflow-diagram <system>` | Generates a Mermaid flowchart showing data flows and system boundaries |
| `generate-readme` | `/generate-readme` | Scaffolds or regenerates README.md with guided sections |

---

## Understanding the Three Types of Files

### `copilot-instructions.md` — Always On
This file is loaded into every Copilot interaction in this repository. It defines the baseline persona, priorities, and universal rules. Think of it as the standing brief that applies to everything Copilot does here.

### `instructions/*.instructions.md` — Triggered by File Type
These files are loaded automatically when Copilot detects that you are working on files matching the `applyTo` pattern in the frontmatter. You do not need to invoke them manually. When you open a `.java` file, `java-spring.instructions.md` is active. When you open a Dockerfile, `ci-cd.instructions.md` is active.

### `agents/*.agent.md` — Invoked on Demand
Agents are purpose-built assistants you invoke in Copilot Chat when you need them. They actively use tools to read your changed code, search the codebase, and produce structured output. The agent's `description` field appears in the agent picker.

---

## How to Use Agents in VS Code Copilot Chat

1. Open the **Copilot Chat** panel (`Ctrl+Shift+I` / `Cmd+Shift+I`).
2. Switch to **Agent mode** — click the model selector dropdown at the bottom of the chat input and choose **Agent**.
3. Type `@` and start typing the agent name (e.g., `@full-stack-code-reviewer`) — it will appear in the autocomplete list.
4. Add your prompt after the agent name.

Agents use tools (`codebase`, `search`, `changes`, `problems`) automatically — you do not need to specify them.

---

## Recommended Developer Workflow

Follow this workflow before opening a pull request:

```
1. Develop your code.

2. Run tests locally.
   - Java:   ./mvnw test  or  ./gradlew test
   - .NET:   dotnet test
   - Node:   npm test  or  npx vitest

3. Self-review with the full-stack reviewer.
   In Copilot Chat (Agent mode):
   @full-stack-code-reviewer Review my current branch changes against main.
   Address any Blockers and Majors before opening the PR.

4. Fill test gaps.
   @test-writer Generate missing tests for the changed code in [file or service name].

5. Update documentation.
   @documentation-writer Review my changes and generate or update any documentation that needs to change.

6. Write the PR description.
   @pr-description-writer Generate a pull request description for my current branch changes.

7. Open the pull request and request a Copilot PR review
   (GitHub.com → Pull Request → "Request a Copilot review").
```

---

## Example Prompts

Copy and adapt these in Copilot Chat (Agent mode):

**Full PR review:**
```
@full-stack-code-reviewer Review all changes in my current branch. Focus on security, API compatibility, and test coverage.
```

**Targeted review of a specific file:**
```
@full-stack-code-reviewer Review OrderService.java for correctness, transaction safety, and observability gaps.
```

**Generate missing tests:**
```
@test-writer Generate unit and integration tests for PaymentRetryService. Cover happy path, transient failure with retry, and permanent failure scenarios.
```

**Write documentation:**
```
@documentation-writer My changes add a new /api/v1/refunds endpoint and a new REFUND_TIMEOUT_SECONDS environment variable. Generate the API docs and configuration reference updates.
```

**Write a PR description:**
```
@pr-description-writer Generate a pull request description for my current branch. This PR adds payment retry logic with exponential backoff.
```

**Debug an incident:**
```
@incident-debugger I'm seeing this error in production:
[paste stack trace or log excerpt here]
It started after the last deployment. What is the likely root cause and how do I fix it?
```

**Security audit:**
```
@security-reviewer Audit my current branch changes for security vulnerabilities. Pay special attention to the new file upload endpoint.
```

**Architecture review:**
```
@architecture-reviewer I'm adding a new PaymentRetryService that calls an external payment provider. Review for scalability, blast radius, and API contract risks.
```

---

## Keeping This Configuration Effective

- **Keep instruction files focused and short.** Copilot loads them on every interaction — long files slow responses and dilute focus.
- **Update after incidents.** If a production incident reveals a pattern the instructions would have caught, add a rule to the relevant instruction file.
- **Fill in `enterprise-custom.instructions.md`.** It contains `# TODO: [ENTERPRISE]` placeholders for your organisation's specific security policies, approved libraries, CI/CD requirements, and observability stack.
- **Review quarterly.** API standards, dependency choices, and security requirements evolve. Set a calendar reminder to review and update the instruction files.
- **Raise a PR for changes.** Treat this configuration as production code — changes should go through peer review.
