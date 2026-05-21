---
name: postgres-database-inspector
description: Safely inspect PostgreSQL databases using psql and existing user authentication via .pgpass.
license: Proprietary
---

# PostgreSQL Database Inspector

## Purpose

Use PostgreSQL command-line tooling (`psql`) to inspect database structure, schema, data, migrations, and query results.

Authentication should use the existing user environment and `.pgpass`.

This skill is intended primarily for:
- schema inspection
- Flyway troubleshooting
- jOOQ-related inspection
- query validation
- debugging backend behavior
- investigating test/dev databases

## Core Principles

- Prefer read-only operations.
- Prefer inspection over modification.
- Avoid dangerous queries by default.
- Keep resultsets small and focused.
- Never dump huge tables unnecessarily.
- Use existing user authentication.
- Treat production environments carefully.

## Safety Rules

Unless explicitly requested:
- do not run DELETE
- do not run UPDATE
- do not run TRUNCATE
- do not run DROP
- do not run ALTER
- do not run INSERT

Prefer:
- SELECT
- EXPLAIN
- information_schema inspection
- pg_catalog inspection

Always clearly state:
- target database
- target environment
- executed query

## Authentication

Authentication should use:
- `.pgpass`
- existing local PostgreSQL tooling
- existing VPN/network access

Do not request passwords when `.pgpass` is configured.

## Connection Pattern

Preferred command:

```bash
PGSSLMODE=require psql \
  -h <host> \
  -p 5432 \
  -U <user> \
  -d <database>
```

Azure PostgreSQL requires SSL. Pass it via the `PGSSLMODE` environment variable — **not** as a positional argument to `psql` (which silently ignores it with a warning).

## Project Environment → Username Map

For the Corporate Lending project (`/Users/ext59986/projects/`), use these usernames when connecting. Passwords are stored in `~/.pgpass` with wildcard host matching (`*:*:*`).

| Environment | Host | Username |
|-------------|------|----------|
| devb | `corp-le-dev-gwc-psql.postgres.database.azure.com` | `CZE_T1_AzureT_POSTGRES_CORP_LE_KDEV_Supports` |
| int | `corp-le-dev-gwc-psql.postgres.database.azure.com` | `CZE_T1_AzureT_POSTGRES_CORP_LE_INT_Supports` |
| prs | `corp-le-dev-gwc-psql.postgres.database.azure.com` | `CZE_T1_AzureT_POSTGRES_CORP_LE_PRS_Supports` |

> Note: The devb application connects as `lending_devb_app` (password via `DEVB_DB_PASSWORD` env var), but for direct `psql` inspection the Azure AD support user above should be used.

## Output Formatting

Prefer:

```bash
psql -P pager=off
```

to avoid pager interaction.

For structured row output:

```bash
psql -x
```

Useful combined form:

```bash
psql -P pager=off -x
```

## Result Limiting

Always limit exploratory queries.

Preferred patterns:

```sql
LIMIT 20
```

Avoid:
```sql
SELECT * FROM huge_table;
```

## Schema Inspection Examples

List schemas:

```sql
SELECT schema_name
FROM information_schema.schemata;
```

List tables:

```sql
SELECT table_schema, table_name
FROM information_schema.tables
WHERE table_type = 'BASE TABLE'
ORDER BY table_schema, table_name;
```

List columns:

```sql
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = 'shared_limit';
```

Inspect enums:

```sql
SELECT t.typname, e.enumlabel
FROM pg_type t
JOIN pg_enum e ON t.oid = e.enumtypid
ORDER BY t.typname, e.enumsortorder;
```

## Flyway Inspection

Useful query:

```sql
SELECT *
FROM flyway_schema_history
ORDER BY installed_rank DESC
LIMIT 20;
```

## jOOQ / Generated Schema Investigation

Useful for:
- enum inspection
- constraint inspection
- index inspection
- generated-table validation

Prefer metadata queries over raw table dumps.

## Explain Plans

When query performance matters:

```sql
EXPLAIN ANALYZE
SELECT ...
```

Be careful on large/shared environments.

## Script Usage

For larger investigations, prefer temporary `.sql` scripts:

```bash
psql ... -f inspect.sql
```

instead of giant inline commands.

## Environment Awareness

Always distinguish:
- local DB
- dev DB
- integration DB
- production DB

Be especially careful on shared environments.

## Output Expectations

After running queries, report:
- environment/database used
- executed query
- key findings
- suspicious results
- possible next queries

Summarize rather than dumping massive raw output.

## When Not To Use

Do not use this skill for:
- bulk destructive changes
- production data manipulation
- uncontrolled schema modifications
- migration execution without explicit approval
