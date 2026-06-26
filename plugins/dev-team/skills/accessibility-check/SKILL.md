---
name: accessibility-check
description: Use when UI changes need an accessibility review — "check accessibility", "a11y review of this diff", "is this component accessible". Covers semantic HTML, labels, keyboard navigation, focus management, ARIA, forms, error and loading states, alt text, reduced motion, and contrast where inferable from code. Not for flow/copy quality (use ux-review) or state logic (use frontend-state-check).
---

# Accessibility Check

Review the accessibility of the UI changes in the current diff or context.

**Use when:**
- A diff touches components, pages, forms, dialogs, or navigation and needs an a11y pass.
- A new interactive pattern (modal, menu, combobox, toast, carousel) was built or modified.

**Not for:**
- Flow, copy, or overall UX quality → `ux-review`.
- State logic, data fetching, or render correctness → `frontend-state-check`.

## Workflow

1. Detect the frontend stack from real files first — `package.json` + lockfile, `tsconfig.json`, `next.config.*`, Tailwind/design-token config, and a11y-relevant deps (Radix, Headless UI, React Aria, `eslint-plugin-jsx-a11y`, axe). Never assume.
2. Scope the review to the changed UI files (`git diff`, or the files the user pointed at) plus the shared primitives they render — read each one fully before judging it.
3. Semantics: native `button`/`a`/`label`/`fieldset` over `div` + `onClick`; heading levels descend without skipping; landmarks where the diff adds page structure.
4. Names and alt text: every input has an associated label (placeholder is not a label); icon-only buttons get `aria-label`; informative images get descriptive `alt`, decorative ones `alt=""`.
5. Keyboard: everything interactive is Tab-reachable in reading order; Enter/Space/Escape/arrow keys handled where the pattern requires; no keyboard traps.
6. Focus: no removed focus styles without a visible replacement; focus moves into dialogs on open and returns to the trigger on close; focus lands sensibly after route changes and async updates.
7. ARIA only when native semantics are insufficient — flag redundant or wrong roles; verify state attributes (`aria-expanded`, `aria-invalid`, `aria-current`) are bound to real component state, not hardcoded.
8. Forms and feedback: errors tied to fields via `aria-describedby` and announced (`role="alert"`/`aria-live`); loading states exposed programmatically (`aria-busy`, live status text), not spinner-only.
9. Motion and contrast: non-trivial animation respects `prefers-reduced-motion`; flag contrast risks where tokens/classes make them inferable (e.g., light-gray text on white) and defer the rest to manual checks.

For the full checklist, read references/checklist.md when doing a thorough review.

## Safety

- Review only — no edits, no state-changing commands.
- Mark every finding as code-verified or needs-visual/manual confirmation; never present an inference as fact.
- Do not flag library-internal ARIA (Radix, Headless UI, React Aria ship their own); review only the props and wiring the repo adds.
- Run a11y lint only via the repo's real scripts if they exist (`jsx-a11y`, axe in tests); never invent commands.

## Output format

## Accessibility findings
Concrete issues found in the changed code:
| Issue | Severity | File | Why it matters | Recommended fix |

## Keyboard/focus checks
Each interaction traced through the code — tab order, key handlers, traps, focus on open/close/async — with pass/fail and the evidence line.

## Manual checks
What code alone cannot prove — actual contrast ratios, screen-reader announcement order, 200% zoom reflow — each with a one-line "how to test".

## Reporting

- Separate code-verified findings (file read, line cited) from inferred ones.
- If a shared primitive, token file, or generated style could not be inspected, list it under **Manual checks** with what is needed to confirm.
- Rate severity by user impact: blocker (unusable by keyboard/screen reader), major (degraded), minor (polish).
