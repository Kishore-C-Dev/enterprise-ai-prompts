---
applyTo: "**/*.{java,ts,js,cs,sql,xml,yml,yaml,json,properties}"
description: "Use when writing or reviewing database access code, queries, migrations, or ORM usage for Java, Node.js, and .NET"
---

# Database Instructions

## All Stacks

- Use parameterised queries or ORM query builders — never concatenate user input into SQL strings.
- Keep transaction scopes as small and short-lived as practical; commit or roll back promptly.
- Do not make remote HTTP or messaging calls inside a database transaction unless explicitly required and documented.
- Avoid unbounded queries: always apply `LIMIT`/`TOP`/`Take` and pagination for list endpoints; never return all rows of a large table.
- Define indexes for columns used in frequent `WHERE`, `JOIN`, and `ORDER BY` clauses; review execution plans for new queries on large tables.
- Keep schema migrations backward compatible when possible: add columns as nullable or with defaults; rename in two phases (add → backfill → remove old).
- Avoid destructive migrations (column drops, table renames, type changes) without a rollback plan and a maintenance window. Document the rollback steps in the migration file header or PR description.
- Do not expose raw database error messages, constraint names, or SQL details to API consumers.
- Handle concurrent write conflicts: use optimistic locking (version columns / ETags) or pessimistic locking where appropriate; document the choice.
- Close all connections and cursors promptly; use connection pooling; configure pool size, timeout, and overflow settings explicitly.

## Java

- Use lazy loading only within an open Hibernate/JPA session; be aware of the open-session-in-view anti-pattern — disable `spring.jpa.open-in-view=true` unless intentional.
- Prefer `@Query` with JPQL or native SQL parameters over Criteria API for complex queries; avoid building JPQL strings with string concatenation.
- Use Flyway or Liquibase for all schema migrations; never run DDL manually in production.
- Apply `@Version` for optimistic locking on entities subject to concurrent updates.
- Use `@Transactional(readOnly = true)` for read-only service methods to hint the persistence provider and connection pool.
- Avoid calling `findAll()` on repositories without pagination for tables that can grow unboundedly.

## Node.js

- Use Prisma, TypeORM, or Sequelize parameterised query APIs — never pass user input into `query()` raw string methods.
- Configure `connectionLimit`, `acquireTimeout`, and `idleTimeoutMillis` in your connection pool.
- Keep Prisma schema migrations in version control (`prisma/migrations/`); run `prisma migrate deploy` in CI/CD, not `prisma migrate dev`.
- Use database transactions (`prisma.$transaction`, TypeORM `QueryRunner`, Sequelize `transaction`) for multi-step writes.
- Avoid N+1 queries: use `include`/`eager loading` or batched queries; never query inside a loop over a result set.
- Validate migration scripts in a staging environment before applying to production.

## .NET

- Use `AsNoTracking()` for all read-only EF Core queries to avoid unnecessary change tracking overhead.
- Use `IDbContextFactory<T>` in long-lived services (background workers, singletons) to scope `DbContext` lifetimes correctly.
- Always pass a `CancellationToken` to EF Core async methods (`ToListAsync`, `SaveChangesAsync`, etc.).
- Use EF Core migrations (`dotnet ef migrations add`) for all schema changes; keep migration files in source control.
- Avoid `Include()` chains that load unbounded collections; use projection (`Select`) or split queries for large graphs.
- Use `ExecuteSqlRawAsync` only with parameterised placeholders; never with string interpolation directly from user input.
- Configure connection resilience with `EnableRetryOnFailure` for transient database errors; set appropriate retry limits.
