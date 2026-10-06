---
name: dev-team
description: Orchestrate a Codex subagent team for Nayoo feature work, bug fixes, refactors, investigations, reviews, performance/security/DevOps/database/docs tasks, release readiness, management updates, post-mortems, or multiple parallel work items. Use when the user explicitly invokes $dev-team or asks for AGENTS/dev-team to receive a task, plan it, delegate to subagents, wait for completion, apply 9arm disciplines when relevant, and report results. Do not use for single-file or tightly-coupled tasks where one agent is enough.
---

# dev-team - Codex subagent orchestration

Invoke this skill as `$dev-team [feature-or-bug-or-worklist]`.

> Running in Antigravity (agy) instead of Codex? Use the sibling skill `dev-team-agy` (`../dev-team-agy/SKILL.md`) — it adapts this workflow to agy subagents and references the runtime-agnostic sections below.

The main session is the **lead AGENT**. It receives the user's request, inspects the real repo context, creates a plan, spawns only the relevant Codex subagents, assigns ownership, waits for results, reviews the outcome, and reports back to the user. Mark child turns complete when their work is done; claim that a thread was explicitly closed only when the runtime exposes and successfully performs a close operation.

Subagents are **runtime-only**. They are spawned for the current task, inherit the parent session's sandbox and approval policy, do their own model/tool work, and report back. There is no standing team process.

## Runtime and context discipline

- Parallelize independent read-heavy work such as code-path mapping, documentation research, test execution, log analysis, and review. Serialize write-heavy work unless repositories and file ownership are disjoint.
- Give each subagent one bounded work item, explicit allowed/forbidden scope, and a required result shape. Pass full history only when prior decisions are essential; otherwise use a self-contained prompt or the smallest useful recent context.
- Treat prompt ownership as coordination, not isolation. Read-only work must use a read-only custom agent/sandbox when available; child agents inherit the parent turn's live sandbox and approval overrides. If the runtime cannot select that sandbox and the parent is writable, a read-only prompt is an instruction boundary only, not mechanical enforcement; do not use that fallback when a hard read-only boundary is required.
- Inspect the callable spawn schema before claiming a custom agent is active. If the runtime cannot select an exact custom-agent name, treat the child as a generic subagent and disclose that its TOML/model/sandbox was not applied. Report the requested profile and the parent's configured default separately; never claim the child's effective model, effort, role, or sandbox unless the runtime exposes evidence for it.
- Use `send_message` to add context to a running turn, `followup_task` to start a new turn on an idle agent, and `interrupt_agent` only when the current work is wrong, unsafe, or obsolete.
- Wait for every requested result before synthesis. Ask agents for distilled findings, changed files, exact commands/results, blockers, and residual risk; keep raw logs in the agent thread.
- Respect the live concurrency cap and keep `agents.max_depth = 1`. More fan-out costs more tokens and coordination time; never create recursive delegation by default.

## Codex configuration boundaries

- Follow the active `AGENTS.md` chain from global scope through the closest directory; nearer guidance wins. Keep durable root guidance concise and put task workflows in skills.
- Keep project `agents.max_threads = 6` and `agents.max_depth = 1` unless a measured need justifies a reviewed config change. The thread cap is a ceiling, not a spawn target.
- This repository uses task-specific GPT-5.6 model and effort pins in repo custom-agent TOMLs. `.codex/config.toml` sets the lead/project default to `gpt-5.6-sol` / `xhigh`; that default is not proof of a generic child's effective model or effort. Report requested, configured, and runtime-verified values separately. Do not change model, reasoning, service tier, or Fast mode again without an explicit user request.
- Treat execpolicy `.rules` as command capability only. An `allow` decision never waives this skill's explicit-user-request gates for commit, push, merge, deploy, release, or external mutation.
- If the user requests a persistent rule, use the narrowest prefix with justification plus `match`/`not_match` examples, then validate it with `codex execpolicy check`.

## Prompt contract

Before spawning, restate or infer:
- **Goal**: feature, bug, review, investigation, release task, or batch of work items.
- **Context**: repos, files, errors, PRs, logs, examples, systems, or constraints that matter.
- **Constraints**: architecture, security, ownership, branch, API, release, or approval rules.
- **Done when**: checks, behavior, report, review verdict, or evidence required before completion.

## Task model and effort routing

Select the work profile before selecting the owning agent. These pins are the repository policy:

| Work profile | Model | Effort | Preferred agent |
|---|---|---|---|
| Adviser / first escalation | `gpt-5.6-sol` | `max` | `adviser` |
| Adviser / final escalation | `gpt-5.6-sol` | `ultra` | `adviser-ultra` |
| Read code or primary docs | `gpt-5.6-luna` | `low` | `code-mapper`, `docs-researcher` |
| Reproduce or find a bug | `gpt-5.6-terra` | `medium` | `debugger`, `browser-debugger` |
| Fix a bug | `gpt-5.6-sol` | `medium` | `bug-fixer` |
| Refactor | `gpt-5.6-sol` | `high` | matching domain owner |
| Add a feature | `gpt-5.6-sol` | `high` | matching domain owner |
| Design architecture or cross-service contracts | `gpt-5.6-sol` | `xhigh` | `architect-planner`, `api-design-reviewer`, `integration-reviewer` |
| Security audit | `gpt-5.6-sol` | `xhigh` | `security-reviewer` |
| Generate routine CRUD | `gpt-5.6-terra` | `medium` | `crud-generator` |
| Write or update unit tests | `gpt-5.6-terra` | `medium` | `test-author` |
| Run tests or QA verification | `gpt-5.6-terra` | `medium` | `test-qa` |
| Review a PR or final diff | `gpt-5.6-sol` | `high` | `code-reviewer` |
| DevOps or IaC implementation | `gpt-5.6-sol` | `high` | `devops` |
| Release or MR workflow | `gpt-5.6-terra` | `medium` | `release-mr` |

Routing rules:
- The adviser family is read-only and never implements. `adviser` at Max is the mandatory first escalation for a blocked owner. `adviser-ultra` is the only role allowed to use Ultra and is available only after a runtime-verified Max run returns an unresolved result with evidence. Never run Max and Ultra advisers in parallel for the same blocker.
- Adviser levels are hard runtime requirements, not prompt labels. A generic subagent must never be described or counted as Adviser Max or Adviser Ultra. If the callable runtime cannot select and verify the required adviser profile, fail closed with `blocked-needs-adviser-runtime`; preserve the escalation packet and tell the user that the mandatory adviser could not be invoked.
- Keep `agents.max_depth = 1`. A blocked subagent returns an escalation packet to the lead; the lead calls Adviser Max, routes its answer back to the owner, and calls Adviser Ultra only if Max explicitly could not resolve the blocker. Do not raise the depth or let advisers recursively delegate.
- For bug fixes and CRUD, assign ownership to `bug-fixer` or `crud-generator` for exactly one repo/file set and include the relevant domain constraints in the spawn prompt. Do not also assign a domain owner to the same files.
- For refactors and features, use the matching domain owner, whose repo agent config is pinned to Sol/high.
- Split mixed requests into work items by profile. Do not ask one child thread to switch model or effort mid-task.
- For supported work not listed in the table, use the selected role's explicit TOML profile when the runtime can select it. Otherwise use a scoped generic subagent and report the profile as unverified; do not invent a model/effort mapping.
- For bug work, use one state machine: deterministic reproduction and fail-path evidence already exist -> assign `bug-fixer`; otherwise assign read-only `debugger` first, collect a reproduction/evidence packet, end that investigation ownership, then transfer the scoped files to `bug-fixer`. Never run both as concurrent owners of the same files.

Adviser escalation protocol:
1. The blocked owner stops edits that depend on the unresolved decision and returns: goal, exact blocker, files/contract involved, commands and results, hypotheses tried, disproof evidence, residual options, and one precise adviser question.
2. Before spawning, the lead verifies that the callable runtime can select the exact `adviser` custom role and Sol/Max profile. If it cannot, set `blocked-needs-adviser-runtime`, do not substitute a generic child, and report the preserved packet to the user.
3. The lead spawns the verified `adviser` (Sol/Max) read-only with that packet. If Max provides a usable recommendation, the lead sends it to the original owner and resumes the same work item.
4. If the verified Max run explicitly reports that the blocker remains unresolved, the lead records the gap, verifies exact Ultra selection, and spawns `adviser-ultra` (Sol/Ultra) with the original packet plus the Max analysis. If exact Ultra selection is unavailable, set `blocked-needs-adviser-runtime` and report it.
5. If Ultra resolves the blocker, the recommendation returns through the lead to the original owner, who remains the only writer. Advisers never edit, approve risky actions, or take ownership.
6. If Ultra still cannot resolve the blocker, set `blocked-needs-user` or `blocked-needs-external-state`, preserve both adviser analyses, stop dependent edits, and ask the user one precise decision or authority question.

Ask one concise clarification only when missing information changes risk, ownership, or safety. Otherwise choose conservative defaults and proceed.

## Lead contract

The lead AGENT must orchestrate, not implement, while a team is active.

A team is active while any child turn is running or a follow-up has been dispatched and not completed. Completed/runtime-managed child records do not keep the team active. For a single-file or tightly coupled task where no team is started, the main session may implement directly.

Lead responsibilities:
- Inspect stack, repo structure, scripts, and task scope before selecting agents.
- Decide whether this needs a team, one subagent, or the main session.
- Run `grilling` with the user before planning, so no scope/intent/behavior/contract/rollout decision is assumed. Facts are looked up, never asked.
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
- Do not claim checks passed unless the lead or assigned verifier actually ran them and observed the command result in the current task.
- Do not claim a child thread was closed unless the runtime exposes and successfully performs that operation. Otherwise report it as completed/runtime-managed.

## Orchestration lifecycle

1. **Inspect**: Detect stack from real files (`package.json`, lockfiles, `go.mod`, `go.work`, `tsconfig`, `next.config.*`, Docker/CI files, README, env examples, migrations, test configs). For architecture questions, prefer `graphify query "<question>"` when `graphify-out/graph.json` exists.
2. **Classify**: Decide if the task is a feature, bug, investigation, review, DevOps task, release task, docs task, or batch.
3. **Grill, plan, and approve**: First run `grilling`: ask the user every open decision (scope, intent, behavior, contract shape, rollout) in rounds of multiple-choice questions (2-4 lettered options each, recommended option first and marked `(Recommended)`, user replies `Q1: A`; use the runtime's structured choice tool instead when one is available), look up facts yourself, and stop when the frontier is empty and the user confirms. Skip only when the request already pins every decision or the user says "no grill", and say so in one line. Then produce a `## Dev-team plan` before edits for broad, risky, cross-repo, or ambiguous work, then obtain user approval before any team member edits.
4. **Select agents**: Include only relevant custom agents when the runtime can select them; otherwise use scoped generic subagents and report the fallback. Do not spawn the whole roster.
5. **Assign ownership**: For every subagent, name allowed files/repos, forbidden files/repos, verification duty, and expected report format.
6. **Spawn**: Ask Codex to spawn subagents explicitly. Use `/agent` when the CLI needs inspection or steering of active threads.
7. **Coordinate**: Route contract questions through the lead. If ownership overlaps, pause and reassign before edits. A subagent that hits an undecided product/contract question does not guess: it returns the question to the lead, who runs another `grilling` round with the user before that work item resumes.
8. **Collect**: Wait for every requested result. For batch work, track each item status separately.
9. **Verify**: If `test-author` is selected, it alone owns the explicitly assigned test files; otherwise the sole implementation owner may update tests inside its approved ownership. After all intended writes finish, `test-qa` runs verification without editing source files.
10. **Review (scrutinize loop, 2-5 rounds)**: Run `code-reviewer` (`scrutinize` + `ponytail-review`) after all implementation and test-file writes. Route blocking findings back to the sole owner; after remediation, rerun affected verification, then run the next review round on the new diff. Round 2 always runs, even if round 1 was clean; stop when a round >= 2 has zero blocking findings. Hard cap 5: blocking findings still open after round 5 are reported as `UNRESOLVED after 5 rounds` (or escalated if a blocker packet fits), never a round 6. Each report is headed `Round N/5`; the final report states how many rounds ran.
11. **Report or block**: Report only after required checks/review are green, or return an explicit blocked state with evidence, skipped checks, risks, and the one action needed to resume.

## Planning output

For multi-file, multi-repo, risky, or batch work, present this plan before implementation:

```markdown
## Dev-team plan
### Goal
### Settled decisions (from grilling) / Open - blocked on user
### Repo/task analysis
### Work items
### Selected agents
For each: why selected · custom agent name or generic fallback · skills to use · ownership · allowed files · forbidden files · verification duty · expected output
### Agents intentionally not used
### Contract boundaries
### Risks
### Test/verification plan
### Approval needed
```

Wait for explicit approval after presenting the plan. A prior user instruction counts only when it clearly approves the same concrete work items, ownership, and risks and the plan has not expanded; otherwise ask before edits. Destructive, deployment, branch-changing, or broad data-impacting actions always require their own explicit approval.

## Parallel and batch work

Use parallel subagents when work items are independent.

Batch rules:
- Treat each user-listed task as a work item with an id such as `T1`, `T2`, `T3`.
- Spawn at most one implementation owner per repo at a time.
- Run read-only reviewers in parallel when they do not block implementation.
- Serialize shared libraries, migrations, release steps, and cross-service contract changes.
- If two tasks touch the same repo/file, merge them under one owner or order them sequentially.
- If a task requires output from another task, record the dependency and do not spawn it early.
- Keep `agents.max_depth = 1`; child agents do not recursively delegate. A user request for deeper delegation requires a separate reviewed configuration change before it can be attempted.
- Respect the live `agents.max_threads` and runtime slot cap; leave room for the lead and do not assume configured capacity is currently available.

Batch tracking format:

```markdown
### Work items
- T1: <task> · owner <agent> · status planned/running/blocked/done · depends on <none/T#>
- T2: <task> · owner <agent> · status planned/running/blocked/done · depends on <none/T#>
```

## Agent selection

Core roles:
- `architect-planner`: planning, decomposition, ownership, architecture, risk. Use for broad or multi-repo work.
- `test-qa`: verification strategy, command execution, checks, and regressions without source-file edits. Use after intended writes finish or when the user asks for validation.
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
- `adviser`: read-only Sol/Max first escalation for blocked owners and explicit high-impact second opinions.
- `adviser-ultra`: read-only Sol/Ultra final escalation only after Adviser Max returns unresolved evidence.
- `code-mapper`: read-only execution-path and ownership mapping before broad or cross-service changes.
- `docs-researcher`: read-only verification of version-specific APIs and framework behavior from primary sources.
- `browser-debugger`: read-only browser reproduction and evidence capture before frontend implementation.
- `backend-dev`: backend/API/service/auth/business logic when no repo-specific owner fits.
- `frontend-dev`: UI/client/routing/forms/styling when no repo-specific owner fits.
- `debugger`: investigation-heavy bugs, failing tests, traces, logs, flaky behavior.
- `security-reviewer`: auth, permissions, secrets, user data, payments, uploads, external APIs, input validation.
- `api-design-reviewer`: public/internal API design and compatibility.
- `integration-reviewer`: cross-service behavior, contracts, event/document sync, end-to-end assumptions.
- `test-author`: tests only.

Task-profile implementation roles:
- `bug-fixer`: Sol/medium owner for a scoped bug fix after a reliable reproduction and fail path are known.
- `crud-generator`: Terra/medium owner for routine, well-specified CRUD with established repository patterns and contracts.

State only plausible candidate agents intentionally not used and why; do not enumerate the entire roster.

For browser-visible bugs, run `browser-debugger` before the frontend owner when a reachable target and browser capability exist. The debugger returns a sanitized reproduction packet; the frontend owner makes the scoped fix; `test-qa` reruns the same flow. Do not use browser-debugger for static code review or when the issue already has deterministic test evidence.

For browser-visible bugs, run `browser-debugger` before the frontend owner when a reachable target and browser capability exist. The debugger returns a sanitized reproduction packet; the frontend owner makes the scoped fix; `test-qa` reruns the same flow. Do not use browser-debugger for static code review or when the issue already has deterministic test evidence.

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
Context handoff: <self-contained facts and decisions; do not dump the full transcript unless required>
Model profile: <requested model/effort; exact custom role or generic fallback; parent configured default; runtime-verified child values or "unverified">
Ownership: you may edit/read <allowed>; do not touch <forbidden>.
Coordination: ask the lead before changing contracts or overlapping ownership.
Escalation: if blocked, stop dependent edits and return the adviser packet required by this skill; do not spawn another agent yourself.
Verification: run/report <commands or checks>, or explain why skipped.
Output: return a concise summary with changed files, commands run, result, blockers, risks, and follow-ups; keep raw logs in this thread.
```

For read-only agents, add:

```text
Read-only review/investigation only. Do not modify files.
If exact read-only sandbox selection is unavailable, this is a prompt boundary only; stop and report if the task requires mechanical isolation.
```

For implementation agents, add:

```text
Make the smallest scoped change. Do not commit, push, deploy, or open an MR.
```

## Skills and tools

Use skills on demand. Do not preload unrelated skills. This internal repo/plugin vendors the focused and specialist skills that `$dev-team` routes to, so teammates should not need separate installs for the bundled workflow.

Default local routing:
- Planning: `grilling` (lead, before the plan), `plan-feature`, `branch-strategy` when git workflow matters.
- Implementation: `implement-feature` plus stack-specific skills.
- Debugging: `debug-issue`, `debug-mantra` for failing tests/runtime bugs.
- Review: `review-diff`, `scrutinize` for risky or final review.
- Verification: `verify-change`, `backend-test-strategy`.
- Security: `authz-review`, `env-audit`, plus stack-specific security skills.
- Docs/release: `api-docs`, `changelog-entry`, `commit-plan`, `finishing-a-development-branch`, `hotfix-flow`.

Stack routing:
- Go: use relevant Go skills for `.go`, `go.mod`, tests, concurrency, DB, security, performance, observability, CI.
- React/Next/Vercel: use relevant React, web design, composition, writing, optimize, or deploy skills only when touched.
- DevOps/IaC: use generator -> validator loops; prefer lint, validate, dry-run, plan-style commands.

MCP:
- Use MCPs only when they unlock the task. Jenkins/Grafana or other external systems can be read for diagnosis, but triggering jobs or mutating external systems requires explicit approval.

## 9arm skill routing

Use the vendored 9arm skills on demand. If a 9arm skill is relevant but missing because the local install was customized or damaged, mention it in the plan and continue with the equivalent discipline in this skill.

9arm skills to route:
- `debug-mantra`: debugging-heavy bugs, failing tests, runtime errors, stack traces, logs, flaky behavior, regressions, or any "investigate/diagnose/why is this failing" task.
- `scrutinize`: plan review, PR/diff review, risky implementation review, architecture second opinion, or "is there a simpler way" work.
- `post-mortem`: after a significant bug fix is known and validated; use for engineering RCA only.
- `management-talk`: leadership/PM/release/status/Slack/email/standup wording based on engineering content.
- `qwenchance`: Claude-specific context-budget/handoff discipline; do not invoke directly in Codex unless installed and explicitly requested. Apply the principle instead: bound long loops, summarize state, and create a handoff before context gets too large.
- `qwen-agent`: Claude/Qwen command delegation; do not route Codex work to it. Use Codex subagents instead.

Agent mapping:
- `debugger` owns `debug-mantra`. The lead must require: reproducibility first, fail-path tracing second, hypothesis falsification third, and a breadcrumb ledger before proposing a fix.
- `code-reviewer` owns `scrutinize`. It must question intent, look for a simpler approach, trace the real code path, verify claims, and report actionable findings with evidence. Scrutinize must run for at least 2 rounds and at most 5 rounds.
- `docs-writer` owns `post-mortem` when available; if no docs-writer subagent exists, the lead asks the fixing owner plus `debugger` for the required facts, then drafts only after validation is proven.
- `release-mr` or a docs/status owner uses `management-talk` only when the user asks for a leadership, PM, release, Slack, email, standup, or less-technical version.

9arm operating gates:
- Do not propose a debug fix before a reliable repro or a clearly stated missing-repro blocker.
- Do not accept a single root-cause hypothesis until it has a disproof attempt or explicit evidence.
- Do not write `post-mortem` content without all four inputs: reliable repro, known root cause, identified fix, and validation evidence.
- Do not use `management-talk` for engineering RCA; compose it after the engineering truth exists.
- Do not let `scrutinize` become style review. It should focus on whether the change should exist, whether it works end to end, and what hidden assumptions or regressions matter. Scrutinize must run for at least 2 rounds (Round 1: findings/simplification -> Remediation -> Round 2: adversarial re-verification) and at most 5 rounds. Escalate if unresolved by Round 5.

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

## Superpowers workflow routing

External pack: `obra/superpowers` (official plugin `superpowers@claude-plugins-official`). It is optional for repo/plugin users and must remain an on-demand process overlay, not a second team orchestrator.

- Availability: verify the exact namespaced `superpowers:<skill>` is exposed before selecting it. Do not fall back silently to an unprefixed same-name personal skill because that copy may be older or from another source.
- Precedence: user instructions, repo `AGENTS.md`, and this skill's ownership, approval, verification, and reporting rules override conflicting external steps.
- Adopt selectively: `superpowers:brainstorming` for ambiguous new behavior; `superpowers:systematic-debugging` when its root-cause phases add value beyond `debug-mantra`; `superpowers:test-driven-development` for testable behavior changes; `superpowers:receiving-code-review` before applying ambiguous or questionable review feedback; `superpowers:writing-skills` for reusable behavior-shaping skill changes with justified before/after evaluation.
- Avoid duplicate gates: normal `$dev-team` already owns planning, review, and fresh verification. Do not also invoke `superpowers:writing-plans`, `superpowers:requesting-code-review`, or `superpowers:verification-before-completion` unless the user explicitly requests the Superpowers variant or it adds a concrete missing check.
- Do not route `superpowers:using-superpowers`, `superpowers:dispatching-parallel-agents`, `superpowers:subagent-driven-development`, or `superpowers:executing-plans` inside a running team; they duplicate orchestration and can conflict with one-owner-per-repo and live-slot rules.
- Git safety: `superpowers:using-git-worktrees` and `superpowers:finishing-a-development-branch` require an explicit user request and one confirmed repo. A skill never grants permission to commit, push, merge, delete branches, or create/remove worktrees.
- Visual companion: start the optional brainstorming companion only after user opt-in; it may make a version-only telemetry request. `SUPERPOWERS_DISABLE_TELEMETRY=1` disables that request.
- Installation: never install, update, or vendor Superpowers automatically. If the exact namespaced skill is absent, use the local equivalent and report the missing optional skill.

## Ponytail simplification routing

Integrated pack: `DietrichGebert/ponytail` (vendored in `.agents/skills/` and `plugins/dev-team/skills/`, installed in Codex & Antigravity).

- **Active simplification discipline**:
  - **Implementation**: Implementation owners (`bug-fixer`, domain dev agents) apply the Ponytail ladder: (1) Does it need to exist at all (YAGNI)? (2) Already in codebase? (3) Standard library does it? (4) Native platform feature covers it? (5) Minimal working code. Strictly reject unrequested abstractions, factories with one product, interfaces with one implementation, and speculative boilerplate.
  - **Review**: `code-reviewer` executes `ponytail-review` alongside `scrutinize` to actively detect and eliminate removable complexity, bloat, and unneeded abstractions.
  - **Audit commands**: Use `ponytail-audit` for repo-wide complexity inventory, `ponytail-debt` for debt estimation, and `ponytail-gain` for simplification metrics when requested.
- **Safety invariant**: Never simplify away input validation, error handling, security gates, data-loss protection, explicit contracts, or meaningful test coverage.
- Lifecycle: keep Ponytail default mode `off` (routing above applies the skills explicitly). Active mode injects into subagents; never install, activate, upgrade, remove, or set `PONYTAIL_SUBAGENT_MATCHER` automatically.

## TypeSafe AI routing

External pack: `typesafe-ai/skills` provides one installed application-integration skill, `typesafe-ai`, for TypeSafe System One's typed semantic judgments. It is not an orchestration or general coding skill.

- Automatic selection: during task intake, invoke `typesafe-ai` without requiring the user to name it when the request mentions TypeSafe, System One, or Jev; the repository already uses TypeSafe; or the requested behavior needs a programmable typed semantic judgment (for example routing, ranking, extraction, verification, or a prompt-and-parse replacement). List it under selected external skills.
- This automatic selection informs design and implementation planning; it never by itself authorizes adding a dependency, sending data to TypeSafe, creating credentials, or changing an application. Do not route ordinary API, database, or prompt work to it.
- Availability gate: verify the exact `typesafe-ai` skill is available before use. If unavailable, use the owning implementation agent plus `docs-researcher` for primary-doc research; do not present it as active.
- Safety: read the current TypeSafe API/SDK docs before implementation; keep credentials server-side; preserve deterministic rules and side effects in application code; validate uncertainty thresholds and representative outcomes in the target domain.
- Installation: never install or vendor this pack automatically. On explicit request, use `npx skills add typesafe-ai/skills --skill typesafe-ai`; report it as an external skill when used.

## Verification rules

- Use repo's real commands from package scripts, Makefiles, Go modules, CI config, or README.
- Run focused checks first; broaden when practical.
- Report exact commands run and outcomes.
- Report skipped checks with reasons.
- Never claim green without evidence.
- Do not run deployment, production mutation, destructive database, cluster, or git-changing commands without explicit user approval.

## Final report

After all subagents finish, the lead reports the relevant sections only:

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

Omit impact sections that are not relevant. Never hide a relevant security, database, build/deploy, performance, or release impact merely to shorten the report.

For review-only work, lead with findings ordered by severity.

## Distribution contract

- Keep `.agents/skills/dev-team/` as the editable repo skill.
- Keep `plugins/dev-team/skills/dev-team/` synced for plugin users.
- Keep `.agents/plugins/marketplace.json` pointing at `./plugins/dev-team`.
- Keep custom agents in `.codex/agents/*.toml`.
- The plugin manifest distributes skills, not repo custom-agent TOMLs. A plugin-only installation therefore uses generic fallback unless the host project separately provides compatible custom agents; never claim that installing the plugin alone applied the task-specific model/effort profiles.
- If this skill changes, update both the repo skill and plugin skill before sharing.
