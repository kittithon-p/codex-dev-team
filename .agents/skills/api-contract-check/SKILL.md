---
name: api-contract-check
description: Use when API request/response shapes change and compatibility needs review — "did this break the API", "check this endpoint change", or any diff touching routes, handlers, DTOs, OpenAPI/gRPC/GraphQL definitions. Covers breaking-change detection, status/error codes, validation tightening, client impact, and contract tests. Not for general code review (use review-diff) or documentation updates (use api-docs).
---

# API Contract Check

Review API request/response contract compatibility and breaking changes in the current diff.

**Use when:**
- A diff changes request/response shapes, routes, status codes, or validation rules.
- An endpoint, DTO, OpenAPI/Swagger spec, `.proto`, or GraphQL schema changed and clients may break.

**Not for:**
- General correctness/style review of the same diff → `review-diff`.
- Updating API documentation after the contract is settled → `api-docs`.

## Workflow

1. Find the contract's source of truth from real files first — OpenAPI/Swagger specs, `.proto` files, GraphQL schemas, validation schemas (zod/joi, Go validator tags), shared TypeScript types, Go request/response structs and their `json` tags, route handlers, generated clients. Never reason from the diff alone.
2. List every changed endpoint/operation from the diff (`git diff` against the base branch) and map each to its handler, schema, and types.
3. Check request shape changes: new required fields, removed/renamed fields, type or nullability changes, changed defaults, stricter parsing of existing inputs.
4. Check response shape changes: removed/renamed fields, type changes, fields becoming optional/nullable, envelope or pagination structure changes.
5. Check status and error code changes: different success codes, new or removed error codes, changed error body shape that clients parse.
6. Check validation changes: stricter rules on existing inputs (required, enum, length, format) are breaking for current senders; looser rules can break downstream consumers expecting clean data.
7. Give a backward-compatibility verdict per endpoint: additive (safe), tolerant (depends on client strictness), or breaking.
8. Trace frontend/client impact: grep the repo (and sibling frontend/mobile packages in a monorepo) for callers of changed endpoints and consumers of changed types. In TypeScript codebases, the repo's typecheck script (fallback: `npx tsc --noEmit`) surfaces type drift.
9. Recommend contract tests for each breaking or tolerant change: schema/spec diff checks in CI, handler tests asserting response shape and status codes, generated-client regeneration. Use the repo's real test scripts (package.json, Makefile, `go test`) — name them only if they exist.

## Safety

- Review only — no edits, no state-changing commands.
- Read-only commands only: `git diff`, grep, file reads; use the repo's real scripts first before generic fallbacks like `npx tsc --noEmit` or `go test ./...`.
- Never invent endpoints, fields, clients, or commands not visible in the repo.
- If the spec and the implementation disagree, report the mismatch — do not silently trust one side.

## Output format

## Contract changes
Endpoint by endpoint: before → after for request, response, status codes, and validation.

## Breaking risks
Each breaking or tolerant change, with how it fails at runtime for an existing client.

## Affected clients
Frontend, mobile, SDKs, internal services, external consumers — with file:line call sites where found.

## Validation gaps
Inputs that changed without matching validation, or validation that changed without a contract update.

## Tests needed
Concrete contract tests to add or update, mapped to the repo's real test commands.

## Migration notes
Versioning, deprecation windows, client-first vs server-first rollout ordering.

## Reporting

- Separate confirmed breaks (traced to a real call site) from potential breaks (no caller found in this repo — external or mobile clients may still exist).
- If the contract's source of truth could not be located, say so under **Validation gaps** and list where you looked.
