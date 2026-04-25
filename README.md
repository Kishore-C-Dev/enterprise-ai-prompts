# enterprise-ai-prompts

A complete GitHub Copilot prompt library for polyglot enterprise engineering teams. Drop the `.github/` directory into any repository to activate senior-architect-level code reviews, developer assistants, diagram generators, and a README generator for Java, Node.js, and .NET.

---

## What's Inside

### Instructions (`.github/instructions/`)

Auto-attached to Copilot based on file type. Applied when writing or editing code.

| File | Applies To | Purpose |
|---|---|---|
| `security.instructions.md` | Java, Node.js, .NET source files | OWASP Top 10, secrets, injection, auth, TLS |
| `database.instructions.md` | Java, Node.js, .NET + SQL files | Parameterised queries, transactions, migrations, pagination |
| `observability.instructions.md` | Java, Node.js, .NET source + config | Structured logging, metrics, distributed tracing |
| `springboot.instructions.md` | Java + Spring config files | Architecture layers, transactions, Actuator, resilience |
| `testing.instructions.md` | Java, Node.js, .NET test files | AAA structure, test scope, mocking, framework conventions |
| `senior-architect.instructions.md` | All files | Scalability, blast radius, API contracts, ADR recommendations |
| `enterprise-custom.instructions.md` | All files | **Placeholder** — fill in your organisation's standards |

### Prompts (`.github/prompts/`)

On-demand prompts invoked from Copilot Chat.

| File | Invoke With | Purpose |
|---|---|---|
| `sequence-diagram.prompt.md` | `/sequence-diagram <flow description>` | Generates a Mermaid `sequenceDiagram` for a feature or call chain |
| `dataflow-diagram.prompt.md` | `/dataflow-diagram <system description>` | Generates a Mermaid `flowchart TD` showing data flows and boundaries |
| `generate-readme.prompt.md` | `/generate-readme` | Scaffolds or regenerates `README.md` from the codebase |

### Agents (`.github/agents/`)

Persistent Copilot agents invoked from Copilot Chat or the agent picker.

| Agent | Purpose |
|---|---|
| `java-senior-reviewer` | Full Java / Spring Boot code review: security, DB, observability, testing |
| `node-senior-reviewer` | Full Node.js / TypeScript code review: async safety, security, DB, observability |
| `dotnet-senior-reviewer` | Full .NET / C# code review: null safety, EF Core, security, observability |
| `security-reviewer` | Deep OWASP Top 10 security audit for any stack |
| `architecture-reviewer` | System design, API contracts, resilience patterns, ADR recommendations |
| `readme-generator` | Generates or updates `README.md` from repository context |
| `java-springboot-developer` | Implements Java / Spring Boot features following project conventions |
| `node-developer` | Implements Node.js / TypeScript features following project conventions |
| `dotnet-developer` | Implements .NET / C# features following project conventions |
| `test-writer` | Generates unit and integration tests for Java, Node.js, or .NET |

### Chatmodes (`.github/chatmodes/`)

Chat session modes that set the review persona and output format for the session.

| Chatmode | Purpose |
|---|---|
| `java-code-review` | Senior architect Java review session |
| `node-code-review` | Senior architect Node.js review session |
| `dotnet-code-review` | Senior architect .NET review session |
| `security-review` | OWASP security audit session |
| `architecture-review` | Architectural assessment session |

---

## How to Use

### Option A — Drop into a project

Copy the `.github/` directory into your repository root:

```bash
cp -r .github/ /path/to/your-project/.github/
```

Copilot will automatically pick up instruction files based on the `applyTo` glob patterns. Agents and prompts become available in Copilot Chat immediately.

### Option B — Use as a standalone Copilot workspace

Open this repository directly in VS Code with the GitHub Copilot Chat extension. All agents, prompts, and instructions are available in the context of this workspace.

### Option C — Reference specific files

Copy only the instruction or agent files relevant to your stack. Each file is self-contained.

---

## Adding Enterprise-Specific Instructions

Open `.github/instructions/enterprise-custom.instructions.md` and fill in the `# TODO: [ENTERPRISE]` sections:

- Company coding standards and approved libraries
- Internal security policies (data classification, cipher suites, SIEM)
- Approved cloud and infrastructure patterns
- Internal service mesh / API gateway conventions
- Compliance requirements (PCI-DSS, HIPAA, SOC2, GDPR)
- CI/CD quality gates
- Approved observability stack (Splunk, Datadog, Dynatrace, etc.)
- On-call and incident runbook conventions

Remove sections that do not apply. Keep the file concise — Copilot loads it on every interaction.

---

## Using the Diagram Prompts

**Sequence diagram:**
```
/sequence-diagram Payment checkout flow from cart to order confirmation
```

**Data flow diagram:**
```
/dataflow-diagram Order ingestion pipeline from API gateway to fulfilment service
```

Both prompts produce a Mermaid diagram ready to paste into a README, Confluence page, or GitHub wiki. Mermaid renders natively in GitHub Markdown.

---

## Licence

Internal use — replace this section with your organisation's licence.
