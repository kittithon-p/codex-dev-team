---
name: debug-issue
description: Investigate a bug, failing test, runtime error, log, or stack trace using evidence-first debugging.
argument-hint: [bug-description-or-error]
---

# Debug Issue

Issue:
$ARGUMENTS

Instructions:
1. Gather evidence from error messages, tests, logs, and relevant code. Localize the failing hop first: Next app -> BFF (:8080/:8088) -> system service (:8081-:8087) -> DocumentDB/Meilisearch — curl each hop directly when possible.
2. Form 2-4 hypotheses. Common causes in this workspace: BFF gateway <-> service model field drift (snake_case/MarshalJSON), DocumentDB-unsupported operators, env differences local/uat/prod, worker sync lag.
3. Test hypotheses using focused reads or commands (`git -C <repo> log --oneline -10` for recent regressions; `graphify query` for unfamiliar areas).
4. Identify the most likely root cause.
5. Recommend the smallest safe fix.
6. Do not edit files unless explicitly asked. Never print `.env.local` or secrets while gathering evidence.

Return:
- Reproduction status
- Evidence
- Hypotheses considered
- Root cause
- Recommended fix
- Tests to verify the fix
