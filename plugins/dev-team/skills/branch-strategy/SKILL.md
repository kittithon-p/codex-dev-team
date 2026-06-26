---
name: branch-strategy
description: Decide safe branch naming, base branch, branch split, and ownership boundaries for a feature, bug fix, refactor, release, or hotfix.
argument-hint: [change-description]
---

# Branch Strategy

Change:
$ARGUMENTS

Workflow:
1. Inspect current branch and repo conventions.
2. Identify base branch candidates such as `main`, `master`, `develop`, `release/*`.
3. Identify whether this should be:
   - feature branch
   - bugfix branch
   - hotfix branch
   - release branch
   - docs branch
   - chore branch
4. Suggest branch name using existing conventions if known.
5. Suggest whether work should be one branch or split into multiple branches.
6. Identify branch ownership for agent-team work.

Output format:

## Branch recommendation
## Base branch
## Branch name
## Why this branch type
## Split vs single branch
## Agent/file ownership
## Risks
## Commands only after approval
