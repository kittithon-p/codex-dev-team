## graphify

This project can use a knowledge graph at `graphify-out/` when one has been generated.

When the user types `/graphify`, invoke the `skill` tool with `skill: "graphify"` before doing anything else.

Rules:
- For codebase questions, first run `graphify query "<question>"` when graphify-out/graph.json exists. Use `graphify path "<A>" "<B>"` for relationships and `graphify explain "<concept>"` for focused concepts. These return a scoped subgraph, usually much smaller than GRAPH_REPORT.md or raw grep output.
- Dirty graphify-out/ files are expected after hooks or incremental updates; dirty graph files are not a reason to skip graphify. Only skip graphify if the task is about stale or incorrect graph output, or the user explicitly says not to use it.
- If graphify-out/wiki/index.md exists, use it for broad navigation instead of raw source browsing.
- Read graphify-out/GRAPH_REPORT.md only for broad architecture review or when query/path/explain do not surface enough context.
- After modifying code, run `graphify update .` only when `graphify-out/graph.json` already exists or the task explicitly asks to create/update the graph.

## dev-team

Use `$dev-team [feature-or-bug]` when a task benefits from Codex subagents working in parallel, such as cross-repo features, multi-layer reviews, or cross-service bug investigations.

Rules:
- `$dev-team` is a repo skill in `.agents/skills/dev-team/`; commit that folder so other teammates can use the same workflow.
- The same workflow is packaged as a local Codex plugin at `plugins/dev-team/`, exposed through `.agents/plugins/marketplace.json` for teammates who prefer installing it from the Codex plugin directory.
- This repo vendors the focused and specialist skills used by `$dev-team` in both `.agents/skills/` and `plugins/dev-team/skills/`, so internal teammates can clone or install the plugin and use the workflow without separate skill installs.
- Use bundled specialist skills on demand only. Do not preload all Go, frontend, 9arm, git/release, security, testing, or ops skills unless the task needs them.
- Prefer `$dev-team [task]` over `/dev-team`; Codex explicit skill invocation uses `$skill-name`.
- The dev team is runtime-only. Codex spawns subagents for the current task, waits for their results, and then closes completed agent threads.
- Do not use a team for single-file or tightly-coupled edits; use one agent or the main session instead.
- Prefer independent read-heavy subagent work; parallel agents consume additional tokens, and write-heavy work must keep disjoint ownership.
- Keep project limits at `agents.max_threads = 6` and `agents.max_depth = 1` unless a measured need justifies a reviewed change.
- Good `$dev-team` prompts include Goal, Context, Constraints, and Done when. If these are missing, infer conservative defaults and ask only when the missing detail changes the implementation risk.
- The lead AGENT must orchestrate: inspect, plan, spawn relevant subagents, assign file ownership, wait for results, review/verify, then report the final status.
- For multiple tasks, track each item (`T1`, `T2`, ...) and run independent work in parallel only when file/repo ownership does not overlap.
- Execpolicy `.rules` control out-of-sandbox command handling; they never authorize commit, push, merge, deploy, release, or external mutation without the user's explicit request.
- Do not persist model, reasoning, service-tier, or Fast mode changes from a dev-team run unless the user explicitly asks for that configuration change.
