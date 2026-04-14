# Keyboard & Focus — Fix Guide

> For complete before/after code patterns, load:
> `navable://docs/fix-patterns/tabindex,focus-order-semantics,keyboard,target-size,accesskeys,no-autoplay-audio,scrollable-region-focusable`

## Rules

| Rule ID | Summary | Impact |
|---|---|---|
| `tabindex` | Element has positive `tabindex` (overrides DOM order) | serious |
| `focus-order-semantics` | Interactive role not in tab order | minor |
| `keyboard` | Interactive element not keyboard accessible | critical |
| `target-size` | Touch target smaller than 24×24 CSS pixels | serious |
| `accesskeys` | Duplicate `accesskey` values | serious |
| `no-autoplay-audio` | Audio autoplays > 3s without controls | moderate |
| `scrollable-region-focusable` | Scrollable region not keyboard accessible | serious |

## Tabindex Rules

- `tabindex="0"` — adds element to natural tab order
- `tabindex="-1"` — focusable programmatically only
- **Never use positive values** — they override DOM order and create unpredictable navigation

## Framework Patterns

**React / Next.js:**
- Use native `<button>` or `<a>` for interactive elements — they have built-in keyboard support.
- Custom interactive elements need `tabIndex={0}` + `onKeyDown` handler (Enter/Space).
- Set `min-width`/`min-height` of 44px on touch targets. Tailwind: `min-w-11 min-h-11`.
- Add `muted` to autoplay video. Always include `controls`.
- Scrollable containers without focusable children need `tabIndex={0}`, `role="region"`, `aria-label`.

**Vue / Nuxt:**
- Use `@click` on `<button>`, not on `<div>`. Custom elements need `tabindex="0"` + `@keydown`.
- Tailwind: `min-w-11 min-h-11` for touch targets.
- Use `:muted="true"` and `:controls="true"` on autoplay media.

## Common Mistakes

- Using `<div onclick>` instead of `<button>` — divs have no keyboard support by default.
- Small icon buttons without padding — the visual icon may be 16px but the hit area needs ≥ 44px.
- Scrollable code blocks or overflow containers missing keyboard access.
- `accesskey` collisions across components rendered on the same page.
