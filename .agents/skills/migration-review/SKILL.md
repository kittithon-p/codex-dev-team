---
name: migration-review
description: Use when a database migration is written and needs a safety review before it ships — "review this migration", "is this migration safe to deploy". Covers destructive operations, locking on large tables, data integrity, deploy order vs code, rollback, and verification. Not for planning new schema/data changes (use database-change) or overall ship safety (use release-safety-check).
---

# Migration Review

Review the database migration in the current diff/context before it ships.

**Use when:**
- A migration file is in the diff (or named by the user) and needs a pre-ship safety check.
- A schema, index, or data-backfill change is about to deploy and someone asks if it is safe.

**Not for:**
- Planning or designing a new schema/data change → `database-change`.
- Release readiness beyond the migration itself → `release-safety-check`.

## Workflow

1. Inspect the repo first: identify the migration tool and conventions — migration folders, `package.json` scripts, `Makefile` targets, `go.mod` (golang-migrate, goose, atlas), ORM configs (Prisma, Drizzle, TypeORM), or script-based migrations (common with MongoDB). Use `git diff` to isolate the exact migration under review.
2. Read the migration plus the schema/models it touches, and the code paths that query the affected tables/collections.
3. Destructive risk: flag DROP/TRUNCATE, column/field removals, renames, type narrowing or changes, and any data-losing operation — each needs explicit justification or a safer expand/contract alternative.
4. Locking/performance risk: estimate affected table/collection size where possible; flag full-table rewrites, synchronous index builds (require `CONCURRENTLY` or background indexes), long transactions, and unbatched backfills.
5. Data integrity risk: check new constraints against existing data, orphaned references, NOT NULL/defaults on populated tables, and partial-failure behavior — is the migration transactional, idempotent, resumable?
6. Deploy order: determine migration-before-code vs code-before-migration, and exactly what breaks during the rollout window (old code on new schema, new code on old schema).
7. Rollback plan: verify the down path exists and actually restores state; name what is unrecoverable (dropped data) and its mitigation (backup, snapshot, expand/contract).
8. Verification: list concrete pre/post checks — the repo's real migration dry-run/plan commands first, then row/document counts, constraint checks, and a staging run on production-sized data.

## Safety

- Review only — no edits, no running migrations, no state-changing commands.
- Never invent migration commands; use only what the repo's scripts/Makefile/CI show, and say so if none exist.
- Treat any data-losing operation as blocking until the user explicitly accepts the loss.
- If table size or production data shape is unknown, flag it as unknown — never assume "small table".

## Output format

## Migration review
One-paragraph verdict — ship / ship with changes / block — and why.

## Destructive risk
Drops, renames, type changes, data-losing operations; severity plus a safer alternative for anything blocked.

## Locking/performance risk
Locks, index builds, full scans, backfill cost on large tables — with online-migration options.

## Data integrity risk
Constraints vs existing data, orphaned references, partial-failure and idempotency behavior.

## Deploy order
Migration-before-code or code-before-migration, and what breaks if the order is violated.

## Rollback plan
Whether the down path is real and tested; what state is unrecoverable and its mitigation.

## Verification
Exact commands/checks to run before and after, using the repo's real scripts where they exist.

## Reporting

- Separate verified facts (read from migration/schema/code) from assumptions (table size, traffic, data shape).
- If table size, production data, or the down path could not be verified, say so explicitly and state what is needed to confirm.
