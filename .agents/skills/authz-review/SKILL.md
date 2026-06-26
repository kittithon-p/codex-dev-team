---
name: authz-review
description: Use when authentication or authorization needs review — "is this endpoint protected", "can users see each other's data", or any diff touching middleware, guards, roles, sessions, or ownership checks. Maps protected resources to required permissions, verifies where checks happen, and hunts bypass paths (IDOR, mass assignment, route-level gaps). Not for a full security pass (use security-audit) or design-stage threat analysis (use threat-model).
---

# Authz Review

Review authentication and authorization for the current diff and its surrounding routes — who can reach what, and which checks actually enforce it.

**Use when:**
- A diff adds or changes routes, handlers, middleware, guards, roles, or ownership checks.
- Someone asks "is this endpoint protected" or "can user A access user B's data".

**Not for:**
- Full security review (injection, secrets, dependencies, headers) → `security-audit`.
- Design-stage asset/attacker modeling before code exists → `threat-model`.

## Workflow

1. Find the repo's real auth pattern first and read it before judging anything — Next.js: `middleware.ts`, auth helpers (`getServerSession`/`auth()`), server actions; Go: router middleware chains, handler guards; elsewhere: decorators, route groups, gateway config.
2. Scope the review from `git diff` (or the files in context): inventory the protected resources it touches — endpoints, server actions, mutations, background jobs, admin pages, and the data they expose.
3. For each resource, state the required permission as the code implements it: authenticated only? specific role? ownership of the object? tenant match?
4. Locate where each check actually happens (file:line) — middleware, handler, service layer, or query filter — and confirm the resource passes through it. The dangerous gaps are routes that skip the shared pattern.
5. Hunt bypass paths: direct object reference (ID from URL/body used without an ownership check — in MongoDB, queries missing a `userId`/tenant filter), mass assignment (request body bound straight into update/insert, including role fields), route-level gaps (new route outside the guarded group, unguarded HTTP method, public sibling endpoint), privilege escalation (user-controlled role/flag changes).
6. Any fix you propose must follow the repo's existing auth pattern — never a parallel auth path.
7. Define the authz tests needed per role and resource: allowed and denied cases, plus cross-user/cross-tenant attempts. Use the repo's real test commands (package.json scripts, Makefile, `go test`); if none exist, say so.

## Safety

- Review only — no edits, no state-changing commands.
- Never propose a second auth mechanism alongside the existing one; extend what is there.
- Do not print tokens, session secrets, or credentials encountered while reading.
- If a check cannot be traced (generated routes, external gateway), report it as unverified — never assume a middleware covers a route.

## Output format

## Protected resources
Endpoints/actions/data touched by the change that require auth.

## Required permissions
Role, permission, or ownership rule per resource, as the code implements it.

## Current checks
Where each check happens (file:line): middleware, handler, service, query filter.

## Missing/bypass risks
Resources reachable without the expected check — IDOR, mass assignment, route-level gaps, escalation — each with file:line and severity.

## Tests needed
Authz cases to add: per-role allowed/denied, cross-user and cross-tenant attempts.

## Reporting

- Separate verified findings (traced in code, with file:line) from suspicions (pattern suggests risk but unconfirmed).
- List every auth path that could not be inspected, with what is needed to confirm it.
