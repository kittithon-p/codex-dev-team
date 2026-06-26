---
name: implement-feature
description: Implement an approved feature plan with minimal, scoped changes. Use after planning is complete.
argument-hint: [approved-plan-or-feature]
---

# Implement Feature

Approved plan or feature:
$ARGUMENTS

Instructions:
1. Re-read the approved plan and relevant files.
2. Confirm the minimal set of repos and files to edit.
3. Implement in small steps, one repo at a time (each directory is its own git repo).
4. Follow existing patterns (response envelope via `util.ResponseSuccess`/`ResponseError`, `system/model|repository|service|handler` layering, app-specific frontend conventions).
5. Update or add tests where behavior changes.
6. Run the relevant verify gate inside each touched repo:
   - Go: `go build ./...` -> `go vet ./...` -> `gofmt -l .` (empty) -> `go test ./...`
   - Frontend: `npx tsc --noEmit` -> `npm run lint` -> `npm run test`
7. Report:
   - files changed (per repo)
   - implementation summary
   - commands run
   - results
   - risks or manual checks needed

Rules:
- Do not expand scope.
- Do not rewrite unrelated code.
- Do not claim verification if commands were not run.
- Do not commit, push, or open an MR unless explicitly asked.
