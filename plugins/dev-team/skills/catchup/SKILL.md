---
name: catchup
description: Restore context across the multi-repo workspace — recent branch/status/commits per repo — after a /clear or when resuming work. Trigger on "catch up", "what was I doing", "where did we leave off", or resuming a session.
---

# Catchup (multi-repo)

Rebuild a picture of recent work. The workspace ROOT is NOT a git repo — each service is its own repo, so always use `git -C <repo> ...`.

If an argument names a repo, focus on it. Otherwise survey the active repos. Candidate dirs: `backend-for-frontend/*`, `service/go-*`, `worker-job/*`, `frontend/next-frontend-*-service`, `utils/go-common-*`, `devops/*`.

For each repo that has a `.git` and is NOT clean on its default branch (or has uncommitted changes), report:
- current branch: `git -C <repo> branch --show-current`
- working-tree changes: `git -C <repo> status --short`
- recent commits: `git -C <repo> --no-pager log --oneline -5`
- divergence from develop: `git -C <repo> rev-list --left-right --count origin/develop...HEAD` (if `develop` exists)

Then summarize, grouped by repo, in plain text (no emojis): which repos have in-flight work (branch + dirty files), what the last commits were doing, any half-finished thread. Omit clean repos on their default branch. End with a one-line suggestion of where to resume.
