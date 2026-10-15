---
name: antislop-accessibility
description: "Accessibility skill for antislop (WCAG 2.2, strict). Load for ANY UI work, not only audits: semantic HTML, accessible names, cursor and interaction affordances, keyboard, focus, forms, ARIA, motion. Enforces R-39."
allowed-tools: Bash(npx *) Read Write Edit Glob Grep
---
# antislop-accessibility

> Anti Slop: Rules for AI Coding Agents. Native web-quality skill

> Part of the antislop system. Read together with `antislop.md` (the core). This skill deep-dives the accessibility concern in full WCAG 2.2 depth. `antislop-human` stays the home of contrast and the contrast checker; this skill covers everything else a person needs to perceive, operate, and understand the UI. Adapted from the MIT-licensed `accessibility` skill of [web-quality-skills](https://github.com/addyosmani/web-quality-skills) by Addy Osmani.

## How to use this skill

- **Native, not optional.** Load this skill together with the core for every UI or web task, not only when someone asks for an audit. Its governing core rules (R-25, R-26, R-32, R-39) are Hard Gate: a FAIL blocks delivery.
- Build it right the first time. The guidance below is how the page is written, not a checklist bolted on at the end.
- Every PASS in the core Delivery Gate needs evidence: a command run, a trace read, an element inspected. Never claim a score or a metric that was not measured (R-36).
- When this skill and the visual skills disagree, function wins: an accessible, fast page with a plain look beats a beautiful page that excludes people or stalls.

Comprehensive accessibility guidelines based on WCAG 2.2 and Lighthouse accessibility audits. Goal: make content usable by everyone, including people with disabilities.

## Interaction Affordances (antislop strict layer)

This layer is antislop's own addition and is enforced by R-39. It covers the small signals that tell a person "this can be used", for mouse, touch, keyboard, and assistive tech at once. A UI that looks right but hides these signals is unfinished.

### Clickable Without a Pointer Cursor

- **Tell:** a `<button>`, a `<summary>`, a `<label>` for a checkbox, a custom `[role="button"]`, a clickable card, or a styled `<select>` that keeps the arrow cursor. Common after a CSS reset (Tailwind preflight resets `button` to `cursor: default`).
- **Why:** the cursor is the first affordance a mouse user reads. An arrow over a control says "this is text", so people hesitate or never click. It reads as a template that was styled but never used.
- **Fix:** every enabled interactive element gets `cursor: pointer`, set once in the base layer, not per component. Disabled controls get `cursor: not-allowed`. Text inputs keep `cursor: text`; drag handles get `grab` / `grabbing`. Never put `cursor: pointer` on something that does nothing (R-26).

```css
/* Base layer: one place, every interactive element */
a[href], button, summary, select, label[for],
[role="button"], [role="link"], [role="tab"], [role="menuitem"], [role="option"], [role="switch"],
input[type="checkbox"], input[type="radio"], input[type="file"],
input[type="submit"], input[type="button"], input[type="reset"], input[type="range"], input[type="color"] {
  cursor: pointer;
}
:disabled, [aria-disabled="true"] { cursor: not-allowed; }
```

### Div or Span Pretending to Be a Control

- **Tell:** `<div onClick>`, `<span onClick>`, or `<a>` without `href` used as a button; `<a href="#" onClick>` used for an action.
- **Why:** it is invisible to the keyboard and to screen readers: no focus, no Enter/Space, no role. A whole group of people cannot press it.
- **Fix:** an action is a `<button type="button">`, a navigation is an `<a href>` to a real destination. Only when neither is possible: add `role`, `tabindex="0"`, Enter and Space handlers, and the state attributes (`aria-pressed`, `aria-expanded`).

### Icon-Only Control With No Name

- **Tell:** a close, menu, search, or social button that contains only an SVG or icon font, with no text, `aria-label`, or visually hidden label.
- **Why:** a screen reader announces "button" and nothing else. Voice-control users cannot say its name.
- **Fix:** give it an accessible name (`aria-label="Close dialog"` or a `.sr-only` span), mark the icon `aria-hidden="true"`, and add a visible tooltip only if it helps sighted users too. Use the same word the visible UI uses (WCAG 2.5.3).

### No Hover, Active, or Disabled State

- **Tell:** buttons and links that look identical at rest, on hover, while pressed, and when disabled; a disabled button that is only greyed out and still clickable.
- **Why:** state feedback tells people their action registered. Without it a click feels lost and a disabled control looks broken.
- **Fix:** distinct `:hover`, `:active`, `:focus-visible`, and disabled styles, each not dependent on color alone. Prefer the `disabled` attribute; if a disabled control must stay focusable for a tooltip, use `aria-disabled="true"` and block the action in code.

### Hit Area Smaller Than the Target

- **Tell:** a 16px icon button with no padding, inline links packed so tight that a thumb hits the neighbor, a checkbox whose label is not clickable.
- **Why:** small targets exclude people with tremor, low precision, or a phone in one hand (WCAG 2.5.8).
- **Fix:** at least 24x24 CSS px with spacing everywhere, and 44x44 for primary touch controls (R-03). Wrap checkboxes and radios in their `<label>` so the text is part of the target.

### Motion That Ignores the User Setting

- **Tell:** parallax, auto-playing carousels, scroll-jacking, or large entrance animations that still run under `prefers-reduced-motion: reduce`.
- **Why:** motion can trigger nausea and vestibular disorders, and an auto-advancing carousel moves content away from slow readers (WCAG 2.2.2, 2.3.3).
- **Fix:** wrap non-essential motion in `@media (prefers-reduced-motion: no-preference)`, and give anything that moves for more than 5 seconds a visible pause control.

### Missing Page-Level Semantics

- **Tell:** no `lang` on `<html>`, no `<main>`, headings chosen for size instead of order, a generic `<title>` on every page, no skip link on a page with a long navigation.
- **Why:** screen-reader users navigate by landmarks, headings, and the page title; without them the page is one long undifferentiated stream, read in the wrong language voice.
- **Fix:** `<html lang>`, one `<main>`, `<header>`/`<nav>`/`<footer>` landmarks, one `<h1>` and no skipped levels, a unique descriptive `<title>`, and a "Skip to content" link as the first focusable element.

### Form Field Without a Real Label

- **Tell:** placeholder used as the only label, a label not tied to its input, errors shown only as a red border, required fields marked only with a red asterisk.
- **Why:** the placeholder disappears on typing and is low contrast; unlabeled inputs are announced as "edit text"; color-only errors are invisible to many.
- **Fix:** a visible `<label for>` on every field, `autocomplete` tokens for personal data, `aria-invalid` plus an error message linked with `aria-describedby`, and the word "required" (or `required` attribute) next to the asterisk. Move focus to the first error on submit.

### Accessibility Affordance Checklist

Run with the core Delivery Gate (R-39). All answers must be **yes**:

- [ ] Does every enabled interactive element show `cursor: pointer` (and disabled ones `not-allowed`), set once in the base layer?
- [ ] Is every action a `<button>` and every navigation an `<a href>`, with no clickable `div`/`span`?
- [ ] Does every icon-only control have an accessible name, with the icon `aria-hidden`?
- [ ] Do controls have distinct hover, active, focus-visible, and disabled states, not color only?
- [ ] Is every target at least 24x24 CSS px (44x44 for primary touch controls)?
- [ ] Does all non-essential motion stop under `prefers-reduced-motion: reduce`?
- [ ] Are `lang`, `<title>`, landmarks, heading order, and a skip link in place?
- [ ] Does every form field have a visible label, `autocomplete` where relevant, and text errors linked to the field?

## Evidence-led audit workflow

When a rendered page is available:

1. Run a live Lighthouse Accessibility audit when that capability is available; with Chrome DevTools MCP, use `lighthouse_audit`. Use mobile navigation mode for a general public page or snapshot mode when reloading would lose authenticated or user-created state.
2. Use failed audit nodes to localize the relevant component or template instead of searching the whole repository for generic patterns.
3. Inspect a rendered accessibility-tree snapshot for names, roles, states, landmarks, and heading structure; with Chrome DevTools MCP, use `take_snapshot`. Exercise the affected flow with the keyboard.
4. Fix the source, then re-run the same audit and manual interaction.

If the live tools are unavailable, use Lighthouse CLI or axe for automated coverage and complete the same manual checks. Automated tools detect only a subset of accessibility barriers: a score of 100 is not WCAG conformance, and a low score does not replace issue-level evidence.

## WCAG Principles: POUR

| Principle | Description |
|-----------|-------------|
| **P**erceivable | Content can be perceived through different senses |
| **O**perable | Interface can be operated by all users |
| **U**nderstandable | Content and interface are understandable |
| **R**obust | Content works with assistive technologies |

## Conformance levels

| Level | Requirement | Target |
|-------|-------------|--------|
| **A** | Minimum accessibility | Must pass |
| **AA** | Standard compliance | Should pass (legal requirement in many jurisdictions) |
| **AAA** | Enhanced accessibility | Nice to have |

---

## Perceivable

### Text alternatives (1.1)

**Images require alt text:**
```html
<!-- ❌ Missing alt -->
<img src="chart.png">

<!-- ✅ Descriptive alt -->
<img src="chart.png" alt="Bar chart showing 40% increase in Q3 sales">

<!-- ✅ Decorative image (empty alt) -->
<img src="decorative-border.png" alt="" role="presentation">

<!-- ✅ Complex image with longer description -->
<figure>
  <img src="infographic.png" alt="2024 market trends infographic" 
       aria-describedby="infographic-desc">
  <figcaption id="infographic-desc">
    <!-- Detailed description -->
  </figcaption>
</figure>
```

**Icon buttons need accessible names:**
```html
<!-- ❌ No accessible name -->
<button><svg><!-- menu icon --></svg></button>

<!-- ✅ Using aria-label -->
<button aria-label="Open menu">
  <svg aria-hidden="true"><!-- menu icon --></svg>
</button>

<!-- ✅ Using visually hidden text -->
<button>
  <svg aria-hidden="true"><!-- menu icon --></svg>
  <span class="visually-hidden">Open menu</span>
</button>
```

**Visually hidden class:**
```css
.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

### Color contrast (1.4.3, 1.4.6)

| Text Size | AA minimum | AAA enhanced |
|-----------|------------|--------------|
| Normal text (< 18px / < 14px bold) | 4.5:1 | 7:1 |
| Large text (≥ 18px / ≥ 14px bold) | 3:1 | 4.5:1 |
| UI components & graphics | 3:1 | 3:1 |

```css
/* ❌ Low contrast (2.5:1) */
.low-contrast {
  color: #999;
  background: #fff;
}

/* ✅ Sufficient contrast (7:1) */
.high-contrast {
  color: #333;
  background: #fff;
}

/* ✅ Focus states need contrast too (3:1 against background, WCAG 1.4.11) */
:focus-visible {
  outline: 2px solid currentColor;
  outline-offset: 2px;
}
```

**Don't rely on color alone:**
```html
<!-- ❌ Only color indicates error -->
<input class="error-border">
<style>.error-border { border-color: red; }</style>

<!-- ✅ Color + icon + text -->
<div class="field-error">
  <input aria-invalid="true" aria-describedby="email-error">
  <span id="email-error" class="error-message">
    <svg aria-hidden="true"><!-- error icon --></svg>
    Please enter a valid email address
  </span>
</div>
```

### Media alternatives (1.2)

```html
<!-- Video with captions -->
<video controls>
  <source src="video.mp4" type="video/mp4">
  <track kind="captions" src="captions.vtt" srclang="en" label="English" default>
  <track kind="descriptions" src="descriptions.vtt" srclang="en" label="Descriptions">
</video>

<!-- Audio with transcript -->
<audio controls>
  <source src="podcast.mp3" type="audio/mp3">
</audio>
<details>
  <summary>Transcript</summary>
  <p>Full transcript text...</p>
</details>
```

---

## Operable

### Keyboard accessible (2.1)

**All functionality must be keyboard accessible.** Prefer native interactive elements, `<button>`, `<a href>`, and form controls handle Enter/Space activation, focus, and assistive-tech semantics for free. Only add manual keyboard handling when you cannot use a native element.

```html
<!-- ❌ Non-interactive element with click only: not focusable, no keyboard activation -->
<div class="card" onclick="handleAction()">Open</div>

<!-- ✅ Best: use a native button -->
<button type="button" onclick="handleAction()">Open</button>
```

```javascript
// ✅ When you MUST use a non-interactive element (e.g. div with role="button"),
// make it focusable AND handle keyboard activation. Do NOT add this to a native
// <button>, Enter/Space already fire click, so you'd double-trigger.
element.setAttribute('role', 'button');
element.setAttribute('tabindex', '0');
element.addEventListener('click', handleAction);
element.addEventListener('keydown', (e) => {
  if (e.key === 'Enter' || e.key === ' ') {
    e.preventDefault();
    handleAction();
  }
});
```

**No keyboard traps.** Users must be able to Tab into and out of every component. Use the [modal focus trap pattern](references/A11Y-PATTERNS.md#modal-focus-trap) for dialogs, the native `<dialog>` element handles this automatically.

### Focus visible (2.4.7)

```css
/* ❌ Never remove focus outlines */
*:focus { outline: none; }

/* ✅ Use :focus-visible for keyboard-only focus */
:focus {
  outline: none;
}

:focus-visible {
  outline: 2px solid currentColor; /* inherits text color → already contrast-checked */
  outline-offset: 2px;
}

/* ✅ Or pick a brand color and verify ≥3:1 contrast against every background it lands on */
button:focus-visible {
  box-shadow: 0 0 0 3px rgba(0, 95, 204, 0.5);
}
```

### Focus not obscured (2.4.11), new in 2.2

When an element receives keyboard focus, it must not be entirely hidden by other author-created content such as sticky headers, footers, or overlapping panels. At Level AAA (2.4.12), no part of the focused element may be hidden.

```css
/* ✅ Account for sticky headers when scrolling to focused elements */
:target {
  scroll-margin-top: 80px;
}

/* ✅ Ensure focused items clear fixed/sticky bars */
:focus {
  scroll-margin-top: 80px;
  scroll-margin-bottom: 60px;
}
```

### Skip links (2.4.1)

Provide a skip link so keyboard users can bypass repetitive navigation. See the [skip link pattern](references/A11Y-PATTERNS.md#skip-link) for full markup and styles.

### Target size (2.5.8), new in 2.2

Interactive targets must be at least **24 × 24 CSS pixels** (AA). Exceptions: inline text links, elements where the browser controls the size, and targets where a 24px circle centered on the bounding box does not overlap another target.

```css
/* ✅ Minimum target size */
button,
[role="button"],
input[type="checkbox"] + label,
input[type="radio"] + label {
  min-width: 24px;
  min-height: 24px;
}

/* ✅ Comfortable target size (recommended 44×44) */
.touch-target {
  min-width: 44px;
  min-height: 44px;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}
```

### Dragging movements (2.5.7), new in 2.2

Any action that requires dragging must have a single-pointer alternative (e.g., buttons, inputs). See the [dragging movements pattern](references/A11Y-PATTERNS.md#dragging-movements) for a sortable-list example.

### Timing (2.2)

```javascript
// Allow users to extend time limits
function showSessionWarning() {
  const modal = createModal({
    title: 'Session Expiring',
    content: 'Your session will expire in 2 minutes.',
    actions: [
      { label: 'Extend session', action: extendSession },
      { label: 'Log out', action: logout }
    ],
    timeout: 120000
  });
}
```

### Motion (2.3)

```css
/* Respect reduced motion preference */
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

---

## Understandable

### Page language (3.1.1)

```html
<!-- ❌ No language specified -->
<html>

<!-- ✅ Language specified -->
<html lang="en">

<!-- ✅ Language changes within page -->
<p>The French word for hello is <span lang="fr">bonjour</span>.</p>
```

### Consistent navigation (3.2.3)

```html
<!-- Navigation should be consistent across pages -->
<nav aria-label="Main">
  <ul>
    <li><a href="/" aria-current="page">Home</a></li>
    <li><a href="/products">Products</a></li>
    <li><a href="/about">About</a></li>
  </ul>
</nav>
```

### Consistent help (3.2.6), new in 2.2

If a help mechanism (contact info, chat widget, FAQ link, self-help option) is repeated across multiple pages, it must appear in the **same relative order** each time. Users who rely on consistent placement shouldn't have to hunt for help on every page.

### Form labels (3.3.2)

Every input needs a programmatically associated label. See the [form labels pattern](references/A11Y-PATTERNS.md#form-labels) for explicit, implicit, and instructional examples.

### Error handling (3.3.1, 3.3.3)

Announce errors to screen readers with `role="alert"` or `aria-live`, set `aria-invalid="true"` on invalid fields, and focus the first error on submit. See the [error handling pattern](references/A11Y-PATTERNS.md#error-handling) for full markup and JS.

### Redundant entry (3.3.7), new in 2.2

Don't force users to re-enter information they already provided in the same session. Auto-populate from earlier steps, or let users select from previously entered values. Exceptions: security re-confirmation and content that has expired.

```html
<!-- ✅ Auto-fill shipping address from billing -->
<fieldset>
  <legend>Shipping address</legend>
  <label>
    <input type="checkbox" id="same-as-billing" checked>
    Same as billing address
  </label>
  <!-- Fields auto-populated when checked -->
</fieldset>
```

### Accessible authentication (3.3.8), new in 2.2

Login flows must not rely on cognitive function tests (e.g., remembering a password, solving a puzzle) unless at least one of:
- A copy-paste or autofill mechanism is available
- An alternative method exists (e.g., passkey, SSO, email link)
- The test uses object recognition or personal content (AA only; AAA removes this exception)

```html
<!-- ✅ Allow paste in password fields -->
<input type="password" id="password" autocomplete="current-password">

<!-- ✅ Offer passwordless alternatives -->
<button type="button">Sign in with passkey</button>
<button type="button">Email me a login link</button>
```

---

## Robust

### ARIA usage (4.1.2)

**Prefer native elements:**
```html
<!-- ❌ ARIA role on div -->
<div role="button" tabindex="0">Click me</div>

<!-- ✅ Native button -->
<button>Click me</button>

<!-- ❌ ARIA checkbox -->
<div role="checkbox" aria-checked="false">Option</div>

<!-- ✅ Native checkbox -->
<label><input type="checkbox"> Option</label>
```

**When ARIA is needed,** use the correct roles and states. See the [ARIA tabs pattern](references/A11Y-PATTERNS.md#aria-tabs) for a complete tablist example.

### Live regions (4.1.3)

Use `aria-live` regions to announce dynamic content changes without moving focus. See the [live regions pattern](references/A11Y-PATTERNS.md#live-regions-and-notifications) for markup and a `showNotification()` helper.

---

## Testing checklist

### Automated testing

Prefer a live Lighthouse audit that returns failing rendered nodes directly to the agent. With Chrome DevTools MCP, this is `lighthouse_audit`. Otherwise:

```bash
# Lighthouse accessibility audit
npx lighthouse https://example.com --only-categories=accessibility

# axe-core
npm install @axe-core/cli -g
axe https://example.com
```

### Manual testing

- [ ] **Keyboard navigation:** Tab through entire page, use Enter/Space to activate
- [ ] **Screen reader:** Test with VoiceOver (Mac), NVDA (Windows), or TalkBack (Android)
- [ ] **Zoom:** Content usable at 200% zoom
- [ ] **High contrast:** Test with Windows High Contrast Mode
- [ ] **Reduced motion:** Test with `prefers-reduced-motion: reduce`
- [ ] **Focus order:** Logical and follows visual order
- [ ] **Target size:** Interactive elements meet 24×24px minimum

See the [screen reader commands reference](references/A11Y-PATTERNS.md#screen-reader-commands) for VoiceOver and NVDA shortcuts.

---

## Common issues by impact

### Critical (fix immediately)
1. Missing form labels
2. Missing image alt text
3. Insufficient color contrast
4. Keyboard traps
5. No focus indicators

### Serious (fix before launch)
1. Missing page language
2. Missing heading structure
3. Non-descriptive link text
4. Auto-playing media
5. Missing skip links

### Moderate (fix soon)
1. Missing ARIA labels on icons
2. Inconsistent navigation
3. Missing error identification
4. Timing without controls
5. Missing landmark regions

## References

- [WCAG 2.2 Quick Reference](https://www.w3.org/WAI/WCAG22/quickref/)
- [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)
- [Deque axe Rules](https://dequeuniversity.com/rules/axe/)
- [Web Quality Audit](../web-quality-audit/SKILL.md)
- [WCAG criteria reference](references/WCAG.md)
- [Accessibility code patterns](references/A11Y-PATTERNS.md)
