# Navigation & Links — Fix Guide

> For complete before/after code patterns, load:
> `navable://docs/fix-patterns/link-name,bypass,frame-title,button-name,label-content-name-mismatch,meta-viewport`

## Rules

| Rule ID | Summary | Impact |
|---|---|---|
| `link-name` | Link has no discernible text | serious |
| `bypass` | No skip navigation mechanism | serious |
| `frame-title` | `<iframe>` missing `title` | serious |
| `button-name` | Button has no discernible text | critical |
| `label-content-name-mismatch` | Accessible name doesn't match visible text | serious |
| `meta-viewport` | Viewport disables user scaling | critical |

## Framework Patterns

**React / Next.js:**
- Next.js `<Link>`: ensure visible text or `aria-label`. Icon-only links need `aria-label`.
- Add skip link as first element in root layout: `<a href="#main" className="sr-only focus:not-sr-only">Skip to content</a>`.
- Icon-only buttons: use `aria-label` or visually hidden `<span className="sr-only">`.
- Remove `maximum-scale` and `user-scalable=no` from viewport meta in `layout.tsx`.

**Vue / Nuxt:**
- Nuxt `<NuxtLink>`: same rules — always provide text content or `aria-label`.
- Add skip link in `App.vue` or default layout, before `<nav>`.
- Check `nuxt.config.ts` `app.head.meta` for zoom restrictions.

## Common Mistakes

- Icon-only buttons/links without any text alternative — screen readers announce nothing.
- `aria-label` that doesn't contain the visible text (e.g. button says "Send" but label says "Submit form").
- Third-party iframes (maps, videos, payment) without `title` — always add descriptive titles.
