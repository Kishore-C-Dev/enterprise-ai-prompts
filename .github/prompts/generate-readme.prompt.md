---
mode: agent
tools:
  - read
  - search
  - edit
description: "Scaffold or regenerate a README.md from the repository codebase — covers overview, architecture, setup, configuration, API, testing, and contributing"
argument-hint: "Optional: list specific sections to generate or regenerate, e.g. 'Getting Started and Configuration only'"
---

# Generate README

You are a senior engineer documenting this repository. Your task is to **create or update `README.md`** at the repo root with accurate, concise, and developer-friendly documentation derived directly from the codebase.

## Steps

1. **Explore the repository.** Read and search:
   - Root files: `pom.xml`, `build.gradle`, `package.json`, `*.csproj`, `Dockerfile`, `docker-compose.yml`, `.env.example`, `Makefile`.
   - Entry point: `main` class, `index.ts/js`, `Program.cs`, or equivalent.
   - Configuration files: `application.yml`, `application.properties`, `.env.example`, `appsettings.json`.
   - Existing docs: any existing `README.md`, `docs/`, `CONTRIBUTING.md`, `CHANGELOG.md`.
   - Test directories to understand test coverage and frameworks in use.

2. **Generate the README** with these sections (omit sections that genuinely do not apply):

---

## README Structure

```markdown
# <Project Name>

> One-sentence description of what this service/library does and why it exists.

## Table of Contents
- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Configuration](#configuration)
- [API Reference](#api-reference)   <!-- omit for non-API projects -->
- [Running Tests](#running-tests)
- [Contributing](#contributing)
- [Licence](#licence)

## Overview
<!-- 2–4 sentences: problem solved, primary use cases, technology stack. -->

## Architecture
<!-- Brief description of key components, layers, and external dependencies.
     Include a Mermaid diagram if helpful (use the dataflow-diagram or sequence-diagram prompts). -->

## Prerequisites
<!-- Bullet list of tools, versions, and accounts needed to run this project locally. -->

## Getting Started
<!-- Step-by-step instructions to clone, configure, build, and run the project locally. -->

## Configuration
<!-- Table of all environment variables / config properties, with description and example values.
     Mark required vs optional. Never include real secrets — use placeholder values. -->

| Variable | Description | Required | Example |
|---|---|---|---|
| ... | ... | Yes/No | ... |

## API Reference
<!-- For REST APIs: list key endpoints with method, path, request shape, response shape, and status codes.
     For libraries: describe the public API surface. -->

## Running Tests
<!-- Commands to run unit tests, integration tests, and any test prerequisites (e.g. Docker). -->

## Contributing
<!-- How to raise issues, branch naming, PR process, and any code style / linting requirements. -->

## Licence
<!-- Licence name and link, or "Proprietary — internal use only." -->
```

---

## Constraints

- Derive all content from the actual codebase — do not invent feature descriptions.
- Use placeholder values (e.g. `your-db-host`, `your-api-key`) in configuration tables; never use real credentials.
- Keep the Getting Started section runnable: a new developer should be able to follow it on a clean machine.
- If a section cannot be accurately populated from the codebase, insert a `<!-- TODO: fill in -->` comment rather than writing inaccurate content.
- Write in clear, direct English — no marketing language or excessive adjectives.
