---
name: commit-plan
description: Plan focused commits and write safe commit messages based on the actual diff without committing.
argument-hint: [optional-scope]
---

# Commit Plan

Scope:
$ARGUMENTS

Rules:
- Do not commit unless explicitly asked.
- Inspect the actual diff before writing commit messages.
- Do not include secrets in commit messages.
- Do not invent changes.
- Use existing commit style if detectable.
- If no style is detectable, recommend concise conventional-style messages, but mark them as suggestions.

Workflow:
1. Inspect `git diff --stat`.
2. Inspect relevant `git diff`.
3. Group changes into logical commits.
4. Identify unrelated changes that should not be included.
5. Suggest commit messages.

Output format:

## Diff summary
## Suggested commit grouping
## Suggested commit messages
## Files per commit
## Risks or unrelated changes
## Commands only after approval
