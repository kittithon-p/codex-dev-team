---
name: plan-feature
description: Plan a feature or cross-cutting change before implementation. Use when the user asks to build, add, refactor, or change behavior across multiple files.
argument-hint: [feature-description]
---

# Plan Feature

Feature request:
$ARGUMENTS

Instructions:
1. Inspect the workspace structure and relevant files. If `graphify-out/graph.json` exists, start with `graphify query "<feature topic>"`.
2. Identify the likely frontend, backend (service/BFF/worker), test, config, and documentation impact. Remember each affected directory is its own git repo.
3. For cross-service features, pin the contract FIRST: BFF `process/gateway/*` type <-> service `system/model` wire type (field names, snake_case, optionality).
4. Produce a concise implementation plan.
5. Include:
   - goal
   - assumptions
   - repos and files likely to change
   - task breakdown (split by repo, one owner per repo)
   - file ownership
   - test strategy (testify/mockery for Go, vitest for frontend)
   - risks (DocumentDB operator limits, shared-lib serialization, env differences local/uat/prod)
6. Do not edit files during this skill unless the user explicitly asks to implement.
