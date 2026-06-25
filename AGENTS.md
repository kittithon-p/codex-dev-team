## graphify

This project has a knowledge graph at graphify-out/ with god nodes, community structure, and cross-file relationships.

When the user types `/graphify`, invoke the `skill` tool with `skill: "graphify"` before doing anything else.

Rules:
- For codebase questions, first run `graphify query "<question>"` when graphify-out/graph.json exists. Use `graphify path "<A>" "<B>"` for relationships and `graphify explain "<concept>"` for focused concepts. These return a scoped subgraph, usually much smaller than GRAPH_REPORT.md or raw grep output.
- Dirty graphify-out/ files are expected after hooks or incremental updates; dirty graph files are not a reason to skip graphify. Only skip graphify if the task is about stale or incorrect graph output, or the user explicitly says not to use it.
- If graphify-out/wiki/index.md exists, use it for broad navigation instead of raw source browsing.
- Read graphify-out/GRAPH_REPORT.md only for broad architecture review or when query/path/explain do not surface enough context.
- After modifying code, run `graphify update .` to keep the graph current (AST-only, no API cost).

## dev-team

Use `$dev-team [feature-or-bug]` when a task benefits from Codex subagents working in parallel, such as cross-repo features, multi-layer reviews, or cross-service bug investigations.

Rules:
- `$dev-team` is a repo skill in `.agents/skills/dev-team/`; commit that folder so other teammates can use the same workflow.
- The same workflow is packaged as a local Codex plugin at `plugins/dev-team/`, exposed through `.agents/plugins/marketplace.json` for teammates who prefer installing it from the Codex plugin directory.
- Prefer `$dev-team [task]` over `/dev-team`; Codex explicit skill invocation uses `$skill-name`.
- The dev team is runtime-only. Codex spawns subagents for the current task, waits for their results, and then closes completed agent threads.
- Do not use a team for single-file or tightly-coupled edits; use one agent or the main session instead.
- Good `$dev-team` prompts include Goal, Context, Constraints, and Done when. If these are missing, infer conservative defaults and ask only when the missing detail changes the implementation risk.
- The lead AGENT must orchestrate: inspect, plan, spawn relevant subagents, assign file ownership, wait for results, review/verify, then report the final status.
- For multiple tasks, track each item (`T1`, `T2`, ...) and run independent work in parallel only when file/repo ownership does not overlap.
