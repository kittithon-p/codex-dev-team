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
- The same skill workflow is packaged as a local Codex plugin at `plugins/dev-team/`, exposed through `.agents/plugins/marketplace.json` for teammates who prefer installing it from the Codex plugin directory. The plugin distributes skills only; exact task-specific roles still require compatible `.codex/agents/*.toml` files in the host repo and a runtime spawn surface that can select them.
- This repo vendors the focused and specialist skills used by `$dev-team` in both `.agents/skills/` and `plugins/dev-team/skills/`, so internal teammates can clone or install the plugin and use the workflow without separate skill installs.
- Use bundled specialist skills on demand only. Do not preload all Go, frontend, 9arm, git/release, security, testing, or ops skills unless the task needs them.
- Prefer `$dev-team [task]` over `/dev-team`; Codex explicit skill invocation uses `$skill-name`.
- The dev team is runtime-only. Codex spawns subagents for the current task and waits for their results. Mark completed turns as completed/runtime-managed; claim an explicit thread close only when the runtime exposes and successfully performs that operation.
- Do not use a team for single-file or tightly-coupled edits; use one agent or the main session instead.
- Prefer independent read-heavy subagent work; parallel agents consume additional tokens, and write-heavy work must keep disjoint ownership.
- Keep project limits at `agents.max_threads = 6` and `agents.max_depth = 1` unless a measured need justifies a reviewed change.
- Good `$dev-team` prompts include Goal, Context, Constraints, and Done when. If these are missing, infer conservative defaults and ask only when the missing detail changes the implementation risk.
- The lead AGENT must orchestrate: inspect, plan, spawn relevant subagents, assign file ownership, wait for results, review/verify, then report the final status.
- For multiple tasks, track each item (`T1`, `T2`, ...) and run independent work in parallel only when file/repo ownership does not overlap.
- Execpolicy `.rules` control out-of-sandbox command handling; they never authorize commit, push, merge, deploy, release, or external mutation without the user's explicit request.
- Do not persist model, reasoning, service-tier, or Fast mode changes from a dev-team run unless the user explicitly asks for that configuration change.
- When running in Antigravity (agy), use the `dev-team-agy` skill. Before writing a plan summary or setting up `/goal`, the lead MUST run a grill-me session (`ask_question`) to eliminate unverified assumptions, clarify requirements, and align decisions with the user.
- The team enforces Ponytail simplification (`.agents/skills/ponytail/`): writers follow the Ponytail ladder (YAGNI, stdlib first, native platform features, zero unrequested abstractions), and review includes `ponytail-review` alongside `scrutinize`.

## shared memory with Claude Code

Codex and Claude Code share one memory store for this workspace, so a fact learned by either is available to both. The directory is exported as `$CLAUDE_MEMORY_DIR` by the `codex-role` wrapper (no absolute path is recorded here, because this file is committed and the location is machine-specific). When that variable is unset, skip this section entirely.

Rules:
- `$CLAUDE_MEMORY_DIR/MEMORY.md` is the index: one line per memory, `- [Title](file.md) — hook`. Read the index first, then open only the specific files the current task needs. There are well over a hundred memory files; never read them all.
- Each memory is one markdown file with frontmatter (`name`, `description`, `metadata.type` of `user` | `feedback` | `project` | `reference`) and links to related memories as `[[other-name]]`.
- Memories are **point-in-time observations, not live state**. A memory that cites a file, line, function, flag, or MR number may be stale. Verify against the current code before acting on it, and say so when a memory turns out to be wrong.
- Read access needs no extra permission: the read-only sandbox already spans the disk. Write access is granted only to the `worker` role, and only to this directory.
- Write lane, to keep the store from being corrupted by two writers: create new memories as `codex-<short-kebab-slug>.md`. Do not edit a memory file you did not create, and do not edit `MEMORY.md` — Claude Code owns the index and links new `codex-*.md` files into it.
- Before writing a new memory, grep the directory for an existing file covering the same fact and extend that instead of adding a near-duplicate.
- Do not record secrets, tokens, passwords, or live credential values in a memory file, even though source code and diffs may be shared freely.
- Do not record what the repo already makes obvious (code structure, git history, things stated in `AGENTS.md` or a repo `CLAUDE.md`). Record the non-obvious: a decision and its reason, a trap that cost real time, a constraint that is not visible in the code.
