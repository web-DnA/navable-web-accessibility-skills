# Color & Contrast — Fix Guide

> For complete before/after code patterns, load:
> `navable://docs/fix-patterns/color-contrast,color-contrast-enhanced,link-in-text-block,css-orientation-lock`

## Rules

| Rule ID | Summary | Impact |
|---|---|---|
| `color-contrast` | Insufficient contrast ratio (AA: 4.5:1 normal, 3:1 large) | serious |
| `color-contrast-enhanced` | Insufficient enhanced contrast (AAA: 7:1 normal, 4.5:1 large) | serious |
| `link-in-text-block` | Link in text not distinguishable without color alone | serious |
| `css-orientation-lock` | Content locked to single display orientation via CSS | serious |

## Minimum Contrast Ratios

- **Normal text** (< 18pt or < 14pt bold): **4.5:1** (AA), **7:1** (AAA)
- **Large text** (≥ 18pt or ≥ 14pt bold): **3:1** (AA), **4.5:1** (AAA)

## Framework Patterns

**React / Next.js:**
- Check contrast in Tailwind classes — avoid low-contrast grays for body text (e.g. `text-gray-400` on white).
- Keep `text-decoration: underline` on inline links, or add `border-bottom`.
- Never use `display: none` in orientation media queries — adapt layout instead.

**Vue / Nuxt:**
- Ensure scoped styles meet ratios. Check `:style` bindings for dynamic colors.
- Watch for `transform: rotate(90deg)` hacks that force orientation.

## Common Mistakes

- Relying on color alone to distinguish links from surrounding text — always add underline or border.
- Low-contrast placeholder text — placeholders still need 4.5:1 ratio.
- Testing contrast with a color picker but forgetting hover/focus/active states.
- Using `user-scalable=no` or `maximum-scale=1` in viewport meta — these restrict zoom.
