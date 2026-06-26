---
name: frontend-state-check
description: Use when frontend state, data fetching, or forms need review — "check the state handling in this diff", "why is this data stale" — or any change touching stores, queries, forms, or async UI. Covers source of truth, derived vs stored state, races, staleness, optimistic updates, loading/empty/error/success states, validation, and hydration. Not for accessibility (use accessibility-check) or UX quality (use ux-review).
---

# Frontend State Check

Review frontend state, data fetching, forms, and UI states in the current diff or feature.

**Use when:**
- A diff touches state management, data fetching, forms, or async UI and needs a correctness review.
- A bug smells like state: flicker, stale data, double submits, hydration mismatches.

**Not for:**
- Labels, keyboard behavior, focus, ARIA → `accessibility-check`.
- Visual hierarchy, copy, flow quality → `ux-review`.

## Workflow

1. Inspect the repo first: `git diff` for the change under review; `package.json` for the state/data libraries actually in use (React Query/SWR, Redux/Zustand, form libs); `next.config.*` to confirm Next.js. Never assume the stack.
2. Map every piece of state in the diff to its source of truth — server cache, URL, form state, local component, global store — and flag misplacements against the repo's existing conventions.
3. Check derived vs stored state: anything computable from existing state must be derived at render, not copied into `useState`/store and synced via effect.
4. Hunt race conditions: out-of-order responses, setState after unmount, concurrent mutations on the same record, debounce gaps, double submit on slow networks.
5. Check staleness: cache keys and invalidation after each mutation, props copied to state at mount, lists not refreshing after create/delete, refetch-on-focus assumptions.
6. Review optimistic updates: rollback path on failure, reconciliation with the server response, no duplicate entries on retry.
7. Verify each async boundary renders all four UX states — loading, empty, error, success — and that errors surface to the user instead of being swallowed.
8. Review forms: client validation paired with server validation, per-field error display, submit guarded while a request is in flight, dirty-state and reset behavior.
9. If the repo is Next.js: check the server/client boundary — `'use client'` placement, only serializable props crossing it, no `window`/browser APIs in server components, hydration mismatch sources (dates, random values, locale).
10. Suggest verification via the repo's real scripts first (lint/typecheck in `package.json`); `npx tsc --noEmit` only as a generic fallback when no script exists.

## Safety

- Review only — no edits, no state-changing commands.
- Never invent repo-specific commands; quote scripts from `package.json` or the Makefile.
- If a data flow cannot be traced in the files read, mark it unverified instead of guessing library behavior.

## Output format

## State review
Each piece of state with its source of truth; misplacements and derived-vs-stored violations, with file references.

## Edge cases
Unmount during fetch, double submit, stale props, empty data, retry/refresh paths — only ones present in this code.

## Race/staleness risks
Out-of-order responses, missing cache invalidation, optimistic-rollback gaps — each tied to a concrete file/line.

## UX states
Loading / empty / error / success coverage per async boundary, plus form validation gaps.

## Tests needed
State transitions and races worth covering, using the repo's existing test setup and commands.

## Reporting

- Separate verified findings (traced in code) from suspicions (inferred from patterns).
- If a library's caching or revalidation behavior was assumed rather than confirmed in the repo, flag it explicitly.
