---
name: changelog-entry
description: Use when a change needs a changelog entry — "write a changelog entry", "add this to the changelog", or when a landed feature/fix needs user-visible release notes. Covers matching the repo's existing changelog format, keep-a-changelog categories, breaking-change call-outs, and migration notes — never invents version numbers. Not for full documentation updates (use update-docs) or release preparation (use release-checklist).
argument-hint: [change to describe]
---

# Changelog Entry

Prepare a changelog entry without inventing versions for: $ARGUMENTS

**Use when:**
- A merged or pending change needs an entry in the repo's changelog convention.
- The user asks what the changelog should say for a specific feature, fix, or breaking change.

**Not for:**
- Updating READMEs, API docs, or setup docs → `update-docs`.
- Pre-release checks, tagging, or version-bump decisions → `release-checklist`.

## Workflow

1. Find the changelog first — `CHANGELOG.md`, `CHANGES.md`, `docs/changelog*`, `.changeset/` (JS/Next.js repos), `.goreleaser.*` or release-note steps in CI workflows (Go repos). If none exists, say so and propose keep-a-changelog before writing anything.
2. Read the last 2–3 released sections and copy the format exactly: heading levels, date format, category names, bullet style, PR/issue link style. The repo's own format wins over keep-a-changelog defaults.
3. Identify what actually changed from the code, not the summary alone — `git diff` / `git log --oneline` against the base branch (use the repo's real scripts first if it generates changelogs, e.g. a changesets or release script in `package.json`/`Makefile`).
4. Classify each change into the repo's categories (keep-a-changelog default: Added / Changed / Deprecated / Removed / Fixed / Security).
5. Write each bullet as user-visible impact — what changed for whom (end users, API consumers, operators), not implementation detail ("refactored handler" is not an entry).
6. Mark breaking changes explicitly under Changed and write migration steps for each one.
7. Place the entry under `Unreleased` (or the repo's equivalent). NEVER invent a version number or release date — if there is no Unreleased section, ask which version the entry targets.

## Safety

- Drafting only by default — no edits; present the entry and only write to the changelog file if the user explicitly asks, touching that file alone.
- Never fabricate version numbers, dates, or PR/issue references — use real refs from git or omit them.
- Do not list changes you could not confirm in the diff/commits — flag them as unverified instead.

## Output format

## Changelog entry
The full entry formatted exactly as it should appear in the repo's changelog, under Unreleased or the confirmed target version.

### Added
New user-facing capabilities — one bullet per feature, phrased for the audience that gains it.

### Changed
Behavior changes, with breaking changes marked explicitly (e.g. **BREAKING:**).

### Fixed
Bug fixes phrased as the symptom users no longer hit, not the internal cause.

### Security
Security-relevant fixes or hardening; omit detail that helps exploit unpatched versions.

### Migration notes
Concrete steps users/operators must take (config, env vars, schema, API calls) — "None" if drop-in.

## Reporting

- State which changelog file and format was matched, naming the released section mirrored.
- Separate entries verified from the diff/commits from entries based only on the user's description.
- If the target version is unknown, place the entry under Unreleased and ask before assigning any version.
