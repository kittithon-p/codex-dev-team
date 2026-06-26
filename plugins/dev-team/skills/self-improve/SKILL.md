---
name: self-improve
description: Run a session retro that distills durable lessons into this workspace's auto-memory + AGENTS.md + skills, so each session makes the next one smarter. Use at the end of a substantial session, or when the user says "self-improve", "retro", "สรุปบทเรียน", "เรียนรู้จากงานนี้", or "อัปเดต memory". Human-approved writes only.
---

# Self-improve (workspace retro loop)

Closes the feedback loop the self-improving-agent skill describes, but wired to THIS workspace's real memory system instead of a parallel JSON store (don't fragment memory).

## Where learnings go (in priority order)
1. **Auto-memory** (`~/.Codex/projects/.../memory/*.md` + `MEMORY.md` index) — durable, non-obvious, project-specific facts. This is the primary store. Follow the existing frontmatter format (`name` / `description` / `metadata.type: reference|project|feedback|user`) and add a one-line pointer to `MEMORY.md`.
2. **AGENTS.md** (root) — only cross-cutting rules/pitfalls that every session/agent must obey. **Human-approved only** — propose the diff, do not self-edit silently (per the guide's rule).
3. **A skill** (`.Codex/skills/<name>/SKILL.md`) — only when a RECURRING multi-step workflow emerges (not for one-off facts).
4. **An agent** (`.Codex/agents/*.md`) — only when a recurring review/role gap emerges.

## Process
1. **Extract** from the session: what was learned, what went wrong, what the user corrected, any new convention/port/contract/gotcha discovered.
2. **Filter (the quality gate):** keep only items that are (a) durable (not one-off), (b) non-obvious (not already derivable from code/AGENTS.md), (c) project-specific. Drop the rest. Distill, don't dump — `MEMORY.md` loads every session.
3. **Dedup** against existing memory (read `MEMORY.md` first); UPDATE an existing entry rather than create a near-duplicate.
4. **Propose** the concrete writes (new/updated memory files, MEMORY.md lines, any AGENTS.md/skill change) and get a quick yes before writing AGENTS.md or skills. Memory writes can proceed if clearly durable.
5. **Apply + report** what changed, as clickable links. No emojis.

## Anti-patterns
- Over-generalizing from one experience.
- Saving what the repo already records (structure, past fixes, git history).
- Letting `MEMORY.md` bloat — prune/merge stale entries when you touch the area.
- Auto-editing AGENTS.md without approval.

## On errors / corrections
When the user corrects guidance or a command fails after following a rule, capture the correction as a `feedback`-type memory (with **Why:** + **How to apply:**) so the mistake isn't repeated. Verify the fix before asserting it.

Related: the cross-cutting rules live in [[first-principles]] (`.Codex/rules/`); the backend reference is the `nayoo-service-map` skill.
