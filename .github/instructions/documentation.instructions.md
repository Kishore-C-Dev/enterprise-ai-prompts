---
applyTo: "README.md,docs/**/*.md,**/*.md,openapi*.yaml,openapi*.yml"
description: "Documentation standards — accuracy, coverage of behavior changes, examples, migration notes, and operational guidance"
---

# Documentation Instructions

## Accuracy and Sync

- Documentation must stay accurate with the code. A PR that changes behavior, configuration, or setup without updating the relevant docs is incomplete.
- Outdated documentation actively harms teams — it causes incorrect deployments, wasted debugging time, and onboarding failures.
- If you cannot update the docs as part of the same PR, create a follow-up ticket and reference it in the PR description.

## What to Update

When a code change affects any of the following, update the corresponding documentation:

| What changed | Docs to update |
|---|---|
| New or changed API endpoint | OpenAPI spec + API reference section of README or `docs/api.md` |
| New or changed configuration | Configuration reference (`docs/configuration.md` or README config section) |
| Changed setup or run instructions | README "Getting Started" / "Running Locally" section |
| New dependency or tooling requirement | README prerequisites section |
| Changed deployment procedure | Deployment guide (`docs/deployment.md` or equivalent) |
| Changed error behavior or new error types | Troubleshooting guide + API error reference |
| Breaking change | Migration guide with step-by-step instructions (see below) |
| New operational metric, alert, or log pattern | Runbook and on-call guide |
| Changed environment variable or secret requirement | Configuration reference |

## Breaking Changes

- Document breaking changes prominently — a `## Breaking Changes` section in the relevant migration doc or PR description.
- Include: what changed, why it changed, what consumers need to do, and by when (deprecation / removal timeline).
- Example format:

```markdown
## Breaking Change: Order status field renamed

**Affected:** `/api/v1/orders` response body
**Change:** `status` field renamed to `orderStatus` to align with platform standards.
**Migration:** Update all consumers to read `orderStatus` instead of `status` before the v1 endpoint is retired on 2025-06-01.
**v2 endpoint available at:** `/api/v2/orders`
```

## Examples and Commands

- Prefer concrete examples over abstract descriptions — a working `curl` command teaches faster than two paragraphs.
- Include examples for: API request/response bodies, configuration values, CLI commands, environment variable setup.
- Keep examples runnable — test them against a real or local environment before merging.

## Conciseness and Actionability

- Keep documentation concise. If a paragraph can be replaced with a command, replace it.
- Use numbered steps for procedures; use bullet lists for reference material.
- Avoid stale screenshots — they become misleading as UIs change. Use text-based instructions or Mermaid diagrams for flows.
- Do not include environment-specific assumptions intended for only your team (internal Slack channels, internal tool names) in docs shared across teams or open-sourced.

## Operational Notes

- When behavior visible to support or on-call teams changes, include a note for them:
  - New or changed error messages (so they know what to search for in logs).
  - New or changed metrics or alerts.
  - New runbook entries for new failure modes.
  - Changes to health check behavior.
- Operational notes belong in the PR description and, for persistent changes, in the service's runbook.

## OpenAPI / Swagger

- Update the OpenAPI spec in the same PR as the API change — not in a follow-up.
- Every new endpoint must have: a summary, a description, all request parameters documented, all response schemas defined, and at least one example.
- Mark deprecated fields and endpoints with `deprecated: true` and include a `description` noting the replacement.

## Maintenance

- Remove or update documentation that is no longer accurate when refactoring or deleting features.
- Stale "how to" sections are worse than no documentation — they cause active confusion.
- Flag documentation that is unclear, outdated, or missing as a `Minor` review finding; flag missing operational docs for a significant behavior change as a `Major` finding.
