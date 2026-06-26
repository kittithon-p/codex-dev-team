---
name: merge-conflict-plan
description: Plan safe merge conflict resolution by understanding both sides, preserving intent, and identifying verification steps.
argument-hint: [conflict-context]
---

# Merge Conflict Plan

Context:
$ARGUMENTS

Rules:
- Do not resolve conflicts blindly.
- Do not choose ours/theirs without understanding intent.
- Do not run destructive git commands without explicit approval.
- Preserve user changes.
- After conflict resolution, run focused tests/checks.

Workflow:
1. Identify conflicted files.
2. Understand both sides of each conflict.
3. Determine intended final behavior.
4. Propose resolution strategy per file.
5. Identify tests/checks needed after resolution.
6. Ask for approval before editing conflicted files if broad/risky.

Output format:

## Conflict summary
## Files affected
## Intent from current branch
## Intent from incoming branch
## Proposed resolution
## Risks
## Verification plan
## Commands only after approval
