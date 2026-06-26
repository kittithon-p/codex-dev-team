---
name: verify
description: Run this workspace's build/vet/format/test verification chain for the repo(s) touched, and report OK/ERR per step. Use before claiming work done or before an MR. No commits. Trigger on "verify", "run the checks", or finishing a code change in a Go/Next repo here.
---

# Verify gate

Run the verification chain for the target repo. If an argument names a repo/path, use it; otherwise infer the repo(s) from files changed this session (or ask which repo). Report each step `OK` or `ERR` with failing output. Do NOT commit, push, or open an MR.

## Go service / BFF / worker (`service/*`, `backend-for-frontend/*`, `worker-job/*`, `utils/go-common-*`)
Run from inside the repo, in order; continue the chain on failure to surface all problems:
1. `go build ./...`
2. `go vet ./...`
3. `gofmt -l .` -> must print nothing (any path = unformatted; run `gofmt -w` on it; note the PostToolUse hook already auto-gofmt's edited files)
4. `go test ./...` (regenerate mocks via mockery if an interface changed)

## Frontend (`frontend/next-frontend-*-service`)
Use node 24.14 (`nvm use 24.14`) if needed:
1. `npx tsc --noEmit`
2. `npx eslint <changed files>` (or the repo's `/eslint-fix`)
3. `npm run test` -> `vitest run` ONLY if the repo has a vitest config; if its AGENTS.md says "no test runner", skip and say so.

## Report
Compact table `step | OK/ERR`, then one line `VERIFIED: <repo>` or `FAILED: <n> step(s)`. Multiple repos -> verify + report each. Never claim verified without running the commands and seeing the output.
