---
name: handoff
description: Write (or update) a session handoff doc so cross-repo work can be resumed later or picked up by a teammate/agent. Trigger on "write a handoff", "save context for later", "hand this off", or wrapping up a multi-repo task mid-flight.
---

# Handoff

Produce a concise, resumable handoff for the current work. Save to `docs/handoffs/handoff_<YYYY-MM-DD>_<slug>.md` (use today's date; derive `<slug>` from the argument or the task). If a handoff for this topic already exists today, UPDATE it (append to "Work done", don't rewrite history).

Keep it under ~500 words, plain text, no emojis. The workspace ROOT is not a git repo — report per-repo state with `git -C <repo>`.

Structure:
- **Goal** — one or two sentences.
- **Repos in play** — for each touched repo: path, current branch, dirty/clean, last relevant commit.
- **Contracts in flight** — any BFF gateway type <-> service `system/model` wire shape being changed (the top cross-service pitfall); note both sides + whether agreed.
- **Work done** — append-only bullets with `file:line` refs.
- **Next steps** — the immediate next 1-3 actions, each with its verify step.
- **Open questions / blockers** — anything ambiguous or waiting on a human.
- **Verify** — the exact commands to confirm done (per repo: `go build/vet/gofmt -l/go test`, or `tsc/eslint/vitest`).

Do not commit the handoff or any code unless explicitly asked. End by printing the handoff file path as a clickable link.
