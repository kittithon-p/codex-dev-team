---
name: grilling
description: Interview the user in rounds until every decision behind a request is settled, so nothing is silently assumed. Used by the dev-team lead before planning, or when the user asks to be grilled or to stress-test a plan, decision, or idea.
---

# grilling

Interview the user until you share one understanding of the request. Nothing the team would otherwise guess stays unasked.

## Design tree

Map the request as a tree of decisions: each decision branches into the decisions that depend on it. The **frontier** is every open decision whose prerequisites are already settled, meaning the questions you can ask now without guessing at answers you have not heard.

## Rounds

- Ask the whole frontier in one round. Every question is multiple choice: 2-4 concrete, mutually exclusive options, the recommended one first and marked `(Recommended)`, each with its trade-off. The user may always answer outside the options.
- If the runtime offers a structured choice tool (e.g. `request_user_input`), use it. Otherwise use the text format below and ask the user to reply like `Q1: A, Q2: B`.
- Wait for the user's answers, then recompute the frontier and ask the next round.
- A question that depends on another question still open in this round belongs to a later round.
- No emojis. Text format:

```
**Q1 - <title>**: <question>
  A) <option> (Recommended) - <trade-off>
  B) <option> - <trade-off>
  C) <option> - <trade-off>

**Q2 - <title>**: ...
  A) ...
```

## Facts vs decisions

- **Facts are yours to find.** Never ask the user for something a file, command, or tool can answer: ports, existing types, current behavior, which repo owns what. Look it up read-only, or hand it to a read-only explorer subagent.
- Do not block on a lookup. Only the questions downstream of a running lookup wait for it; ask the rest of the frontier now.
- **Decisions belong to the user**: scope, intent, priority, behavior, contract shape, rollout. Put each one to them and wait.

## Done

You are done when the frontier is empty: every branch visited, nothing silently assumed, and the user has confirmed you share one understanding. Do not act before that confirmation. Hand the result on as a `Settled decisions` list. Anything the user left unanswered is listed as `Open - blocked on user`, never assumed.
