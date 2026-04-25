---
mode: ask
tools:
  - read
  - search
description: "Generate a Mermaid flowchart or C4-style diagram showing data flows, system boundaries, and component dependencies"
argument-hint: "Describe the system, module, or data flow to diagram (e.g. 'payment processing pipeline', 'user authentication flow', 'order ingestion to fulfilment')"
---

# Generate Data Flow Diagram

You are a senior architect. Your task is to produce a **Mermaid `flowchart TD`** that clearly illustrates data flows, system boundaries, component relationships, and the transformation of data as it moves through the system.

## Steps

1. **Identify the scope.** If the user named a feature, module, or data pipeline — search the codebase to locate the relevant files. Identify the data entry points, processing steps, storage layers, and exit points.

2. **Identify components.** Categorise each component:
   - **External systems** — clients, third-party APIs, external databases, partner services.
   - **API / entry points** — REST controllers, GraphQL resolvers, message consumers, schedulers.
   - **Processing / services** — business logic, transformers, validators, orchestrators.
   - **Storage** — relational DB, NoSQL, object storage, cache, message queue/topic.
   - **Output / integrations** — outbound calls, published events, notifications, reports.

3. **Map data flows.** For each edge:
   - What data moves between components (type, format, protocol — e.g. `HTTP/JSON`, `Kafka message`, `SQL query`)?
   - Is the flow synchronous or asynchronous?
   - Are there transformation or enrichment steps?

4. **Generate the diagram.** Use Mermaid `flowchart TD` syntax:
   - Use `subgraph` blocks to show service or system boundaries.
   - Label edges with the data type or protocol where meaningful.
   - Use different node shapes to distinguish component types:
     - `[Rectangle]` for services / processors
     - `[(Cylinder)]` for databases / storage
     - `([Rounded])` for external systems / clients
     - `{Diamond}` for decision points
   - Use `:::classDef` styles if you need to colour-code component categories.

5. **Wrap the diagram** in a fenced code block:

   ````markdown
   ```mermaid
   flowchart TD
       ...
   ```
   ````

6. **Add a brief narrative** (3–5 bullet points) describing:
   - The primary data path (happy flow).
   - Key transformation or enrichment steps.
   - Error paths or dead-letter handling.
   - Any compliance or security boundaries worth highlighting (e.g. PII leaves the trust boundary here).

## Constraints

- Keep node labels short and recognisable; avoid internal class or method names.
- Group components into `subgraph` blocks by service or bounded context.
- Show data formats / protocols on edges only where they add meaningful context.
- If the scope is very large, diagram the top-level flow and note which subsystems can be expanded separately.
