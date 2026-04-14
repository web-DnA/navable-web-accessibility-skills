---
name: review-component
description: >
  Reviews a component's source code for accessibility issues without running a browser. Checks ARIA
  attribute usage, semantic HTML patterns, keyboard navigation, and focus management. Use when the
  user asks to review, check, or audit a component, file, or code for accessibility.
license: MIT
compatibility:
  Requires the navable MCP server (npx -y @navable/mcp) for ARIA pattern and semantic HTML
  resources.
metadata:
  argument-hint: <file-path>
---

# Accessibility Code Review

## When to Use

- No running dev server needed
- Works on any component file (React, Vue, Svelte, plain HTML)
- Complements browser-based scanning — catches patterns scanners miss
- Good for PR reviews or pre-commit checks

## Review Checklist

Work through each section. Report findings grouped by impact level (critical, serious, moderate,
minor).

### 1. Semantic HTML

- Are interactive elements using native HTML? (`<button>`, `<a>`, `<input>`, `<select>`)
- Are `<div>` or `<span>` used where semantic elements exist? (e.g. `div` with `onClick` → should be
  `<button>`)
- Load `navable://docs/semantic-html` for HTML element → implicit ARIA role mapping

**Common mistakes:**

- `<div onClick>` instead of `<button>`
- `<span>` styled as a link instead of `<a>`
- `<div class="header">` instead of `<header>`

### 2. ARIA Usage

- Are ARIA roles correct for the pattern? (load `navable://docs/aria-patterns`)
- Are required ARIA attributes present? (e.g. `role="checkbox"` needs `aria-checked`)
- Is `aria-hidden="true"` used on containers with focusable children?
- Are ARIA attribute values valid? (`"true"`/`"false"` strings, not booleans)

**Common mistakes:**

- `aria-label` that doesn't match visible text (violates 2.5.3)
- Missing `aria-expanded` on disclosure triggers
- `aria-hidden="true"` on containers with buttons/links inside

### 3. Keyboard Navigation

- Can all interactive elements be reached via Tab?
- Do custom widgets handle Arrow keys, Enter, Escape per the APG pattern?
- Is there a visible focus indicator? (check for `outline: none` without a replacement)
- Are there keyboard traps? (modal dialogs, dropdown menus)

**Common mistakes:**

- `tabindex` > 0 (creates unpredictable tab order)
- Missing `onKeyDown` handler on custom interactive elements
- `outline: none` / `outline: 0` in CSS without alternative focus style

### 4. Form Accessibility

- Do all inputs have associated `<label>` elements (via `for`/`id` or wrapping)?
- Are error messages linked to inputs via `aria-describedby`?
- Is `autocomplete` set on personal data fields? (name, email, tel, address)
- Do required fields use `aria-required="true"` or the `required` attribute?

**Common mistakes:**

- Placeholder as the only label
- Error messages not programmatically associated with inputs
- Missing `autocomplete` on address/payment fields

### 5. Images and Media

- Do images have `alt` text? (decorative images get `alt=""`)
- Do SVGs have `aria-label` or `<title>`, or `aria-hidden="true"` if decorative?
- Do videos have `<track kind="captions">`?

**Common mistakes:**

- Missing `alt` on `<img>` (screen readers announce the filename)
- SVG icons without `aria-hidden="true"` (announced as "image" with no description)

### 6. Color and Contrast

- Is color the sole means of conveying information? (e.g. red = error, green = success without
  text/icons)
- Are interactive states (hover, focus, active) distinguishable without color?

**Common mistakes:**

- Form validation that only uses red borders (no text/icon)
- Links in text distinguished only by color (need underline or other cue)

## Output Format

Present findings as a prioritized list:

```
## Accessibility Review: ComponentName.tsx

### Critical
1. **Line 42:** `<div onClick={handleSubmit}>Submit</div>` — use `<button>` for interactive elements (keyboard inaccessible)

### Serious
2. **Line 15:** `<img src="logo.png">` — missing `alt` attribute
3. **Line 28:** `<input placeholder="Email">` — no `<label>` associated

### Moderate
4. **Line 8:** `<div class="nav">` — use `<nav>` for navigation landmark

### Recommendations
- Consider adding `aria-live="polite"` to the status message region (line 55)
- Add `autocomplete="email"` to the email input (line 28)
```

## MCP Resources (load on demand)

- `navable://docs/semantic-html` — HTML elements → implicit ARIA roles
- `navable://docs/aria-patterns` — 25 WAI-ARIA widget patterns with keyboard requirements
- `navable://docs/fix-patterns` — before/after code examples for common violations

## Gotchas

- **This review cannot catch runtime issues** like color contrast, focus management across route
  changes, or dynamic ARIA state bugs. Recommend `run_accessibility_scan` for runtime validation.
- **Framework-specific patterns matter.** React uses `htmlFor` not `for`, `className` not `class`,
  `tabIndex` not `tabindex`.
- **Don't flag decorative images missing alt** — they should have `alt=""`, not descriptive alt
  text.
- **Custom component libraries** (MUI, Radix, Headless UI) often handle ARIA internally. Check
  library docs before flagging missing ARIA on library components.
