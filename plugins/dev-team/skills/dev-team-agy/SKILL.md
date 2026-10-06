---
name: dev-team-agy
description: Antigravity (agy) version of dev-team. Orchestrate a team of agy subagents (define_subagent + invoke_subagent) for Nayoo feature work, bug fixes, refactors, investigations, reviews, security/DevOps/release tasks, or parallel work items. Enforces a mandatory pre-plan grill-me phase (ask_question) before writing plans, defining goals, or executing /goal to eliminate unverified assumptions. Use this instead of `dev-team` whenever running in Antigravity. Do not use for single-file or tightly-coupled tasks where one agent is enough.
---

# dev-team-agy — Antigravity subagent orchestration

The main agy session is the **lead**. It inspects, plans, defines and invokes only the relevant subagents, assigns ownership, waits for results, verifies, and reports.

This file contains **only what differs from Codex**. Everything runtime-agnostic is owned by the Codex skill and MUST be followed from there:

| Topic | Read from [dev-team/SKILL.md](../dev-team/SKILL.md) |
|---|---|
| Goal / Context / Constraints / Done-when | `## Prompt contract` |
| Lead may not edit while a team is active; approval gates for commit/push/merge/deploy/release | `## Lead contract` |
| Inspect → Classify → Plan → … → Report (steps 1–5, 7–11) | `## Orchestration lifecycle` |
| Plan format | `## Planning output` |
| Batch tracking (`T1`, `T2`, …) | `## Parallel and batch work` |
| Which role for which job, bug state machine (`debugger` → `bug-fixer`) | `## Agent selection`, routing rules under `## Task model and effort routing` |
| One owner per file set, BFF ↔ service contract first | `## Contract and ownership rules` |
| Spawn prompt shape | `## Spawn prompt template` |
| Skill/9arm/superpowers routing, verification, final report | the matching sections |

Ignore Codex-only material there: `.codex/config.toml`, `agents.max_threads`, execpolicy `.rules`, `$dev-team` syntax, `/agent`, `followup_task`, `interrupt_agent`, sandbox flags, GPT-5.6 model/effort pins.

## Runtime facts (agy)

- Subagent model is chosen per invocation via `invoke_subagent` → `Model`. Only four values exist: `inherit` (same model as the lead), `pro`, `flash`, `flash_lite`. There is **no reasoning-effort knob**.
- Roles are not files. The lead creates them at runtime with `define_subagent`; a definition lasts for the conversation, so define each role once and reuse it.
- Read-only is enforced mechanically by `enable_write_tools=false` (no edit/run tools), not by prompt.
- Never set `enable_subagent_tools=true` → depth stays 1; subagents cannot spawn subagents.
- At most **6** subagents running concurrently. The cap is a ceiling, not a target.

## Role source of truth

Role prompts live in `.codex/agents/<role>.toml` (`developer_instructions`). Do not copy them into this skill.

If that file does not exist (e.g. installed via the `dev-team` plugin only, which ships no TOMLs), define the role with a short scoped `system_prompt` written from that role's entry in `## Agent selection` of the Codex skill plus the runtime adapter block, and report the role as `generic fallback` in the final `### Model profile`.

To define a role:
1. `view_file` the TOML.
2. `define_subagent` with:
   - `name`: `agy-<role>` (e.g. `agy-code-mapper`)
   - `description`: the TOML `description`
   - `system_prompt`: the TOML `developer_instructions` verbatim, followed by the **runtime adapter** block below
   - `enable_write_tools`: from the table below (derived from TOML `sandbox_mode`)
3. `invoke_subagent` with `TypeName=agy-<role>`, the `Model` from the table, and the `Workspace` from the workspace rules.

Runtime adapter block (append to every system_prompt):

> You are running inside Antigravity, not Codex. Ignore any instruction about Codex sandbox modes, `followup_task`, `interrupt_agent`, `/agent`, or GPT model/effort tiers. You cannot spawn subagents. If blocked, stop and return an escalation packet (goal, evidence with absolute path:line, what was tried and exact results, the single question that would unblock you) to the lead. Report back with `send_message`-style distilled findings: changed files, exact commands and results, blockers, residual risk.

## Model and permission map

| Role (`.codex/agents/*.toml`) | agy `Model` | `enable_write_tools` |
|---|---|---|
| `code-mapper`, `docs-researcher` | `flash` | false |
| `debugger`, `browser-debugger` | `flash` | false |
| `crud-generator`, `test-author`, `test-qa`, `release-mr` | `flash` | true |
| `bug-fixer` | `pro` | true |
| `backend-dev`, `bff-go`, `system-service-go`, `go-common-libs`, `worker-go`, `frontend-dev`, `frontend-next`, `devops` | `pro` | true |
| `code-reviewer` | `pro` | false |
| `architect-planner`, `api-design-reviewer`, `integration-reviewer`, `security-reviewer` | `inherit` | false |
| `adviser` (escalation step 1) | `inherit` | false |

Not used in agy: `adviser-ultra` (needs an effort tier agy cannot express), `codex-worker` (Codex-backed writer; only if the user explicitly asks). `codex-adviser` is replaced by the lead calling the wrapper directly (see Escalation).

`flash_lite` is not used by default. Downgrade a role only if the user asks to save tokens.

When reporting, state the **requested** `Model` alias per subagent. agy does not expose which concrete model backs `pro`/`flash`; never claim one.

## Workspace rules

- Read-only roles → `Workspace=inherit`.
- Exactly one writer active → `Workspace=inherit`.
- Two or more writers active at the same time → each writer gets `Workspace=share` (own branch, shared repo storage). The lead reviews and merges their branches after all writers finish; still assign disjoint file ownership so merges are trivial.
- This workspace is multi-repo: state the repo(s) each writer owns in its prompt.
- Never use `Workspace=branch` unless the user asks (slow, duplicates storage).

## Coordination tools (Codex → agy)

| Codex | agy |
|---|---|
| spawn custom agent | `define_subagent` (once) + `invoke_subagent` |
| `send_message` to running turn | `send_message` |
| `followup_task` on idle agent | `send_message` to that conversation ID |
| `interrupt_agent` | `manage_subagents` `kill` (only when work is wrong, unsafe, or obsolete) |
| `/agent` inspection | `manage_subagents` `list` |
| wait | stop calling tools; agy wakes the lead on each message. Optionally `schedule` with `TimerCondition=any` as a liveness guard. Do not poll. |

## Escalation (when an owner is blocked)

1. **agy adviser** — invoke `agy-adviser` (`Model=inherit`, read-only) with the owner's escalation packet. Route the answer back to the owner via `send_message`.
2. **Codex cross-family consult** — only if step 1 explicitly could not resolve it. The lead (not a subagent) runs:
   ```bash
   cat > /tmp/codex-adviser-<slug>.md <<'BRIEF'
   <self-contained brief: forced choice, evidence with absolute path:line, what was tried + exact results, competing positions, constraints, unverified items labelled ASSUMPTION:, and the STOP-if-missing-context clause>
   BRIEF
   ~/.claude/codex-roles/codex-role adviser /tmp/codex-adviser-<slug>.md <workdir>
   ```
   Trust the answer only if the output ends with `codex-role: VERIFIED model=gpt-5.6-sol effort=xhigh sandbox=read-only`. Anything else → report the consult as untrustworthy and stop at `blocked-needs-adviser`.
3. Still blocked → report blocked state to the user with evidence and the one action needed.

Never run steps 1 and 2 in parallel for the same blocker.

## Mandatory Pre-Plan Grill-Me Phase (ลดความคิดไปเอง / Zero Assumptions)

ก่อนที่ Lead จะเขียนสรุปว่าจะทำอะไร (Planning, Scope summary) หรือกำหนดเป้าหมายในโหมด `/goal` **ต้องทำขั้นตอน Grill-Me ร่วมกับผู้ใช้ก่อนเสมอ**:

1. **Inspect Codebase First**: สำรวจโค้ด, architecture, config, หรือ evidence ที่เกี่ยวข้องให้ครบถ้วนก่อน เพื่อตัดคำถามที่มีคำตอบชัดเจนในโค้ดอยู่แล้วออก ไม่ถามเรื่องพื้นฐานที่หาดูเองได้
2. **Interactive Grill-Me via `ask_question`**:
   - ใช้เครื่องมือ `ask_question` สัมภาษณ์ผู้ใช้ในประเด็นสำคัญทีละข้อ (หรือชุดคำถามที่เกี่ยวข้องกัน):
     - Requirements, edge cases, และ business logic ที่คลุมเครือ
     - Architectural trade-offs และ design decisions
     - ขอบเขตที่ต้องทำ (In-scope) vs สิ่งที่ไม่ควรแตะ (Out-of-scope)
     - Acceptance criteria, Done-when condition, และมาตรการ rollback/verification
   - ทุกคำถามต้องมีตัวเลือกที่แนะนำ `(Recommended)` เป็นตัวเลือกแรกเสมอ
3. **Eliminate Assumptions**: ห้าม "คิดไปเอง" หรือ assume requirement, contract หรือ user preference เองเด็ดขาด หากจุดใดมีความเสี่ยงหรือส่งผลกระทบ ต้องถามให้ได้ข้อสรุปที่แท้จริง
4. **Formulate Plan & /goal**:
   - หลังจากได้คำตอบชัดเจนจากผู้ใช้ผ่าน grill-me แล้วเท่านั้น จึงนำข้อสรุปและ constraints มาเขียน `## Dev-team plan` หรือสรุปเป้าหมาย `/goal`
   - หากทำงานในโหมด `/goal` (long-running): สรุป Goal และ Done-when criteria ให้ชัดเจนตามที่ได้ตกลงจากการ grill-me รัน verification checklist ให้ครบถ้วนอย่างละเอียดก่อนจบ และรายงานผลพร้อม tag `<!-- GOAL_COMPLETE -->`

## Lifecycle deltas

Follow the Codex `## Orchestration lifecycle`, with these mandatory modifications:
- **Step 2.5 (Grill-Me — Mandatory before Plan & Goal)**: ก่อน Step 3 (Plan and approve) หรือก่อนเขียนสรุปว่าจะทำอะไร / กำหนดเป้าหมาย `/goal` ต้องเรียก `ask_question` เพื่อทำ Grill-Me ขจัดความคลุมเครือและลดความคิดไปเองตามข้อกำหนดข้างต้น
- **Step 3 (Plan and approve)**: สรุปแผน `## Dev-team plan` โดยอิงผลลัพธ์จากข้อตกลงที่ได้จากการ Grill-Me ไม่ตั้งสมมติฐานขึ้นเอง
- **Step 4 (Select)**: pick roles from the map above; define only those.
- **Step 6 (Spawn)**: `invoke_subagent` with explicit `Model` and `Workspace`; batch independent subagents in one call. ผู้รับผิดชอบเขียนโค้ด (implementation owners เช่น `bug-fixer`, domain dev) ให้ใช้หลักการ **`ponytail`** (The ladder: YAGNI, standard library ก่อน, native platform features ก่อน, no unrequested abstractions, minimal code).
- **Step 8 (Collect)**: wait for every subagent's message before synthesis.
- **Step 10 (Review with Scrutinize & Ponytail)**: การรีวิวด้วย `code-reviewer` / `scrutinize` ร่วมกับ `ponytail-review` **ต้องทำอย่างน้อย 2 รอบ และไม่เกิน 5 รอบ**:
  - **Round 1 (Initial Review & Simplification)**: วิพากษ์เจตนา ค้นหาทางเลือกที่เรียบง่ายกว่า (simpler alternative ตาม Ponytail ladder) ตรวจจับความซับซ้อนส่วนเกิน (removable complexity / bloat) และ trace code path เชิงลึก
  - **Remediation**: ส่ง findings กลับไปให้ implementation owner แก้ไข
  - **Round 2 (Mandatory Re-Verification)**: ตรวจสอบซ้ำหลังการแก้ไข แม้รอบแรกจะผ่าน รอบสองต้องทำ adversarial stress-test เพื่อหา edge cases และ regressions แอบแฝง
  - **Rounds 3–5 (Iterative Fix & Review)**: รันซ้ำเฉพาะเมื่อยังมีข้อบกพร่องระดับ major/blocker
  - **Hard Cap ที่รอบ 5**: หากครบ 5 รอบแล้วยังตกลงกันไม่ได้หรือยังไม่ผ่าน ให้หยุดลูปทันทีและ escalate ขึ้นมาให้ Lead / User ตัดสิน
- **Step 11 (Report)**: include a `### Model profile` line per subagent: `agy-<role> — Model=<alias>, write=<true|false>, Workspace=<mode>`. หากทำงานภายใต้ `/goal` ให้รวม verification checklist และแท็ก `<!-- GOAL_COMPLETE -->` เมื่อเสร็จสมบูรณ์
