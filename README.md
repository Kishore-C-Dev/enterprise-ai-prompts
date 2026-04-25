# enterprise-ai-prompts

A complete GitHub Copilot configuration library for polyglot enterprise engineering teams. Drop the `.github/` directory into any repository to activate senior-architect-level code reviews, developer assistants, test generation, documentation writing, PR description drafting, incident debugging, and diagram generation for Java, Node.js, and .NET.

---

## What's Inside

### Always-On Instructions (`copilot-instructions.md`)

Loaded into every Copilot interaction in the repository. Establishes the senior architect persona, universal coding standards (security, logging, validation, error handling), severity labels, and review behavior.

### File-Type Instructions (`.github/instructions/`)

Auto-attached to Copilot when you open or edit files matching each instruction's `applyTo` glob. No manual invocation needed.

| File | Applies To | Purpose |
|---|---|---|
| `company-review.instructions.md` | All files | Severity definitions (Blocker/Major/Minor/Question), 10 required review areas, output format, merge recommendation |
| `java-spring.instructions.md` | `.java`, `pom.xml`, `build.gradle`, Spring config | Java 17/21 standards, constructor injection, thin controllers, Optional rules, Problem Details, dependency review |
| `dotnet.instructions.md` | `.cs`, `.csproj`, `.sln`, `appsettings*.json` | ASP.NET Core DI, async/await safety, EF Core, validation, appsettings secrets, xUnit/Testcontainers |
| `nodejs.instructions.md` | `.ts`, `.js`, `.tsx`, `package.json`, lock files | TypeScript strict mode, validation, event loop safety, error middleware, dependency audit |
| `security.instructions.md` | All files | OWASP Top 10 + Blocker checklist: secrets, injection, auth, crypto, containers |
| `api-standards.instructions.md` | Controller, route, and OpenAPI files | Backward compatibility, versioning, DTOs, pagination, idempotency, OpenAPI sync |
| `logging-observability.instructions.md` | All source and config files | Structured logs, correlation IDs, trace context, sensitive data protection, loop noise rules |
| `database.instructions.md` | Repository, DAO, entity, migration, SQL files | Parameterized queries, N+1 prevention, transaction safety, migration rollback plans |
| `ci-cd.instructions.md` | Workflow, Dockerfile, Helm, K8s, Terraform files | Secrets, container security, quality gates, zero-downtime rollout, IAM least privilege |
| `documentation.instructions.md` | Markdown and OpenAPI files | Accuracy, breaking change docs, examples, operational notes |
| `testing.instructions.md` | Test files (all stacks) | AAA style, behavior-focused names, contract tests, regression tests, Testcontainers |
| `springboot.instructions.md` | Java + Spring config files | Spring Boot architecture, configuration, transactions, resilience, Actuator |
| `observability.instructions.md` | All source and config files | SLF4J/Micrometer (Java), pino/winston (Node.js), ILogger/Serilog (.NET) |
| `senior-architect.instructions.md` | All files | Scalability, blast radius, API contracts, data consistency, ADR recommendations |
| `enterprise-custom.instructions.md` | All files | **Placeholder** — fill in your organisation's specific standards |

### Agents (`.github/agents/`)

On-demand assistants invoked in Copilot Chat. Each agent uses tools to read your changed files, search the codebase, and produce structured output.

| Agent | Best Used For |
|---|---|
| `full-stack-code-reviewer` | Primary pre-PR review — comprehensive across correctness, security, tests, API, observability, and docs |
| `pr-description-writer` | Generate a structured PR description with impact, risks, rollback plan, and reviewer focus areas |
| `test-writer` | Generate unit, integration, regression, and contract tests for the detected stack |
| `documentation-writer` | Write missing or outdated docs directly to files — README, `docs/api.md`, `docs/configuration.md`, `docs/runbook.md`, `docs/migration.md`, and OpenAPI YAML |
| `incident-debugger` | Analyze stack traces, logs, and recent changes to find root cause and fix options |
| `security-reviewer` | Deep OWASP Top 10 security audit with Blocker/Major/Minor findings |
| `architecture-reviewer` | Principal engineer lens: scalability, blast radius, API contracts, ADR recommendations |
| `java-senior-reviewer` | Focused deep-dive review for Java/Spring Boot changes |
| `node-senior-reviewer` | Focused deep-dive review for Node.js/TypeScript changes |
| `dotnet-senior-reviewer` | Focused deep-dive review for .NET/C# changes |
| `java-springboot-developer` | Implement Java/Spring Boot features following project conventions |
| `node-developer` | Implement Node.js/TypeScript features following project conventions |
| `dotnet-developer` | Implement .NET/C# features following project conventions |
| `readme-generator` | Generate or regenerate a structured README.md from the codebase |

### Prompts (`.github/prompts/`)

Slash-command prompts invoked from Copilot Chat.

| File | Invoke With | Purpose |
|---|---|---|
| `sequence-diagram.prompt.md` | `/sequence-diagram <flow>` | Mermaid sequence diagram for a feature flow or call chain |
| `dataflow-diagram.prompt.md` | `/dataflow-diagram <system>` | Mermaid flowchart showing data flows and system boundaries |
| `generate-readme.prompt.md` | `/generate-readme` | Scaffold or regenerate `README.md` from the codebase |

---

## Deploying to Another Repository

### Option A — Copy the directory (recommended)

```bash
# From the enterprise-ai-prompts repo root
cp -r .github/ /path/to/your-project/

# Or using rsync to merge without overwriting existing files
rsync -av --ignore-existing .github/ /path/to/your-project/.github/
```

Copilot automatically picks up instruction files via their `applyTo` patterns. Agents and prompts become available in Copilot Chat immediately after VS Code reloads the workspace.

### Option B — Git subtree (stays in sync)

```bash
# In your target repository, add this repo as a remote
git remote add ai-prompts https://github.com/your-org/enterprise-ai-prompts.git
git fetch ai-prompts

# Pull the .github directory as a subtree
git subtree add --prefix=.github ai-prompts main --squash

# To update later
git subtree pull --prefix=.github ai-prompts main --squash
```

### Option C — GitHub template repository

1. Go to this repository's **Settings** → check **Template repository**.
2. When creating a new repository, select this as the template. The `.github/` directory is included automatically.

### Option D — Copy only what you need

Each file is self-contained. Copy only the instruction or agent files relevant to your stack:

```bash
# Java-only project
cp .github/copilot-instructions.md /path/to/project/.github/
cp .github/instructions/java-spring.instructions.md /path/to/project/.github/instructions/
cp .github/instructions/security.instructions.md /path/to/project/.github/instructions/
cp .github/instructions/testing.instructions.md /path/to/project/.github/instructions/
cp .github/agents/java-senior-reviewer.agent.md /path/to/project/.github/agents/
cp .github/agents/full-stack-code-reviewer.agent.md /path/to/project/.github/agents/

# Node.js-only project
cp .github/copilot-instructions.md /path/to/project/.github/
cp .github/instructions/nodejs.instructions.md /path/to/project/.github/instructions/
cp .github/instructions/security.instructions.md /path/to/project/.github/instructions/
cp .github/instructions/testing.instructions.md /path/to/project/.github/instructions/
cp .github/agents/node-senior-reviewer.agent.md /path/to/project/.github/agents/
cp .github/agents/full-stack-code-reviewer.agent.md /path/to/project/.github/agents/

# .NET-only project
cp .github/copilot-instructions.md /path/to/project/.github/
cp .github/instructions/dotnet.instructions.md /path/to/project/.github/instructions/
cp .github/instructions/security.instructions.md /path/to/project/.github/instructions/
cp .github/instructions/testing.instructions.md /path/to/project/.github/instructions/
cp .github/agents/dotnet-senior-reviewer.agent.md /path/to/project/.github/agents/
cp .github/agents/full-stack-code-reviewer.agent.md /path/to/project/.github/agents/
```

---

## Using Agents in VS Code

### How to invoke an agent

1. Open **Copilot Chat** — `Ctrl+Shift+I` (Windows/Linux) or `Cmd+Shift+I` (Mac).
2. Switch to **Agent mode** — click the model selector at the bottom of the chat input and choose **Agent**.
3. Type `@` followed by the agent name. It appears in the autocomplete list.
4. Add your prompt after the agent name and press Enter.

Agents use tools automatically — you do not need to specify them. The tools available vary by agent:

| Tool | Used by |
|---|---|
| `codebase` | All agents — reads files in the workspace |
| `search` | All agents — searches the codebase for symbols and patterns |
| `changes` | Review and writing agents — reads the current branch diff |
| `problems` | Review agents — surfaces compile and lint errors |
| `edit` | `documentation-writer`, `test-writer` — writes content directly to files |

### How to invoke a prompt (slash command)

1. Open Copilot Chat in any mode.
2. Type `/` followed by the prompt name. It appears in the autocomplete list.
3. Add your description after the prompt name and press Enter.

---

## Using Copilot for PR Reviews on GitHub.com

GitHub Copilot can review pull requests directly on GitHub without VS Code:

1. Open a pull request on GitHub.com.
2. In the **Reviewers** panel on the right, click the gear icon.
3. Select **Copilot** from the reviewer list.
4. Copilot reads your repository's `copilot-instructions.md` and instruction files automatically and posts inline review comments.

The `company-review.instructions.md` and `security.instructions.md` files directly shape the quality and focus of Copilot's PR review comments.

---

## Recommended Pre-PR Workflow

Run through this in order before opening a pull request. Each step takes 1–2 minutes.

```
Step 1 — Write and test your code
  Run your test suite locally before reviewing.
  Java:   ./mvnw test  or  ./gradlew test
  .NET:   dotnet test
  Node:   npm test  or  npx vitest run

Step 2 — Self-review with the full-stack reviewer
  In Copilot Chat (Agent mode):
  @full-stack-code-reviewer Review my current branch changes.
  Fix any Blockers and Majors before moving on.

Step 3 — Fill test gaps
  @test-writer Generate missing tests for [service or file name].

Step 4 — Update documentation
  @documentation-writer Review my changes and update any docs that need changing.

Step 5 — Write the PR description
  @pr-description-writer Generate a pull request description for my current branch.

Step 6 — Open the PR and request a Copilot review on GitHub.com
```

---

## Sample Agent Commands

Copy and adapt these in Copilot Chat (Agent mode — `@agent-name prompt`):

### Full-stack code review

```
@full-stack-code-reviewer Review all changes in my current branch. Focus on security, API compatibility, and test coverage.
```

```
@full-stack-code-reviewer Review OrderService.java for correctness, transaction safety, and observability gaps.
```

```
@full-stack-code-reviewer I changed the payment endpoint to accept a new idempotencyKey field. Check backward compatibility and validate the error handling is consistent.
```

### Stack-specific reviews

```
@java-senior-reviewer Review the changes in src/main/java/com/example/payments/ — focus on transaction boundaries and exception handling.
```

```
@node-senior-reviewer Review the new route handlers in src/routes/orders.ts — check async safety and input validation.
```

```
@dotnet-senior-reviewer Review OrderController.cs and OrderService.cs — focus on async/await correctness and EF Core query efficiency.
```

### Security audit

```
@security-reviewer Audit my current branch changes for security vulnerabilities.
```

```
@security-reviewer Review the new file upload endpoint in UploadController.java — check MIME validation, size limits, and path traversal risks.
```

```
@security-reviewer Scan the new Dockerfile and GitHub Actions workflow for secrets and container security issues.
```

### Test generation

```
@test-writer Generate unit and integration tests for PaymentRetryService. Cover happy path, transient failure with retry exhaustion, and permanent failure scenarios.
```

```
@test-writer The OrderController has no tests. Generate @WebMvcTest tests covering POST /orders success, validation failure (missing required fields), and downstream service unavailable.
```

```
@test-writer Generate a contract test for the GET /api/v1/orders/{id} endpoint — verify the response shape matches the OpenAPI spec.
```

### Documentation

The `documentation-writer` agent uses the `edit` tool to write directly to files — it does not print content to the chat window. After it runs, the files below are created or updated on disk and appear as changes in your working tree.

**Where each doc type is written:**

| What you ask for | File written | Format |
|---|---|---|
| Setup, run, env var, or config change | `README.md` | Markdown |
| New or changed API endpoint | `docs/api.md` | Markdown |
| OpenAPI spec update | `openapi.yaml` or `docs/openapi.yaml` | OpenAPI 3.x YAML |
| New or changed config property / env var | `docs/configuration.md` | Markdown |
| Architecture or component change | `docs/architecture.md` | Markdown + Mermaid |
| Breaking change or migration steps | `docs/migration.md` | Markdown |
| New error, alert, or failure mode | `docs/troubleshooting.md` | Markdown |
| New alert or on-call procedure | `docs/runbook.md` | Markdown |
| Changed deployment steps | `docs/deployment.md` | Markdown |
| No `docs/` directory exists yet | Falls back to `README.md` | Markdown |

If the agent cannot determine something from the code alone (e.g., a deployment URL or SLA), it writes a `> **Documentation gap:**` placeholder into the file so the author knows what to fill in manually.

```
@documentation-writer My changes add a new /api/v1/refunds endpoint and a REFUND_TIMEOUT_SECONDS environment variable. Write the API docs and configuration reference.
```

```
@documentation-writer Review the changes in this branch and update any README sections, config docs, or runbook entries that need changing.
```

```
@documentation-writer The order status field was renamed from 'status' to 'orderStatus'. Write a migration guide for consumers of the v1 API.
```

```
@documentation-writer A new PaymentTimeoutException was added. Add a troubleshooting entry and a runbook section describing what triggers it and how on-call should respond.
```

### PR description

```
@pr-description-writer Generate a pull request description for my current branch. This PR adds payment retry logic with exponential backoff.
```

```
@pr-description-writer Write a PR description for the database migration in this branch. The migration adds a non-null column to the orders table.
```

### Incident debugging

```
@incident-debugger I'm seeing this error in production — what is the likely root cause and how do I fix it?

java.lang.NullPointerException: Cannot invoke "com.example.OrderRepository.findById(Long)" because "this.orderRepository" is null
	at com.example.OrderService.getOrder(OrderService.java:42)
```

```
@incident-debugger Production is throwing 500 errors since the last deploy. Here are the logs:

[ERROR] 2025-04-25T14:32:01Z Failed to process payment for order 98765
com.example.PaymentException: Connection refused: payment-service:8080
	at com.example.PaymentClient.charge(PaymentClient.java:55)

What changed recently that could have caused this, and what should I check first?
```

```
@incident-debugger My Jest tests are flaky — they pass locally but fail in CI about 30% of the time with this error:

Error: Timeout - Async callback was not invoked within the 5000ms timeout specified by jest.setTimeout
```

### Architecture review

```
@architecture-reviewer I'm introducing a new PaymentRetryService that calls an external payment provider. Review for scalability, blast radius, and whether I need an ADR.
```

```
@architecture-reviewer Review my changes to the Order domain model — I'm adding a new 'refundStatus' field. Check for API contract risks and data consistency concerns.
```

### Diagram generation

```
/sequence-diagram Payment checkout flow from cart submission to order confirmation, including payment provider call and inventory reservation
```

```
/dataflow-diagram Order ingestion pipeline from API gateway through order service, inventory service, and fulfilment service to the event bus
```

---

## Severity Labels

All review agents and instructions use a consistent four-level severity scale:

| Severity | Meaning | Action |
|---|---|---|
| **Blocker** | Data loss, security breach, production outage, broken build, missing auth | Must be fixed before merge |
| **Major** | Likely bug, broken contract, missing critical test, unsafe transaction | Strong recommendation to fix before merge |
| **Minor** | Observability gap, maintainability debt, inconsistency | Fix at reviewer/author discretion |
| **Question** | Needs clarification before impact can be assessed | Author must respond before merge |

---

## Enterprise Customisation

### 1. Fill in your organisation's standards

Open `.github/instructions/enterprise-custom.instructions.md` and fill in the `# TODO: [ENTERPRISE]` placeholders:

- Approved languages, frameworks, and library versions
- Internal security policies (data classification, cipher suites, SIEM, JWT algorithm allow-list)
- Approved cloud and infrastructure patterns (IaC modules, base image registry)
- Internal service mesh and API gateway conventions
- Compliance requirements (PCI-DSS, HIPAA, SOC 2, GDPR)
- CI/CD quality gates (minimum coverage, required scans, deployment approval rules)
- Approved observability stack (Splunk, Datadog, Dynatrace, Grafana, PagerDuty)
- On-call and incident runbook conventions

Keep the file concise — Copilot loads it on every interaction.

### 2. Adjust instruction strictness

Edit any instruction file to add, remove, or soften rules for your team's context. For example:

- Add approved internal libraries to `nodejs.instructions.md`
- Add your internal secrets manager path pattern to `security.instructions.md`
- Add your DB migration tool conventions to `database.instructions.md`

### 3. Add stack-specific agents

Duplicate an existing agent file and customise it for a specific service, domain, or review persona. Agent files are plain Markdown with YAML frontmatter — no build step required.

---

## Keeping This Configuration Effective

- **Update after incidents.** If a production issue reveals a pattern these instructions would have caught, add a rule to the relevant file.
- **Review quarterly.** API standards, security requirements, and dependency choices evolve. Set a calendar reminder.
- **Keep instruction files focused and short.** Copilot loads them on every interaction — long files dilute focus and slow responses.
- **Treat changes as production code.** Put updates through peer review before merging.
- For a detailed explanation of every file and how the three types (instructions, agents, prompts) interact, see [`.github/AI_REVIEW_README.md`](.github/AI_REVIEW_README.md).

---

## Licence

Internal use — replace this section with your organisation's licence.
