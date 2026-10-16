---
name: antislop-human
description: "Human and accessibility skill for antislop, native and strict (WCAG 2.2). Load for EVERY UI task: contrast, keyboard, focus, pointer cursor and interaction affordances, semantic HTML, accessible names, forms, ARIA, reduced motion. Includes the contrast checker. Enforces R-25, R-32, R-39."
allowed-tools: Bash(python *) Bash(python3 *) Bash(npx *) Read Write Edit Glob Grep
---
# antislop-human

> Anti Slop: Rules for AI Coding Agents. Human skill

> Part of the antislop system. Read together with `antislop.md` (the core). This skill deep-dives the human concern: the UI must stay usable by people with different eyes, hands, and setups. Contrast, keyboard, focus, states, and the mobile details that exclude people.

## How to use this skill

- Load together with `antislop.md` whenever the task builds or edits UI. The core holds the mechanism (the purpose test, the three tiers, the Delivery Gate); this skill holds the human-side depth: the parts of a UI that exclude people with different eyes, hands, and setups.
- Every entry has the same shape: **Tell** (the pattern), **Why** (who it excludes, and why it reads as unfinished), **Fix** (what to do instead), with the governing core rule cited as R-XX.
- Accessibility is not a checklist of extras bolted on at the end. It is part of the core promise that "the UI holds up" (C-4). The Delivery Gate in the core remains the gate; the "Human Skill Checklist" at the end of this file is the supplement to run alongside it.
- This skill is antislop's native accessibility layer: it loads for every UI task, not only on an audit request. It holds contrast, keyboard, focus, the interaction affordances (pointer cursor, real controls, names, states, labels, reduced motion), and the full WCAG 2.2 reference set in `references/`.
- The contrast checker (formula + reference table + script) lives in this skill. Use it for every color pairing you cannot verify by eye.
- This skill keeps only the mobile details that exclude people: zooming, and the on-screen keyboard.

## Color & Contrast

### Low-Contrast Text

- **Tell:** light grey text on a white or near-white background, thin body text, muted labels chosen because they look "elegant" but are hard to read.
- **Why:** it excludes low-vision users and everyone in bright light. It is a visual choice made without checking the standard, which is exactly the kind of default the filter exists to catch.
- **Fix:** meet WCAG AA minimums (R-25): 4.5:1 for normal text, 3:1 for large text (18px+). Compute the ratio; do not eyeball it.

### Text Over a Photo or Gradient

- **Tell:** white text placed directly over an image or gradient that is light in some areas, checked at one bright spot only.
- **Why:** contrast is local. Where the image is light, the text drops below 4.5:1 even if the hero still "looks" fine. R-25 requires testing the whole area the text passes over, not a single point.
- **Fix:** add a scrim or a solid color block behind the text, then verify the worst spot, not the best. If any part of the text area fails, the treatment fails.

### The Grey-on-Grey Hallucination

- **Tell:** "dark grey on black" or "light grey on white" claimed to pass AA without a computation.
- **Why:** this is the most common accessibility hallucination. The eye overestimates contrast on grey pairs, and agents repeat the claim because it sounds plausible. #555555 on black is 2.8:1. It fails.
- **Fix:** never assert a pairing passes. Run the contrast checker (below) or apply the formula. When neither is possible, use the reference table.

### Non-Text Contrast

- **Tell:** interactive components (buttons, icons, input borders, focus indicators, chart segments) distinguished from their background by less than 3:1.
- **Why:** non-text UI carries information by shape and edge. When the edge is a hair of tint, low-vision users cannot find the control. The same bar applies to states like hover and selected.
- **Fix:** give every component boundary and every status indicator a 3:1 ratio against adjacent colors (WCAG 1.4.11). Pair icons with a text label that meets 4.5:1.

### The Contrast Checker

The home of the contrast checker. Three layers, from most to least convenient:

**The script.** When a script runtime is available and the file is present, run it instead of computing by hand. The script ships in this skill's folder (`contrast-check.py`, next to this `SKILL.md`). Run it with `python3` on macOS and Linux, `python` on Windows:

```bash
python3 "${CLAUDE_SKILL_DIR}/contrast-check.py" "#FFFFFF" "#777777"
# normal text: FAIL (4.48 < 4.5)
# large text:  PASS (4.48 >= 3.0)
```

If the `${CLAUDE_SKILL_DIR}` variable is not available in this agent, point the script path at this skill's folder directly. The script exists so agents stop hallucinating AA. It takes two hex colors and prints the ratio and the verdict for both text sizes. If the file is missing, the formula and table below are complete on their own. Never block on the script.

**The formula (WCAG 2.x).**

1. Contrast ratio = (L1 + 0.05) / (L2 + 0.05), where L1 is the lighter relative luminance and L2 the darker.
2. Relative luminance L of one color: convert each channel to 0-1 (`c = hex / 255`), then linearize: if `c <= 0.03928`, `c_lin = c / 12.92`; otherwise `c_lin = ((c + 0.055) / 1.055)^2.4`.
3. `L = 0.2126*R + 0.7152*G + 0.0722*B`.
4. Compare the unrounded ratio: 4.5:1 for normal text, 3:1 for large text (18px+, per R-25). Round only for display, so a pairing at 4.498:1 prints as 4.50 and still fails 4.5:1. The maximum ratio is 21.0 (black on white).

**The reference table** (common pairings, computed with the formula):

| Pairing (text on background) | Ratio | Normal text (4.5) | Large text (3.0) |
|------------------------------|-------|-------------------|------------------|
| Black on white | 21.00 | Pass | Pass |
| White on black | 21.00 | Pass | Pass |
| White on #333333 | 12.63 | Pass | Pass |
| White on #666666 | 5.74 | Pass | Pass |
| #777777 on white | 4.48 | Fail | Pass |
| #777777 on #070707 | 4.50 | Fail | Pass |
| White on #888888 | 3.54 | Fail | Pass |
| White on #999999 | 2.85 | Fail | Fail |
| #555555 on black | 2.82 | Fail | Fail |

The `#777777 on #070707` row is the near-threshold case: 4.4983:1 rounds to 4.50 for display and still fails 4.5:1.

Read the table as a sanity check, not as a substitute. Any pairing not listed, or anything near a threshold, goes through the formula or the script.

## Keyboard

### Removed Focus Outline

- **Tell:** `outline: none` or `outline: 0` with no replacement focus style.
- **Why:** keyboard users cannot see where they are. It is the fastest way to make a UI unusable without a mouse, and R-32 forbids it outright.
- **Fix:** keep or replace the outline with a visible `:focus-visible` style that meets the same contrast bar (3:1 against its neighbors). Never set `outline: none` without a replacement.

### Mouse-Only Patterns

- **Tell:** menus that open on hover only, dropdowns that click-open but do not keyboard-open, drag-and-drop with no keyboard fallback.
- **Why:** each one excludes keyboard and assistive-technology users (R-32). If a control cannot be reached and operated by Tab, Enter, or Space, it does not exist for a whole group of people.
- **Fix:** every interactive element is reachable and operable by keyboard (R-32): logical tab order following visual order, activation with Enter or Space, and dialogs closable with Escape (R-26).

### Broken Tab Order

- **Tell:** focus jumps around the page, skips content, or lands on hidden elements because the DOM order does not match the visual order.
- **Why:** tab order that reads the code order instead of the visual order makes navigation unpredictable (R-32). Users lose their place and the page feels broken.
- **Fix:** keep the source order matching the visual order, add skip links for long pages, and never give real content `tabindex="-1"` unless it is part of a controlled focus trap such as a dialog.

## Focus & States

### Weak or Invisible Focus Indicator

- **Tell:** a focus ring the same color as the background, a ring that only appears on hover, or an indicator thinner than a 1px border.
- **Why:** the focus indicator is how keyboard users know where they are. If it fails the contrast bar or only shows on hover, keyboard-only use breaks (R-32, R-34).
- **Fix:** a visible focus indicator on every interactive element, 3:1 against adjacent colors, in every theme you ship. Check it in dark and light mode.

### Color-Only Feedback

- **Tell:** success, error, and status communicated only by color: red error text, green success border, a tinted chip, with no icon, label, or text.
- **Why:** it excludes color-blind and low-vision users, and it disappears entirely in forced-colors mode. A status that depends on seeing hue is not a status (C-4).
- **Fix:** pair every color signal with text, an icon, or a pattern. Error states are text first: "Password must be at least 8 characters", not just a red border.

### Missing UI States

- **Tell:** a data view with no empty, loading, or error state, or states that exist but are invisible: a spinner with no text, an empty screen with no explanation.
- **Why:** R-27 requires the three states; the accessibility angle is that each must be perceivable and informative, not decorative. A loading spinner with no context reads as a frozen page to screen-reader users.
- **Fix:** every data view has all three states (R-27), each announced or visible: an explicit empty message, a loading state with text, and an error state that says what happened and how to proceed.

## Zoom & Mobile Use

The layout mechanics behind mobile (breakpoints, scale, grids, overflow, tap targets) are the concern of `antislop-layoutmobile`. This section keeps only the mobile details that exclude people: zooming, and the on-screen keyboard.

### Text That Cannot Zoom

- **Tell:** fixed pixel font sizes, or containers with `overflow: hidden` that clip text at 200% zoom.
- **Why:** users must be able to resize text (WCAG 1.4.4). If zooming to 200% clips the content or forces horizontal scroll, the text is not resizable in practice.
- **Fix:** fluid type that reflows with zoom, no clipping containers on text, and verify the layout holds at 200% zoom on a narrow viewport (R-35).

### Mobile Keyboard Covers the Form

- **Tell:** inputs at the bottom of the viewport hidden behind the on-screen keyboard, with no scroll-into-view and no room for the input.
- **Why:** a form the user cannot see or reach is a form they cannot complete. It is a mobile-only exclusion (R-03).
- **Fix:** when an input is focused, it scrolls into view above the keyboard, with enough bottom padding that the focused field is never covered. Test with a real device or an emulated keyboard.

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

Run with the core Delivery Gate and the Human Skill Checklist (R-39). All answers must be **yes**:

- [ ] Does every enabled interactive element show `cursor: pointer` (and disabled ones `not-allowed`), set once in the base layer?
- [ ] Is every action a `<button>` and every navigation an `<a href>`, with no clickable `div`/`span`?
- [ ] Does every icon-only control have an accessible name, with the icon `aria-hidden`?
- [ ] Do controls have distinct hover, active, focus-visible, and disabled states, not color only?
- [ ] Is every target at least 24x24 CSS px (44x44 for primary touch controls)?
- [ ] Does all non-essential motion stop under `prefers-reduced-motion: reduce`?
- [ ] Are `lang`, `<title>`, landmarks, heading order, and a skip link in place?
- [ ] Does every form field have a visible label, `autocomplete` where relevant, and text errors linked to the field?

## WCAG 2.2 Depth

The full WCAG 2.2 guidance ships with this skill as references, read on demand. Open the one that matches the task instead of guessing a success criterion:

- [references/ACCESSIBILITY.md](references/ACCESSIBILITY.md): the audit workflow (Lighthouse, axe, accessibility-tree snapshot), POUR, text alternatives, media, keyboard, focus not obscured, target size, dragging, timing, motion, language, consistent help, forms and errors, redundant entry, accessible authentication, ARIA, live regions, and issues by impact.
- [references/A11Y-PATTERNS.md](references/A11Y-PATTERNS.md): copy-ready markup for modal focus traps, skip links, form labels, error announcement, ARIA tabs, live regions, and screen-reader commands.
- [references/WCAG.md](references/WCAG.md): every WCAG 2.2 success criterion by level, for checking conformance claims.

Never claim WCAG conformance from a Lighthouse score alone: automated tools catch a subset of barriers (R-36).

## Human Skill Checklist

Run these alongside the core Delivery Gate when the task involves UI. All answers must be **yes**:

- [ ] Is every text and background pairing verified against the contrast checker (formula, table, or script), including text over images and gradients? (R-25)
- [ ] Does every interactive component boundary and status indicator meet 3:1 against its background? (non-text contrast)
- [ ] Is the focus indicator visible, high-contrast, and present on every interactive element in every theme? (R-32, R-34)
- [ ] Is every interactive element reachable and operable by keyboard, with dialogs closable via Escape and no `outline: none` without a replacement? (R-32, R-26)
- [ ] Are the empty, loading, and error states of every data view present and perceivable, not color-only? (R-27, C-4)
- [ ] Can text be resized to 200% without being clipped, and does the mobile keyboard never cover a focused input? (R-35)
- [ ] Does every enabled interactive element show `cursor: pointer` (disabled ones `not-allowed`), use a native `<button>` or `<a href>`, carry an accessible name, and show distinct hover, active, focus-visible, and disabled states? (R-39)
- [ ] Are `lang`, `<title>`, landmarks, heading order, a skip link, visible form labels, `alt` text, and `prefers-reduced-motion` handling in place? (R-39)
