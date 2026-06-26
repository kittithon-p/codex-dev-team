---
name: bundle-size-check
description: Use when frontend changes may bloat the client bundle — "did this dependency bloat the bundle", "check bundle impact of this diff". Covers new dependency size justification, heavy/full-library imports, lazy-loading opportunities, tree-shaking blockers, server/client boundary, and route-level impact. Not for broader performance review (use performance-audit) or state logic (use frontend-state-check).
---

# Bundle Size Check

Review the frontend bundle impact of the current diff or described change.

**Use when:**
- A diff adds dependencies, imports, or components that ship to the client.
- A bundle budget, slow page load, or "why did the JS grow" question points at recent changes.

**Not for:**
- Broader performance review (rendering, queries, caching) → `performance-audit`.
- State management and data-flow logic → `frontend-state-check`.

## Workflow

1. Inspect the repo first — `package.json` (dependencies + scripts), lockfile diff, `next.config.*` / bundler config, existing analyzer setup (`@next/bundle-analyzer`, `source-map-explorer`, size-limit config). Never assume the stack or its build commands.
2. Get the change set: `git diff` (or the described change) — list every new dependency and every new/changed import in client-bound code.
3. For each new dependency: check approximate minified+gzip size, tree-shakeability (ESM vs CJS), and whether a lighter alternative or existing dependency already covers it. Demand a justification proportional to its size.
4. Flag heavy imports: full-library imports (`import _ from 'lodash'`, moment-style monoliths, whole icon packs) where named/per-path imports or smaller substitutes work.
5. Flag tree-shaking blockers: CJS-only packages, `sideEffects` mismatches, barrel files re-exporting everything, dynamic `require`.
6. Check the server/client boundary: code that should stay server-side leaking into the client — in Next.js, server-only logic pulled into `"use client"` components; secrets/SDKs/DB clients imported from shared modules.
7. Find lazy-loading opportunities: route-level code splitting, `dynamic import()` / `next/dynamic` for below-the-fold or conditional components, deferring heavy libs (charts, editors, maps) until used.
8. Estimate route-level impact: which routes/pages gain weight, and whether shared chunks grow for every route.
9. Verify with the repo's real scripts first (`package.json` scripts like `analyze`, `build`, size-limit). If none exist, suggest the generic ecosystem command and mark it as a suggestion, not a repo command.

## Safety

- Review only — no edits, no dependency installs, no config changes.
- Run only read-only or build/analyze commands that the repo's own scripts define; never invent repo-specific commands.
- If a build is expensive or cannot run here, report static findings and give the exact command for the user to run.

## Output format

## Bundle impact
One-paragraph verdict: net effect of the change on client bundle size, affected routes, and overall severity.

## New dependencies
Each added package: approximate size, tree-shakeability, justification status, lighter alternative if one exists.

## Heavy imports
Full-library imports, moment-style monoliths, tree-shaking blockers, and server code leaking client-side — with file:line and the fix.

## Lazy-loading opportunities
Routes/components/libraries to load on demand (dynamic import, route-level split), ranked by estimated KB saved.

## Build/check commands
The repo's real analyze/build/size commands run or recommended, with results if run.

## Reporting

- Separate measured numbers (analyzer/build output) from estimates (package registry size, inference from imports) — label each.
- If size could not be measured (no analyzer, build not run), say so explicitly and list the exact command to confirm.
- Flag any client/server boundary call as "needs confirmation" when the bundler config could not be inspected.
