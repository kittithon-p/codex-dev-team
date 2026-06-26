---
name: git-flow
description: Plan safe Git Flow or branch workflow steps for features, fixes, hotfixes, releases, PRs, merges, and handoffs without performing destructive git operations.
argument-hint: [workflow-goal]
---

# Git Flow

Goal:
$ARGUMENTS

Use when:
- planning a feature branch
- preparing a fix branch
- deciding branch base
- splitting work into PRs
- planning merge order
- planning release or hotfix workflow
- coordinating agent-team work across files or branches

Do not use when:
- the user only asks for code implementation and no git workflow is needed
- the repo clearly uses a different workflow and Git Flow would add unnecessary overhead

Workflow:
1. Inspect current git status and branch.
2. Detect branch conventions from existing docs, branch names, CI, or contributing guides.
3. Decide whether the repo appears to use:
   - Git Flow
   - trunk-based development
   - GitHub Flow
   - release branches
   - unknown/custom workflow
4. Recommend a safe workflow.
5. Do not run branch-changing commands unless explicitly approved.
6. Produce a clear plan.

Output format:

## Git workflow verdict
## Current state
## Recommended branch strategy
## Suggested branch name
## Files/areas likely involved
## Commit/PR plan
## Merge/release considerations
## Risks
## Commands to run only after approval
