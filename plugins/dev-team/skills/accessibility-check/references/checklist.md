# Accessibility checklist

Companion to `accessibility-check`. Every item is checkable from code unless marked (manual).

## Semantic HTML
- Interactive elements are native: `button` for actions, `a href` for navigation — never `div`/`span` with `onClick`.
- Heading levels descend without skipping (h1 → h2 → h3); one h1 per page/view.
- Landmarks present where the diff adds page structure: `main`, `nav`, `header`, `footer`.
- Lists use `ul`/`ol`/`li`; data tables use `table` with `th scope` — not styled divs.

## Labels and accessible names
- Every form control has a programmatic label: `<label htmlFor>`, wrapping `<label>`, or `aria-labelledby` — placeholder is not a label.
- Icon-only buttons and links have `aria-label` or visually-hidden text.
- Accessible name matches or contains the visible text (speech-input users).

## Image alt text
- Informative images: `alt` describes content or function — not "image of...".
- Decorative images: `alt=""`; decorative SVG icons: `aria-hidden="true"`.
- SVG icons inside labeled buttons are `aria-hidden="true"` so the name isn't duplicated.
- Next.js `<Image>`: check `alt` isn't an empty string on meaningful images.

## Keyboard navigation
- All interactions Tab-reachable; tab order follows reading order (no positive `tabindex`).
- Pattern-correct keys: Enter/Space activates; Escape closes dialogs/menus; arrows move within menus, tabs, radio groups, comboboxes.
- No keyboard trap: anything you can tab into, you can tab out of.
- Custom interactive elements (`role="button"`) have `tabindex="0"` and key handlers — or better, become native elements.

## Focus states and management
- No `outline: none` / `focus:outline-none` without a visible replacement (`focus-visible` ring counts).
- Dialog/drawer open: focus moves inside; close: focus returns to the trigger.
- Focus trapped inside open modals — and only modals.
- After async content removes the focused element (delete row, wizard step), focus lands somewhere sensible — not `body`.
- SPA route changes move focus to the new content or announce it.

## ARIA — only when native is insufficient
- No ARIA beats wrong ARIA: prefer a native element before any `role`.
- Redundant roles removed (`role="button"` on `<button>`).
- State attributes track real state: `aria-expanded`, `aria-selected`, `aria-checked`, `aria-current` bound to the component's state variable, not hardcoded.
- `aria-hidden="true"` never on focusable elements or their ancestors.
- Library primitives (Radix, Headless UI, React Aria) ship correct ARIA — review only the props the repo passes.

## Forms
- Related inputs grouped with `fieldset`/`legend` (radio groups especially).
- Required fields marked programmatically (`required` or `aria-required`), not asterisk-only.
- `autocomplete` set on identity fields (name, email, tel, address); input types correct (`email`, `tel`, `number`).
- Failed submission moves focus to the first invalid field or an error summary.

## Error messages
- Errors linked to their field via `aria-describedby`; field marked `aria-invalid="true"`.
- Errors announced on appearance: `role="alert"` or `aria-live="assertive"` on the container.
- Error text says what is wrong and how to fix it — not "invalid input".
- Error state not conveyed by color alone (icon or text alongside the red border).

## Loading states
- Async regions expose state: `aria-busy="true"`, or status text in an `aria-live="polite"` region.
- Pending buttons keep an accessible name ("Saving…"), not a name replaced by a spinner.
- Spinner-only feedback is a finding: a CSS animation announces nothing.
- Skeleton placeholders `aria-hidden` with a live-region alternative.

## Reduced motion
- Non-trivial animation (parallax, auto-play carousels, large transitions) gated on `prefers-reduced-motion` or a `useReducedMotion` hook.
- Tailwind: `motion-safe:`/`motion-reduce:` variants on animated utilities.
- Nothing flashes more than 3 times per second.

## Color and contrast (when inferable from code)
- Flag known-low-contrast combos in tokens/classes: e.g., `text-gray-400` on white, placeholder-colored body text.
- Targets: 4.5:1 body text, 3:1 large text and UI components, 3:1 focus ring against adjacent colors.
- Information never carried by color alone (status dots get text or an icon).
- (manual) Verify actual ratios with a contrast tool against the rendered theme.
