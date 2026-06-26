---
name: api-docs
description: Use when API endpoints or schemas changed and the API documentation needs updating or review — "update the API docs", "document this endpoint". Covers finding the docs source of truth (OpenAPI spec, docs/, README), changed endpoints/schemas, request/response examples, and breaking-change notes. Not for general docs (use update-docs) or compatibility review (use api-contract-check).
---

# API Docs

Update or review API documentation so it matches the endpoints and schemas as actually implemented in the current diff.

**Use when:**
- The current diff adds, changes, or removes endpoints, request/response shapes, status codes, or auth requirements.
- API docs drifted from the code and need a review against the actual handlers.

**Not for:**
- README, setup, env vars, or other non-API documentation → `update-docs`.
- Judging whether an API change is breaking/compatible for clients → `api-contract-check` (this skill documents the change; that one reviews it).

## Workflow

1. Find the repo's docs source of truth before writing anything: OpenAPI/Swagger spec (`openapi.yaml`, `swagger.json`), a `docs/` folder, a README API section, GraphQL schema, `.proto` files, or code annotations (Go `swag` comments, JSDoc). Update that source — never create a parallel doc.
2. List the endpoints and schemas changed in the current diff (`git diff` on routes, handlers, DTOs — e.g. Go handlers, Next.js `app/api` routes, controllers): method, path, request/response shape, status codes, auth — before → after.
3. Verify documented behavior against the actual handler code, not the commit message. Document only implemented behavior.
4. Identify which request/response examples are needed: full examples for new endpoints, updated examples for changed shapes. Realistic but fake data only — no real tokens, IDs, or PII.
5. Write breaking-change notes where consumers must act: removed/renamed fields, tightened validation, changed status codes, deprecations, versioning.
6. List the exact docs files to touch, then apply the edits (or report them, in review mode).
7. If docs are generated from annotations (e.g. `swag init`, an npm docs script), edit the annotations and run the repo's real generation script — never hand-edit generated output.

## Safety

- Edit only the identified docs files and annotations — never handlers, routes, or other source code.
- No secrets, real tokens, internal URLs, or PII in examples.
- Do not document unimplemented or unverified behavior; if a handler could not be inspected, flag it instead of guessing.
- Use the repo's real docs-generation scripts (package.json, Makefile); never invent commands.

## Output format

## API docs review
One-paragraph summary: what changed in the API and which docs source of truth was found.

## Endpoints/schemas changed
Each changed endpoint/schema: method, path, shape before → after, status codes, auth.

## Examples needed
Request/response examples added or still missing, with fake-but-realistic data.

## Breaking notes
What consumers must change; deprecation and versioning notes. "None" if fully additive.

## Docs files
Exact files touched (or to touch): spec, docs/ pages, README section, annotation files.

## Reporting

- Separate behavior verified by reading handlers from behavior inferred from the diff alone.
- If the docs source of truth is ambiguous (multiple candidates) or a handler could not be inspected, say so under **API docs review** and ask before writing.
