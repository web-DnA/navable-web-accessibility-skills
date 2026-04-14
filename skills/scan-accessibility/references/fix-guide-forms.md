# Forms & Labels — Fix Guide

> For complete before/after code patterns, load:
> `navable://docs/fix-patterns/label,select-name,autocomplete-valid,input-button-name`

## Rules

| Rule ID | Summary | Impact |
|---|---|---|
| `label` | Form element missing associated label | critical |
| `select-name` | `<select>` missing associated label | critical |
| `autocomplete-valid` | Invalid `autocomplete` attribute value | serious |
| `input-button-name` | `<input type="button/submit/reset">` missing `value` | critical |

## Framework Patterns

**React / Next.js:**
- Use `htmlFor` (not `for`) on `<label>` to associate with input `id`.
- Wrapping label is often simpler: `<label>Email <input /></label>`.
- Prefer `<button type="submit">` over `<input type="submit">` for easier labeling.
- Standard HTML `autocomplete` values work as-is: `email`, `tel`, `street-address`, etc.

**Vue / Nuxt:**
- `for` attribute works as in plain HTML. Wrapping label also works.
- Bind autocomplete statically (`autocomplete="email"`) or dynamically (`:autocomplete`).

## Common Mistakes

- Using `placeholder` as a label substitute — placeholders disappear on input and are not reliably announced.
- Floating-label patterns that remove the `<label>` element — keep a real `<label>` even if visually styled.
- Missing labels on search inputs — even single-field forms need labels (can be visually hidden).
