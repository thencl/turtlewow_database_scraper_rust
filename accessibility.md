# Accessibility

This document defines the accessibility standards, implementation requirements, and audit process for this project. Accessibility is not a post-launch concern — it is a first-class requirement built in from the start. Every UI change must be reviewed against this document before it is considered done.

---

## Philosophy

Every person who uses this application deserves a full, functional experience regardless of how they interact with it. That includes people who navigate by keyboard, use screen readers, have low vision, have motor impairments, or are in environments where a mouse is impractical. Accessibility features also improve usability for everyone — not just users with disabilities.

---

## Compliance Target

- **Standard**: WCAG 2.1 Level AA
- **ARIA practices**: Follow the WAI-ARIA Authoring Practices Guide for all interactive patterns.
- **Keyboard standard**: Every interactive element reachable and operable without a pointing device.
- **Screen reader target**: VoiceOver (macOS/iOS), NVDA (Windows), Orca (Linux).

---

## Color and Visual Design

### Contrast
- Normal text (below 18pt / 14pt bold): minimum contrast ratio **4.5:1** against background.
- Large text (18pt+ or 14pt+ bold): minimum contrast ratio **3:1** against background.
- UI components and graphical elements (icons, borders, focus indicators): minimum **3:1**.
- Never rely on color alone to convey meaning — always pair color with a label, icon, or pattern.

### Typography
- Use relative units (`rem`, `em`) for font sizes — never `px` for base text.
- Line height minimum **1.5** for body text.
- Letter spacing must not be set tighter than the default.
- Text must remain legible when browser zoom is set to 200%.

### Focus Indicators
- Every interactive element must have a visible, high-contrast focus ring.
- Do not use `outline: none` without providing an equivalent custom focus style.
- Focus style must be clearly distinguishable from the element's default and hover states.
- Minimum focus indicator size: 2px solid outline with at least 3:1 contrast against adjacent colors.

### Motion and Animation
- All non-essential animations must respect the `prefers-reduced-motion` media query.
- Animations that loop or auto-play must be stoppable by the user.
- No content may flash more than 3 times per second.

---

## Keyboard Navigation

- **Tab order** must follow the logical visual reading order of the page.
- **All interactive elements** — buttons, links, inputs, selects, custom controls — must be reachable by Tab.
- **All interactive elements** must be activatable by keyboard (Enter or Space for buttons; Enter for links).
- **Modal dialogs** must trap focus inside while open and return focus to the trigger element when closed.
- **Menus and dropdowns** must support arrow key navigation within the open menu.
- **Escape key** must close any modal, dropdown, tooltip, or overlay.
- **Skip navigation link** must be present and visible on focus for any page with repeated navigation.

---

## Semantic HTML

Use the correct HTML element for every purpose. Do not use a `<div>` or `<span>` where a semantic element exists.

| Purpose | Correct element |
|---|---|
| Primary page heading | `<h1>` |
| Section headings | `<h2>` through `<h6>` in logical order |
| Navigation regions | `<nav>` with a distinct `aria-label` |
| Main content area | `<main>` |
| Complementary content | `<aside>` |
| Buttons that trigger actions | `<button>` |
| Links that navigate | `<a href="...">` |
| Form inputs | `<input>`, `<select>`, `<textarea>` with associated `<label>` |
| Lists of items | `<ul>` / `<ol>` / `<li>` |
| Data tables | `<table>` with `<th scope="...">` headers |

---

## ARIA

Use ARIA only when native HTML semantics are insufficient.

### Required ARIA Attributes
- Every `<button>` that contains only an icon must have `aria-label`.
- Every custom interactive widget must have an appropriate `role`.
- Dynamic content regions that update without a page reload must use `aria-live` (polite for non-urgent, assertive for critical alerts only).
- Every modal must have `role="dialog"`, `aria-modal="true"`, and `aria-labelledby` pointing to the dialog title.
- Loading states must be announced: use `aria-busy="true"` on the loading container and update when complete.
- Expanded/collapsed states must use `aria-expanded`.
- Disabled states must use the `disabled` attribute (preferred) or `aria-disabled="true"`.

### ARIA Anti-Patterns to Avoid
- Do not add `role="button"` to a `<div>` — use `<button>` instead.
- Do not use `aria-label` to override visible text — they should match.
- Do not use `aria-hidden="true"` on content that is visually visible and meaningful.
- Do not nest interactive elements (e.g. a button inside a link).

---

## Forms and Inputs

- Every input must have a visible, persistent label — not just placeholder text.
- Placeholder text must not be the only label. Placeholder disappears on input and has low contrast by default.
- Required fields must be marked visually and with `aria-required="true"`.
- Validation errors must be:
  - Displayed as text adjacent to the field, not only by color change.
  - Associated with the input using `aria-describedby`.
  - Announced to screen readers when they appear.
- Success confirmations must be announced via `aria-live`.

---

## Images and Media

- Every meaningful image must have descriptive `alt` text.
- Decorative images must have `alt=""` so screen readers skip them.
- Icons used as buttons must have text labels or `aria-label`.
- Video content must have captions.
- Audio content must have a transcript.

---

## Testing and Auditing

### Automated Testing (baseline — not sufficient alone)
Run these on every significant UI change:

```bash
# Lighthouse accessibility audit
npx lighthouse http://localhost:3000 --only-categories=accessibility

# axe-core in test suite
# Install: npm install --save-dev @axe-core/react
# Usage: import and render in test environment — violations throw
```

### Manual Testing (required for every UI feature)
- [ ] Tab through the entire feature using only the keyboard. Every element reachable and operable.
- [ ] Activate all controls using only Enter and Space.
- [ ] Open the feature with VoiceOver (macOS) or NVDA (Windows) and navigate using screen reader shortcuts.
- [ ] Verify all dynamic content changes are announced correctly.
- [ ] Test at 200% browser zoom — no content clipped or overlapping.
- [ ] Test with `prefers-reduced-motion` enabled — no distracting animations.
- [ ] Test with Windows High Contrast mode or macOS Increase Contrast.

### Audit Frequency
- After every major UI feature addition or change.
- Before every pre-PR review.
- Before every production deployment.
- At least once per sprint on the full application.

---

## Accessibility Checklist (pre-PR)

- [ ] All interactive elements reachable by keyboard in logical order.
- [ ] All interactive elements operable by keyboard (Enter / Space).
- [ ] Visible focus styles present and high-contrast.
- [ ] No color-only indicators used.
- [ ] Contrast ratios meet WCAG 2.1 AA for all text and UI components.
- [ ] All images have meaningful alt text or `alt=""` for decorative images.
- [ ] All form inputs have persistent visible labels.
- [ ] Error messages are text, adjacent to the field, and announced.
- [ ] All ARIA roles and labels are correct and non-redundant.
- [ ] Dynamic content updates are announced via `aria-live`.
- [ ] Modal dialogs trap focus and return focus on close.
- [ ] Lighthouse accessibility score is 90 or above.
- [ ] No critical or serious axe-core violations.

---

## Reporting Accessibility Issues

Log accessibility issues as bugs, not feature requests. Severity:
- **Critical**: A user cannot complete a core task at all (e.g. a modal that cannot be closed by keyboard).
- **Serious**: A user experiences significant difficulty completing a task.
- **Moderate**: A user experiences inconvenience but can still complete the task.
- **Minor**: A small deviation from best practice with minimal user impact.

Critical and serious accessibility bugs are P0 — they block shipping.
