---
name: dev-team
description: Orchestrate a Codex subagent team for Nayoo feature work, bug fixes, refactors, investigations, reviews, performance/security/DevOps/database/docs tasks, release readiness, management updates, post-mortems, or multiple parallel work items. Use when the user explicitly invokes $dev-team or asks for AGENTS/dev-team to receive a task, plan it, delegate to subagents, wait for completion, apply 9arm disciplines when relevant, and report results. Do not use for single-file or tightly-coupled tasks where one agent is enough.
---

# dev-team - Codex subagent orchestration

Invoke this skill as `$dev-team [feature-or-bug-or-worklist]`.

The main session is the **lead AGENT**. It receives the user's request, inspects the real repo context, creates a plan, spawns only the relevant Codex subagents, assigns ownership, waits for results, reviews the outcome, reports back to the user, and closes completed agent threads only after the user is done with the team.

Subagents are **runtime-only**. They are spawned for the current task, inherit the parent session's sandbox and approval policy, do their own model/tool work, and report back. There is no standing team process.

## Prompt contract

Before spawning, restate or infer:
- **Goal**: feature, bug, review, investigation, release task, or batch of work items.
- **Context**: repos, files, errors, PRs, logs, examples, systems, or constraints that matter.
- **Constraints**: architecture, security, ownership, branch, API, release, or approval rules.
- **Done when**: checks, behavior, report, review verdict, or evidence required before completion.

Ask one concise clarification only when missing information changes risk, ownership, or safety. Otherwise choose conservative defaults and proceed.

## Lead contract

The lead AGENT must orchestrate, not implement, while a team is active.

Lead responsibilities:
- Inspect stack, repo structure, scripts, and task scope before selecting agents.
- Decide whether this needs a team, one subagent, or the main session.
- Split by repo or independent work item, not by vague concern.
- Assign exactly one owner per repo/file set.
- Spawn relevant subagents with clear prompts, allowed files, forbidden files, and required output.
- Keep cross-agent contracts explicit before implementation starts.
- Wait for subagent results before synthesizing.
- Run or delegate review/verification before final reporting.
- Report what happened, what changed, what failed, what was skipped, and what still needs human action.

Lead restrictions:
- Do not edit files while the team is active.
- Do not let two subagents edit the same file or repo area.
- Do not commit, push, deploy, merge, tag, release, or mutate external systems without explicit user approval.
- Do not claim checks passed unless a subagent actually ran them and reported the command.
- Do not close the team until the user confirms, or until the task is clearly complete and no follow-up steering is needed.

## Orchestration lifecycle

1. **Inspect**: Detect stack from real files (`package.json`, lockfiles, `go.mod`, `go.work`, `tsconfig`, `next.config.*`, Docker/CI files, README, env examples, migrations, test configs). For architecture questions, prefer `graphify query "<question>"` when `graphify-out/graph.json` exists.
2. **Classify**: Decide if the task is a feature, bug, investigation, review, DevOps task, release task, docs task, or batch.
3. **Plan**: Produce a `## Dev-team plan` before edits for broad, risky, cross-repo, or ambiguous work.
4. **Select agents**: Include only relevant custom agents. Do not spawn the whole roster.
5. **Assign ownership**: For every subagent, name allowed files/repos, forbidden files/repos, verification duty, and expected report format.
6. **Spawn**: Ask Codex to spawn subagents explicitly. Use `/agent` when the CLI needs inspection or steering of active threads.
7. **Coordinate**: Route contract questions through the lead. If ownership overlaps, pause and reassign before edits.
8. **Collect**: Wait for every requested result. For batch work, track each item status separately.
9. **Review**: Use `code-reviewer` for changed code. Use `test-qa` for verification evidence when code/config changes.
10. **Report**: Synthesize one final response with selected agents, results, changed files, checks, risks, skipped checks, and manual follow-ups.

## Planning output

For multi-file, multi-repo, risky, or batch work, present this plan before implementation:

```markdown
## Dev-team plan
### Goal
### Repo/task analysis
### Work items
### Selected agents
For each: why selected · custom agent name · skills to use · ownership · allowed files · forbidden files · verification duty · expected output
### Agents intentionally not used
### Contract boundaries
### Risks
### Test/verification plan
### Approval needed
```

If the user asked for immediate execution and the plan is low-risk, proceed after the plan. If the plan includes destructive, deployment, branch-changing, or broad data-impacting actions, wait for approval.

## Parallel and batch work

Use parallel subagents when work items are independent.

Batch rules:
- Treat each user-listed task as a work item with an id such as `T1`, `T2`, `T3`.
- Spawn at most one implementation owner per repo at a time.
- Run read-only reviewers in parallel when they do not block implementation.
- Serialize shared libraries, migrations, release steps, and cross-service contract changes.
- If two tasks touch the same repo/file, merge them under one owner or order them sequentially.
- If a task requires output from another task, record the dependency and do not spawn it early.
- Keep `agents.max_depth = 1`; child agents should not recursively delegate unless the user explicitly asks.
- Respect `agents.max_threads`; prefer 3-6 active threads unless the user asks for broader fan-out.

Batch tracking format:

```markdown
### Work items
- T1: <task> · owner <agent> · status planned/running/blocked/done · depends on <none/T#>
- T2: <task> · owner <agent> · status planned/running/blocked/done · depends on <none/T#>
```

## Agent selection

Core roles:
- `architect-planner`: planning, decomposition, ownership, architecture, risk. Use for broad or multi-repo work.
- `test-qa`: verification strategy, tests, checks, regressions. Use when code/config changes or the user asks for validation.
- `code-reviewer`: final review of diffs for correctness, regressions, security, and tests. Use before final response when code changed.

Nayoo repo-specific implementation roles:
- `frontend-next`: Next apps (`frontend/next-frontend-console-service`, `frontend/next-frontend-backoffice-service`, `frontend/next-frontend-nayoo-service`).
- `bff-go`: BFF repos (`backend-for-frontend/process-nayoo`, `backend-for-frontend/process-backoffice`).
- `system-service-go`: service repos (`service/go-system-*`, `go-tracking-service`).
- `worker-go`: workers (`worker-job/*`, change-stream, Meilisearch sync).
- `go-common-libs`: shared libs (`utils/go-common-*`). Serialize; never parallel with dependent consumers.
- `devops`: `devops/*`, per-service `k8s/`, Docker, CI/CD, env, observability, reliability.
- `release-mr`: git-flow, MR/PR, release/hotfix flow. Use only when explicitly requested.

Generic specialists:
- `backend-dev`: backend/API/service/auth/business logic when no repo-specific owner fits.
- `frontend-dev`: UI/client/routing/forms/styling when no repo-specific owner fits.
- `debugger`: investigation-heavy bugs, failing tests, traces, logs, flaky behavior.
- `security-reviewer`: auth, permissions, secrets, user data, payments, uploads, external APIs, input validation.
- `api-design-reviewer`: public/internal API design and compatibility.
- `integration-reviewer`: cross-service behavior, contracts, event/document sync, end-to-end assumptions.
- `test-author`: tests only.

Always state agents intentionally not used and why.

## Contract and ownership rules

- Split by repo, not by concern.
- One teammate owns one repo/file set at a time.
- Cross-service changes must agree the contract before implementation: BFF `process/gateway/*` type <-> service `system/model` wire type, including field names, snake_case, optionality, custom marshal behavior, and backward compatibility.
- Worker/search changes must agree the Meilisearch document schema before implementation.
- Shared libs (`utils/go-common-*`, `devops/nayoo-shared-lib`) are serialized: change and publish/bump first, then update consumers one at a time.
- Do not use DocumentDB-unsupported MongoDB operators such as `$facet`, `$graphLookup`, or `$sortByCount`.

## Spawn prompt template

Use this shape when spawning each subagent:

```text
You are <agent-name> for work item <T#>.
Goal: <specific goal>
Context: <repos/files/errors/contracts>
Ownership: you may edit/read <allowed>; do not touch <forbidden>.
Coordination: ask the lead before changing contracts or overlapping ownership.
Verification: run/report <commands or checks>, or explain why skipped.
Output: report changed files, commands run, result, blockers, risks, and follow-ups.
```

For read-only agents, add:

```text
Read-only review/investigation only. Do not modify files.
```

For implementation agents, add:

```text
Make the smallest scoped change. Do not commit, push, deploy, or open an MR.
```

## Skills and tools

Use skills on demand. Do not preload unrelated skills or mention skills that are not installed.

Default local routing:
- Planning: `plan-feature`, `branch-strategy` when git workflow matters.
- Implementation: `implement-feature` plus stack-specific skills.
- Debugging: `debug-issue`, `debug-mantra` for failing tests/runtime bugs.
- Review: `review-diff`, `scrutinize` for risky or final review.
- Verification: `verify-change`, `test-plan`, `regression-check`.
- Security: `security-audit`, `authz-review`, `secrets-check`.
- Docs/release: `api-docs`, `changelog-entry`, `release-checklist`, `pr-prep`.

Stack routing:
- Go: use relevant Go skills for `.go`, `go.mod`, tests, concurrency, DB, security, performance, observability, CI.
- React/Next/Vercel: use relevant React, web design, composition, writing, optimize, or deploy skills only when touched.
- DevOps/IaC: use generator -> validator loops; prefer lint, validate, dry-run, plan-style commands.

MCP:
- Use MCPs only when they unlock the task. Jenkins/Grafana or other external systems can be read for diagnosis, but triggering jobs or mutating external systems requires explicit approval.

## 9arm skill routing

Use `thananon/9arm-skills` on demand when installed. If a 9arm skill is relevant but missing, mention it in the plan as optional and continue with the equivalent discipline in this skill. Install command:

```bash
npx skills add thananon/9arm-skills
```

9arm skills to route:
- `debug-mantra`: debugging-heavy bugs, failing tests, runtime errors, stack traces, logs, flaky behavior, regressions, or any "investigate/diagnose/why is this failing" task.
- `scrutinize`: plan review, PR/diff review, risky implementation review, architecture second opinion, or "is there a simpler way" work.
- `post-mortem`: after a significant bug fix is known and validated; use for engineering RCA only.
- `management-talk`: leadership/PM/release/status/Slack/email/standup wording based on engineering content.
- `qwenchance`: Claude-specific context-budget/handoff discipline; do not invoke directly in Codex unless installed and explicitly requested. Apply the principle instead: bound long loops, summarize state, and create a handoff before context gets too large.
- `qwen-agent`: Claude/Qwen command delegation; do not route Codex work to it. Use Codex subagents instead.

Agent mapping:
- `debugger` owns `debug-mantra`. The lead must require: reproducibility first, fail-path tracing second, hypothesis falsification third, and a breadcrumb ledger before proposing a fix.
- `code-reviewer` owns `scrutinize`. It must question intent, look for a simpler approach, trace the real code path, verify claims, and report actionable findings with evidence.
- `docs-writer` owns `post-mortem` when available; if no docs-writer subagent exists, the lead asks the fixing owner plus `debugger` for the required facts, then drafts only after validation is proven.
- `release-mr` or a docs/status owner uses `management-talk` only when the user asks for a leadership, PM, release, Slack, email, standup, or less-technical version.

9arm operating gates:
- Do not propose a debug fix before a reliable repro or a clearly stated missing-repro blocker.
- Do not accept a single root-cause hypothesis until it has a disproof attempt or explicit evidence.
- Do not write `post-mortem` content without all four inputs: reliable repro, known root cause, identified fix, and validation evidence.
- Do not use `management-talk` for engineering RCA; compose it after the engineering truth exists.
- Do not let `scrutinize` become style review. It should focus on whether the change should exist, whether it works end to end, and what hidden assumptions or regressions matter.

Planning additions when 9arm applies:

```markdown
### 9arm discipline routing
- <skill>: <agent> · why selected · installed/missing · required evidence · fallback if missing
```

Final report additions when 9arm applies:

```markdown
### 9arm results
- <skill>: <agent> · evidence produced · result · gaps/follow-ups
```

## Verification rules

- Use repo's real commands from package scripts, Makefiles, Go modules, CI config, or README.
- Run focused checks first; broaden when practical.
- Report exact commands run and outcomes.
- Report skipped checks with reasons.
- Never claim green without evidence.
- Do not run deployment, production mutation, destructive database, cluster, or git-changing commands without explicit user approval.

## Final report

After all subagents finish, the lead reports:

```markdown
## Dev-team result
### Goal
### Work item status
### Selected agents
### Detected stack
### What changed
### Files changed
### Checks run
### Checks skipped
### Review findings
### Security impact
### Database impact
### Build/deploy impact
### Performance impact
### Release/git impact
### Risks and follow-ups
### Approval-gated actions not run
```

For review-only work, lead with findings ordered by severity.

## Distribution contract

- Keep `.agents/skills/dev-team/` as the editable repo skill.
- Keep `plugins/dev-team/skills/dev-team/` synced for plugin users.
- Keep `.agents/plugins/marketplace.json` pointing at `./plugins/dev-team`.
- Keep custom agents in `.codex/agents/*.toml`.
- If this skill changes, update both the repo skill and plugin skill before sharing.
