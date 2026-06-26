---
name: hotfix-flow
description: Plan urgent hotfix workflow from production issue to fix branch, verification, release, merge-back, and postmortem without performing risky git operations.
argument-hint: [hotfix-context]
---

# Hotfix Flow

Hotfix context:
$ARGUMENTS

Rules:
- Treat hotfixes as high-risk.
- Do not branch, merge, tag, push, deploy, or release without explicit approval.
- Keep scope minimal.
- Verify the fix before release.
- Include merge-back and postmortem follow-up.

Workflow:
1. Identify production issue and severity.
2. Identify likely base branch.
3. Recommend hotfix branch name.
4. Define minimal fix scope.
5. Define verification commands.
6. Define release/rollback plan.
7. Define merge-back plan.
8. Recommend postmortem if appropriate.

Output format:

## Hotfix plan
## Severity and scope
## Branch recommendation
## Minimal fix
## Verification
## Release/rollback plan
## Merge-back plan
## Postmortem follow-up
## Commands only after approval
