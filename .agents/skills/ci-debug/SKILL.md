---
name: ci-debug
description: Use when a CI pipeline fails or workflow configuration misbehaves — "why did CI fail", "the build is red on main", flaky jobs, wrong tool versions, cache misses, or a step that passes locally but fails in CI. Covers locating the failing job/step, classifying code vs config vs flake, and mapping it to a local reproduction. Not for verifying a change locally (use verify-change) or deploy readiness (use deploy-check).
argument-hint: [failure description]
---

# CI Debug

Investigate this CI failure or workflow problem: $ARGUMENTS

**Use when:**
- A CI job is failing and the cause (code, config, or flake) is unknown.
- Workflow configuration misbehaves — wrong tool version, missing cache, bad path filter, matrix or trigger issue.
- A step passes locally but fails in CI, or vice versa.

**Not for:**
- Verifying a change locally before pushing → `verify-change`.
- Judging whether a build is safe to deploy → `deploy-check`.

## Workflow

1. Inspect the CI config from real files first — `.github/workflows/*`, `.gitlab-ci.yml`, `Jenkinsfile`, `bitbucket-pipelines.yml` — plus anything they call (`Makefile`, `package.json` scripts, `Dockerfile`, composite actions). Never reason about CI you have not read.
2. Identify the exact failing workflow, job, and step from logs when available (`gh run list`, `gh run view <id> --log-failed` on GitHub Actions); if logs are unreachable, ask for the failing step's output instead of guessing.
3. Classify the failure: code (test/lint/build genuinely failing), config (wrong version, missing cache key, bad path, unset env var), or environment flake (timeout, network, runner resources).
4. Map the failing CI step to its equivalent local command — the same script the workflow invokes. Use the repo's real scripts first (package.json scripts, Makefile targets, `go test ./...`); never invent commands. Note tool-version differences between CI and local.
5. Reproduce locally where possible — a step that passes locally with identical inputs points to config or environment, not code.
6. Find the trigger in history: `git log` on the workflow file and the failing area — did the config change, or the code?
7. Propose the minimal fix. Do not change CI blindly — every proposed workflow edit must trace to evidence from logs or config, and must state what to watch on the next run.

## Safety

- Investigation only — no edits. Deliver a proposed fix; apply only on explicit approval.
- Never print, echo, or log CI secrets — no dumping `secrets.*`, env blocks, or masked values, even "just for debugging".
- Read-only commands only (`gh run view`, `git log`, file reads); do not re-run, cancel, or trigger workflows unless asked.
- If logs or workflow files are unavailable, say so and stop at a provisional diagnosis — do not guess the failing step.

## Output format

## CI issue
The exact workflow, job, and step that fails, with the error line quoted.

## Likely cause
Code vs config vs flake, with the evidence — log lines, config lines, or the commit that introduced it.

## Local reproduction
The command(s) to reproduce the failing step locally, taken from the repo's own scripts.

## Proposed fix
The minimal change (file and lines for config issues) and why it addresses the cause.

## Validation
The local check to run first, then what to watch on the next CI run to confirm the fix.

## Reporting

- Separate verified facts (read from workflow files and logs) from assumptions (inferred from symptoms).
- If logs or workflow files could not be inspected, flag the diagnosis as provisional and state exactly what is needed to confirm it.
