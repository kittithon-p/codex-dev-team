---
name: review-diff
description: Review the current git diff for correctness, maintainability, security, missing tests, and unintended changes.
argument-hint: [repo-path (optional, e.g. service/go-system-listing-service)]
---

# Review Diff

Target repo(s):
$ARGUMENTS

## Repos with uncommitted changes

!`for d in */ */*/; do [ -d "${d}.git" ] && [ -n "$(git -C "$d" status --porcelain 2>/dev/null | head -c1)" ] && echo "DIRTY: $d"; done; true`

## Instructions

The workspace root is not a git repo. For each target repo (from $ARGUMENTS, or each DIRTY repo above):

1. Run `git -C <repo> diff --stat` then `git -C <repo> diff` (add `--cached` if staged).
2. Review the diff.

Return:
- Summary of what changed (per repo)
- Blocking issues
- Non-blocking suggestions
- Missing tests
- Security or data handling concerns (secrets, `.env.local`, exposed bucket URLs)
- Verification still needed

Rules:
- Prioritize correctness over style.
- Include file paths.
- Check cross-repo contract sync: BFF `process/gateway/*` <-> service `system/model` field shapes.
- Flag DocumentDB-unsupported operators and per-request `count()` on large collections.
- If there are no major issues, say so clearly.
