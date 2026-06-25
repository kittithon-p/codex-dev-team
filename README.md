# Codex Dev Team

Reusable Codex setup for running a lead AGENT with repo-specific subagents.

Use this when you want Codex to receive a task, inspect the repo, plan the work,
delegate independent parts to subagents, wait for the subagents to finish, review
and verify the result, then report back.

## What's Included

- `AGENTS.md` - repo-level Codex instructions.
- `.agents/skills/dev-team/` - the `$dev-team` skill.
- `.codex/agents/` - custom Codex subagents such as `architect-planner`,
  `test-qa`, `code-reviewer`, `frontend-next`, `bff-go`, `system-service-go`,
  `worker-go`, and reviewers.
- `plugins/dev-team/` - installable Codex plugin package.
- `.agents/plugins/marketplace.json` - repo marketplace entry for the plugin.
- `.codex/config.toml` - shared agent concurrency defaults.

## Quick Start

Clone this repo:

```bash
git clone https://github.com/kittithon-p/codex-dev-team.git
cd codex-dev-team
codex
```

Then invoke the skill:

```text
$dev-team แก้ bug login แล้วให้ verify/review ให้ครบ
```

## How To Use In Another Repo

Copy these paths into the root of the repo or workspace where Codex will work:

```text
AGENTS.md
.agents/skills/dev-team/
.codex/agents/
.codex/config.toml
```

Then start Codex from that repo root and invoke:

```text
$dev-team <feature-or-bug-or-worklist>
```

For plugin-based usage, also copy:

```text
.agents/plugins/marketplace.json
plugins/dev-team/
```

Restart Codex, open the plugin directory, and install `Dev Team` from the repo
marketplace.

## Prompt Shape

Good `$dev-team` prompts include:

- Goal: what should change or be investigated.
- Context: repo, files, errors, PR, logs, examples, or systems involved.
- Constraints: safety, architecture, branch, API, release, or ownership rules.
- Done when: tests/checks, behavior, report, or review evidence required.

Example:

```text
$dev-team
Goal: add payment status filter to backoffice listing.
Context: frontend backoffice page, BFF listing endpoint, service query.
Constraints: keep API backward compatible, no deploy.
Done when: focused tests pass, code-reviewer reports no blocking findings.
```

## How It Works

The main Codex session acts as the lead AGENT:

1. Inspect real repo files and detect the stack.
2. Create a `Dev-team plan`.
3. Select only relevant subagents.
4. Assign file ownership and forbidden files.
5. Spawn subagents for independent work.
6. Wait for all subagents to report back.
7. Use `test-qa` for verification and `code-reviewer` for review when code changes.
8. Report final status, changed files, checks, risks, and follow-ups.

Parallel work is supported when ownership does not overlap. Shared libraries,
migrations, release steps, and cross-service contracts are serialized.

## Safety Rules

- Do not commit, push, deploy, merge, tag, release, or mutate external systems
  unless explicitly requested.
- Do not let two subagents edit the same file or repo area.
- Do not claim checks passed unless they actually ran.
- Do not put secrets in `.codex/config.toml` before sharing.
- Keep real tokens, private keys, and `.env*` files out of Git.

## Updating

When changing the dev-team workflow, update both copies:

```text
.agents/skills/dev-team/SKILL.md
plugins/dev-team/skills/dev-team/SKILL.md
```

Then validate:

```bash
python3 /Users/kitti/.codex/skills/.system/skill-creator/scripts/quick_validate.py .agents/skills/dev-team
python3 /Users/kitti/.codex/skills/.system/skill-creator/scripts/quick_validate.py plugins/dev-team/skills/dev-team
python3 /Users/kitti/.codex/skills/.system/plugin-creator/scripts/validate_plugin.py plugins/dev-team
```

