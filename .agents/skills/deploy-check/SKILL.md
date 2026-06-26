---
name: deploy-check
description: Use when a change needs a deploy/build readiness review without deploying — "is this safe to deploy", "check deploy readiness", or any diff touching Dockerfile, CI workflows, build scripts, or env config. Covers build impact, env vars, Docker, CI/CD pipeline, health checks, and rollback notes. Not for a production safety verdict (use release-safety-check), rollback detail (use rollback-plan), or env depth (use env-audit).
---

# Deploy Check

Review deploy/build readiness of the current diff — without deploying anything.

**Use when:**
- A change is about to merge and you need to know whether it builds and deploys cleanly.
- The diff touches Dockerfile, compose files, CI workflows, build scripts, or env examples.

**Not for:**
- A go/no-go production safety verdict → `release-safety-check`.
- A detailed rollback procedure → `rollback-plan`.
- Deep env var coverage across environments → `env-audit`.

## Workflow

1. Inspect the repo's deploy surface first — `Dockerfile`, `docker-compose.*`, CI workflows (`.github/workflows/*`), `Makefile`, `package.json` scripts, `go.mod`, env examples, health-check endpoints. Never assume the stack; detect it from these files.
2. Read the current diff (`git status`, `git diff`) and map each changed file to a deploy concern: build, runtime config, pipeline, data.
3. Check build impact: do the repo's real build scripts still cover the change — new deps, lockfile drift, build args, codegen steps, Next.js build output, Go build tags/CGO? Use the repo's own scripts; never invent commands.
4. Check env vars by NAME only: new, renamed, or removed vars — are env examples, Docker, and CI/deploy config in sync? Never read or print values.
5. Check Docker: base image, COPY paths, exposed ports, entrypoint, multi-stage targets affected by the diff.
6. Check the CI/CD pipeline: which workflow steps the change touches, newly required secrets (names only), cache keys, deploy triggers, deploy order vs migrations.
7. Check health checks: does the change affect startup, readiness, or liveness — slow boot work, DB connections at startup (e.g., MongoDB), graceful shutdown?
8. Note rollback concerns: anything that makes rolling back hard — schema migrations, cache shape, queue/message formats, irreversible data writes.
9. Call out production risk explicitly: blast radius if this ships broken, and whether a human approval or gate is required before deploy.

## Safety

- DO NOT deploy under any circumstance — no deploy commands, no `docker push`, no triggering CI, no tagging.
- Read-only review: no edits, no state-changing commands.
- Env vars and secrets by name only — never read, print, or log values.
- Never invent repo-specific commands; reference only scripts that exist in the repo.

## Output format

## Deploy readiness
One-line verdict — ready / ready with notes / not ready — plus the single biggest blocker.

## Build impact
Build steps, dependencies, build args, and artifacts affected by the diff.

## Env vars
New/changed/removed var names and whether examples, Docker, and CI config are in sync.

## Deployment config impact
Dockerfile, compose, CI/CD workflow, and runtime config changes the diff requires.

## Health checks
Effect on startup, readiness, and liveness — or "no impact" with the evidence.

## Rollback concerns
What makes rollback hard (migrations, cache shape, queue formats); note when `rollback-plan` is warranted.

## Manual approvals
Anything that needs a human decision before deploy — production risk callouts, gates, sign-offs.

## Reporting

- Separate verified facts (read from files/diff) from assumptions (inferred).
- If deploy config could not be found (no Dockerfile, no CI workflows), say so explicitly instead of guessing.
- List anything unverifiable as an open question with what is needed to confirm it.
