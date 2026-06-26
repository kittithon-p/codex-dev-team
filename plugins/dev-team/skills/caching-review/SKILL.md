---
name: caching-review
description: Use when caching behavior needs review — "review this caching", "is this cache safe", or any diff adding a cache, TTL, revalidate option, or Cache-Control header. Covers cache keys, invalidation, staleness, HTTP/framework/DB cache layers, and user-scoped data leaking into shared caches. Not for performance evidence (use performance-audit) or DB-level query analysis (use query-performance-check).
---

# Caching Review

Review the caching behavior in the current diff or context: every cache needs three answers — what is the key, when is it invalidated, and what happens when it is stale.

**Use when:**
- A diff adds or changes a cache — in-memory map, Redis, HTTP headers, framework cache, CDN config.
- Stale or wrong data is suspected and the cache layers need a walkthrough.

**Not for:**
- Measuring whether caching actually helps (latency, profiling evidence) → `performance-audit`.
- Slow queries, indexes, N+1 at the database level → `query-performance-check`.

## Workflow

1. Detect the cache surface from real files first — `next.config.*` and `fetch`/`revalidate`/`unstable_cache` usage, Redis/memcached clients in `package.json`/`go.mod`, middleware setting `Cache-Control`, in-process maps/LRU libs, CDN config in the repo. Never assume.
2. Read the current diff (`git diff`) plus the read/write paths around each cache it touches — where entries are written and every path that mutates the underlying data.
3. Check each cache key for completeness: does it include every dimension that changes the value (user/tenant, locale, version, query params, feature flags)? Can two different values collide on one key?
4. Demand the invalidation story — every cache needs one: TTL, explicit purge on write, event-driven, or revalidate tag. Find write paths that mutate data without touching the cache.
5. Bound staleness per cache: the max age a stale value can survive, and whether the product tolerates it (prices, permissions, inventory usually do not).
6. Check security/privacy: user-scoped or auth-dependent responses in shared caches — `Cache-Control: public`, CDN edge, or in-process maps keyed without a user id; cached authorization decisions.
7. Review HTTP caching headers: `Cache-Control` directives, `ETag`/`Last-Modified`, and `Vary` (especially on `Authorization`/`Cookie`) with proxy/CDN behavior in mind.
8. If the repo is Next.js: check fetch caching defaults, `revalidate`/`cache` options, route segment config, and whether `revalidateTag`/`revalidatePath` covers every mutation path.
9. Map DB/application cache layers (e.g. Redis in front of MongoDB, in-process memoization) and check coherence — which layer wins on disagreement, and how each gets purged.

## Safety

- Review only — no edits, no cache flushes, no state-changing commands.
- Never print cached values that may contain tokens, sessions, or PII.
- If a cache's invalidation story cannot be found in code, report that as a finding — do not assume a TTL exists.

## Output format

## Caching review
One-paragraph verdict: what was reviewed and the overall risk level.

## Cache boundaries
Each cache found: layer, key composition, missing key dimensions, collision risks.

## Invalidation risks
Per cache: the invalidation story (or its absence) and write paths that bypass it.

## Stale data risks
Max staleness per cache and whether the product tolerates it.

## Security/privacy risks
User-scoped data in shared caches, auth-dependent responses cached publicly, missing `Vary`.

## Recommendations
Concrete fixes ordered by risk: key fixes, TTLs, invalidation hooks, header changes.

## Reporting

- Separate verified facts (read from code/diff) from assumptions (inferred CDN/framework defaults).
- If a layer could not be inspected (CDN or infra config outside the repo), say so and state what is needed to confirm it.
