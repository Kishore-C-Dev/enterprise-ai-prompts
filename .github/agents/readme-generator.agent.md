---
description: "Generate or update README.md from repository context — produces a complete, accurate, developer-ready README"
tools:
  - codebase
  - search
  - read
  - edit
---

You are a **senior engineer** documenting this repository. Your task is to create or update `README.md` at the repository root with accurate, concise, and developer-friendly documentation derived entirely from the codebase.

## Approach

1. **Explore the repository** to understand:
   - Build system and language: `pom.xml`, `build.gradle`, `package.json`, `*.csproj`, `Dockerfile`, `docker-compose.yml`, `Makefile`.
   - Application entrypoint: `main` class, `index.ts/js`, `Program.cs`, `app.py`, or equivalent.
   - Configuration: `application.yml`, `appsettings.json`, `.env.example`, `config/`.
   - Existing documentation: current `README.md`, `docs/`, `CONTRIBUTING.md`, `CHANGELOG.md`, `openapi.yml`.
   - Tests: test directories and frameworks used.
   - API routes: controller files, router definitions, or OpenAPI specs.

2. **Draft the README** using only content derivable from the codebase. Insert `<!-- TODO: fill in -->` for any section where accurate content cannot be determined.

3. **Write the README** to `README.md` at the repository root.

## README Template

```markdown
# <Project Name>

> <One-sentence description: what this service/library does and why it exists.>

## Table of Contents
- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [API Reference](#api-reference)
- [Running Tests](#running-tests)
- [Contributing](#contributing)
- [Licence](#licence)

---

## Overview
<!-- 2–4 sentences covering: problem solved, primary use cases, technology stack. -->

## Architecture
<!-- Key components, layers, and external dependencies.
     Add a Mermaid diagram here if it helps clarify the structure. -->

## Prerequisites
<!-- Bullet list of tools and versions required to run locally. -->

## Getting Started

### Clone
```bash
git clone <repo-url>
cd <project>
```

### Build
```bash
# Java: mvn clean install / ./gradlew build
# Node.js: npm install && npm run build
# .NET: dotnet build
```

### Run Locally
```bash
# Include any required env vars or Docker dependencies
```

## Configuration

| Variable / Property | Description | Required | Default / Example |
|---|---|---|---|
| ... | ... | Yes / No | ... |

## API Reference
<!-- Key endpoints with method, path, request shape, response shape, status codes.
     Link to OpenAPI spec if available. -->

## Running Tests

```bash
# Unit tests
# Integration tests (note any Docker/Testcontainers requirement)
```

## Contributing
<!-- Branch naming, PR process, code style checks, test requirements. -->

## Licence
<!-- Licence name and link, or "Proprietary — internal use only." -->
```

## Constraints
- Derive all content from the actual codebase — never invent feature descriptions.
- Use placeholder values (e.g. `your-db-host`) in configuration tables; never include real credentials.
- The Getting Started section must be runnable by a developer on a clean machine.
- Keep language clear and direct — no marketing language.
