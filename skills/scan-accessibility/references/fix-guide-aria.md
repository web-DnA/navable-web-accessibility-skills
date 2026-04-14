# ARIA Usage — Fix Guide

> For complete before/after code patterns, load:
> `navable://docs/fix-patterns/aria-required-attr,aria-valid-attr-value,aria-allowed-attr,aria-hidden-focus,aria-allowed-role,aria-required-children,aria-required-parent,aria-roles,aria-valid-attr,duplicate-id,duplicate-id-aria`

## Rules

| Rule ID | Summary | Impact |
|---|---|---|
| `aria-required-attr` | ARIA role missing required attribute | critical |
| `aria-valid-attr-value` | ARIA attribute has invalid value | critical |
| `aria-allowed-attr` | ARIA attribute not allowed on element's role | critical |
| `aria-hidden-focus` | Focusable element inside `aria-hidden="true"` | serious |
| `aria-allowed-role` | ARIA role not allowed on element type | minor |
| `aria-required-children` | ARIA role missing required child roles | critical |
| `aria-required-parent` | ARIA role not inside required parent | critical |
| `aria-roles` | Invalid or misspelled ARIA role | critical |
| `aria-valid-attr` | Invalid or misspelled ARIA attribute | critical |
| `duplicate-id` | Multiple elements share same `id` | minor |
| `duplicate-id-aria` | Duplicate `id` referenced by ARIA attributes | critical |

## Key Principle

**No ARIA is better than bad ARIA.** Always prefer native HTML elements that provide the needed semantics. Only add ARIA when no native element exists.

For ARIA widget pattern details, load: `navable://docs/aria-patterns/{patternSlug}`

## Framework Patterns

**React / Next.js:**
- ARIA boolean attributes must be string values in JSX: `aria-selected="true"`, not `aria-selected={true}`.
- Use `React.useId()` to generate unique IDs in reusable components — prevents `duplicate-id` and `duplicate-id-aria`.
- TypeScript and `eslint-plugin-jsx-a11y` catch misspelled roles/attributes at lint time.
- For hidden content, prefer removing from DOM or using `inert` attribute over `aria-hidden="true"` on containers with focusable children.

**Vue / Nuxt:**
- Bind ARIA values with `:aria-checked="String(isChecked)"` to ensure string output.
- Use `v-if` to remove hidden content instead of `aria-hidden` on containers with interactive children.
- Use `eslint-plugin-vuejs-accessibility` to catch invalid roles.
- Generate unique IDs per component instance with a composable.

## Common Mistakes

- Adding `role="button"` to a `<div>` but forgetting `tabindex="0"` and keyboard handlers.
- Using `aria-hidden="true"` on a container that still has focusable children.
- Parent/child role mismatch: `role="tablist"` must contain `role="tab"` children.
- Overriding native semantics: `<input type="text" role="button">` — use the right element.
