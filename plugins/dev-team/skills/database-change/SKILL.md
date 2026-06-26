---
name: database-change
description: Use when planning or implementing a schema, migration, index, or data change — "add a field to users", "write a migration for X", "backfill this column safely". Covers schema impact, a migration plan using the repo's real tooling, backfill, rollback, indexes, transactions, and locking. Not for reviewing an existing migration (use migration-review) or query perf (use query-performance-check).
argument-hint: [change description]
---

# Database Change

Plan or implement this database change safely: $ARGUMENTS

**Use when:**
- A schema, collection, index, constraint, or backfill change needs a safe rollout path.
- A feature requires new fields, tables/collections, or reshaping existing data.

**Not for:**
- Reviewing a migration someone already wrote → `migration-review`.
- Diagnosing or tuning slow queries → `query-performance-check`.

## Workflow

1. Detect the database and migration tooling from real files first — migration folders, ORM/ODM config (`prisma/`, `migrations/`, GORM/sqlc setup, Mongoose/Mongo models, raw SQL files), `package.json` scripts, `Makefile` targets, CI workflows. Never assume; never invent migration commands.
2. Read the current schema/models and every query path touching the affected tables or collections before changing anything.
3. State the schema impact: tables/collections, fields, types, indexes, constraints — before → after.
4. Write the migration plan with the repo's real migration tool as ordered, individually applyable steps. For MongoDB (no enforced schema), plan code-level compatibility: readers must handle old and new document shapes during rollout.
5. Plan the backfill for existing rows/documents: defaults, batch size, idempotency, expected volume, and whether it runs inside or outside the migration.
6. Define rollback per step: a reverse migration, or an explicit "irreversible because X" with mitigation (backup, expand–contract).
7. Cover indexes: add ones the new query paths need; flag ones the change orphans. Build large indexes concurrently/in background where the engine supports it.
8. Check transactions and locking: which steps are transactional, which DDL locks tables and for how long; prefer expand–contract over in-place rewrites on large or hot tables.
9. Verify integrity constraints: FKs, uniqueness, NOT NULL on backfilled fields (add the constraint only after the backfill completes), application-level validation for schemaless stores.
10. Add or update tests with the repo's real test commands: migration up/down where the tool supports it, data integrity assertions, and the queries that read the new shape.

## Safety

- Never run destructive commands (drop, truncate, mass update/delete) against real data — local or disposable databases only; production-shaped operations stay as plans for the user to run.
- Implementation edits only the files assigned to this task; nothing else.
- Use only migration/test commands that exist in the repo; if none exist, say so instead of inventing them.
- Destructive or irreversible migrations need explicit user approval before they ship.
- Mind deploy order: code that depends on the migration must not deploy before it, and vice versa.

## Output format

## Database change plan
The change restated in 1–2 sentences, the chosen approach (in-place vs expand–contract), and the detected database + migration tool.

## Migration impact
Ordered migration steps with the repo's real tool, plus locking/duration risk per step.

## Data impact
Existing rows/documents affected: backfill strategy, defaults, volume, idempotency.

## Rollback notes
Reverse path per step, or "irreversible because X" with mitigation.

## Query/index concerns
Indexes added/removed/orphaned and query paths affected by the new shape.

## Tests
Exact repo commands run (or to run) covering migration, integrity, and affected queries.

## Reporting

- Separate verified facts (read from schema/migrations) from assumptions (inferred shapes, volumes).
- If data volume or production topology is unknown, flag it as an open question instead of guessing lock duration or backfill time.
- Report any check that could not run (missing DB, env, services) with the exact command for the user.
