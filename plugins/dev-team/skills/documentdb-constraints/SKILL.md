---
name: documentdb-constraints
description: Use BEFORE writing or editing any MongoDB/mongo-driver aggregation pipeline, query, or index in this workspace's Go services/workers. The backend DB is AWS DocumentDB, NOT real MongoDB — several common operators are rejected at runtime. Trigger on aggregation/$group/$facet/$lookup work, slow-query/count tuning, or new pipeline design.
---

# DocumentDB constraints (this workspace is NOT MongoDB)

The backend datastore is **AWS DocumentDB**, accessed via `go.mongodb.org/mongo-driver`. It is wire-compatible with MongoDB but is a different engine — pipelines that work locally against MongoDB can fail in UAT/prod. Engine version differs by environment: **5.0** on some envs, **8.0** on the Bangkok (ap-southeast-7) env. Always check the AWS DocumentDB compatibility matrix for the target env before using a new operator.

## Operators that are NOT supported — do not use
- `$facet`
- `$graphLookup`
- `$sortByCount`
- `$bucketAuto` (and assume other rarely-used stages are unsupported until verified)

If you need `$facet`-style multi-aggregation, run separate pipelines and combine app-side. If you need `$sortByCount`, use `$group` + `$sort` explicitly.

## Performance: count() aggregations are the known slowness
The documented UAT slowness is NOT hardware — it is frequent `count()`-style aggregations on large collections (e.g. `uat.posts`, 1–1.8s each; IXSCAN walks 20k+ index keys). Prefer:
- App-side **cached counters** over per-request `count()`.
- Confirm queries use an index (IXSCAN, not COLLSCAN) and that the index is selective.

## Before shipping a pipeline
1. Confirm every stage/operator is on the AWS DocumentDB supported list for the target engine version.
2. Avoid per-request counts; cache or precompute.
3. If profiling, slow queries land in CloudWatch `/aws/docdb/docdb-nonprod/profiler` (not `system.profile`) on the 8.0 env.

This skill encodes a recurring real bug — "works on my Mongo, rejected by DocDB." When in doubt, treat an operator as unsupported and verify.
