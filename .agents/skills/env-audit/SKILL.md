---
name: env-audit
description: Use when environment variable usage needs review — "audit env vars", "are all env vars documented", or a diff adds, renames, or reads variables. Compares real code usage against .env.example, docs, and CI/deploy config, and flags client-side exposure and missing-default failure behavior. Not for secret exposure scanning (use secrets-check) or deploy readiness (use deploy-check).
---

# Env Audit

Audit environment variable usage in the current diff and its surrounding config — variable names and read locations only, never values.

**Use when:**
- A change adds, renames, or reads environment variables and needs an audit before merge.
- `.env.example`, setup docs, or CI/deploy config may have drifted from what the code actually reads.

**Not for:**
- Hunting leaked secrets in code, logs, or git history → `secrets-check`.
- Judging whether a change is safe to deploy → `deploy-check`.

## Workflow

1. Detect the stack from real files first — `package.json`, `next.config.*`, `go.mod`, `Dockerfile`, `docker-compose.*`, CI workflows, `Makefile`. The env loading pattern follows the stack.
2. Find every variable the changed code reads: `process.env.X` (Node/Next.js), `os.Getenv` / `viper` / config loaders (Go), `${VAR}` in Dockerfile, compose, and CI files. Start from `git diff`, then trace the touched config modules.
3. Compare against `.env.example` (or `.env.sample`, `env.template`): every variable read in code must appear there with a placeholder — never a real value.
4. Check docs coverage: README/setup docs must list each required variable and what it does.
5. Check CI/deploy presence: is each variable defined or referenced in CI workflows, Dockerfile/compose, or platform config — or will its absence only surface at runtime?
6. Assess client-side exposure: anything `NEXT_PUBLIC_`-prefixed or imported into client components is shipped in the browser bundle — confirm it is safe to be public, and flag server-only secrets given the prefix by mistake.
7. Check defaults and failure behavior per variable when unset: hard fail at startup (good), silent fallback to a wrong value (flag), or crash at first use deep in a request path (flag).

## Safety

- Read-only review — no edits, no state-changing commands.
- Report variable NAMES and read locations only; never print, echo, or copy values from `.env`, shell, or CI secrets.
- Do not quote contents of local `.env` files; at most confirm which names they define.
- Use the repo's real scripts first; otherwise only generic commands (`git diff`, grep). Never invent repo-specific commands.

## Output format

## Env audit
One row per variable found in the audited scope:

| Variable | Used where | Required? | Example/docs status | Risk |
|----------|------------|-----------|---------------------|------|

## Actions needed
Concrete fixes ordered by severity — missing `.env.example` entries, doc gaps, CI/deploy omissions, exposure or failure-behavior problems — each with file and suggested change.

## Reporting

- Separate verified facts (reads found in real files/diff) from assumptions (inferred loaders, platform config you could not see).
- If deploy config lives outside the repo (platform-managed), say so explicitly and list which variables could not be verified there.
