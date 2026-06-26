---
name: backend-test-strategy
description: Use when planning backend tests for API, service, auth, or business logic changes — "what tests does this endpoint need", "plan tests for this service change". Produces unit/integration/API/authz/error-path cases, fixtures and mocks, and repo-real commands. Not for general test planning (use test-plan) or running verification (use verify-change).
argument-hint: [change to test]
---

# Backend Test Strategy

Plan backend tests for: $ARGUMENTS

**Use when:**
- An API endpoint, service, handler, or business rule changed and needs a test plan before or alongside implementation.
- Auth/permission logic or error handling changed and the failure paths need explicit coverage.

**Not for:**
- General test planning across frontend/backend/E2E → `test-plan`.
- Actually running tests and checks on a finished change → `verify-change`.

## Workflow

1. Detect the backend stack and test conventions from real files first — `go.mod` + `*_test.go`, `package.json` test scripts + jest/vitest config, `Makefile` test targets, CI workflows, testcontainers/docker-compose test services. Never assume.
2. Read existing tests nearest the changed code: framework, fixture patterns, mock style, test DB setup. Match them — do not introduce a new testing approach.
3. Read the changed code itself: entry points, dependencies it calls, and the contracts it must keep.
4. Plan unit tests for the business logic — one behavior per case (table-driven if that is the repo's Go style); include boundary inputs.
5. Plan integration tests where the code crosses a boundary (DB, queue, external service) — using the repo's existing test DB/container setup, not a new one.
6. Plan API tests for changed endpoints: status codes, request validation, response shape, and compatibility with existing clients.
7. Plan auth/permission tests for every protected path: unauthenticated, expired/invalid token, wrong role or tenant, ownership bypass via someone else's resource ID.
8. Plan error-path tests: dependency failures, timeouts, malformed input, duplicate/conflict cases — never happy paths only.
9. If the change writes to the DB, plan transaction-behavior tests: rollback on mid-operation failure, no partial writes, concurrent-write conflicts (MongoDB: multi-document transactions and unique-index races).
10. Decide fixtures/mocks: mock external services at the boundary, use real in-repo logic; reuse existing fixtures/factories/builders before inventing new ones.
11. List the exact commands from the repo's package.json scripts, Makefile, or CI workflow that run these tests, smallest scope first. If none exist, say so — do not invent commands.

## Safety

- Planning only — no edits, no test runs, no state-changing commands.
- Never invent repo commands; a generic ecosystem fallback (e.g. `go test ./...`) is allowed only when the repo shows that stack and has no script, marked explicitly as a fallback.
- Never plan tests against shared or production databases — only the repo's documented test DB setup.
- If the test conventions are unclear or conflicting, record an open question instead of guessing a framework.

## Output format

## Backend test strategy
The change restated, the layers it touches (unit/integration/API), and the approach in 2–3 sentences.

## Test cases
Per file or function: happy path, validation, authz, error paths, DB/transaction cases — each concrete and independently checkable.

## Fixtures/mocks
What to mock (external boundaries) vs use real (in-repo logic), and which existing fixtures/factories to reuse.

## Commands
The repo's real test commands (scripts, Makefile, CI), smallest scope first; fallbacks clearly marked.

## Coverage gaps
Behavior left untested by this plan and why, with what is needed to close each gap.

## Reporting

- Separate verified facts (conventions read from real test files and configs) from assumptions (inferred patterns) — flag assumptions for confirmation.
- If test setup could not be inspected (missing services, unreadable CI), list it under **Coverage gaps** with the exact information needed to confirm.
