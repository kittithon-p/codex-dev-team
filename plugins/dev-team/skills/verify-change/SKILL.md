---
name: verify-change
description: Run or recommend the right checks after a code change. Use before finalizing work.
argument-hint: [optional-scope, e.g. repo path]
---

# Verify Change

Scope:
$ARGUMENTS

Instructions:
1. Identify the touched repo(s); run all commands inside the repo, not the workspace root.
2. Choose the smallest relevant verification commands first, then the full gate:
   - Go repos: `go build ./...` -> `go vet ./...` -> `gofmt -l .` (must print nothing) -> `go test ./...`
   - Frontend repos: `npx tsc --noEmit` -> `npm run lint` -> `npm run test` (vitest, where configured)
   - Focused first: `go test ./<pkg>/ -run <TestName> -count=1` or `node_modules/.bin/vitest run <file>`
3. If a command fails, summarize the failure and likely cause.
4. If a command cannot be run, explain why and give the exact command to run manually.
5. For end-to-end verification of a running app, see the existing `verify` skill (runs the app and observes behavior).

Return:
- Commands run (with the repo they ran in)
- Results
- Failures
- Unverified areas
- Recommended next checks
