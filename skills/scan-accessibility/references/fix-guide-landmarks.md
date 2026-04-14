# Landmarks & Regions — Fix Guide

> For complete before/after code patterns, load:
> `navable://docs/fix-patterns/region,landmark-one-main`

## Rules

| Rule ID | Summary | Impact |
|---|---|---|
| `region` | Page content not contained in a landmark region | moderate |
| `landmark-one-main` | Page missing `<main>` or `role="main"` | moderate |

## Expected Landmark Structure

```
<header>         → banner
  <nav>          → navigation
</header>
<main>           → main
  <section>      → region (needs aria-label)
  <aside>        → complementary
</main>
<footer>         → contentinfo
```

## Framework Patterns

**React / Next.js:**
- Wrap layout in semantic elements. In Next.js App Router, add `<main>` inside root `layout.tsx` wrapping `{children}`.
- If multiple `<nav>` elements exist, label each: `<nav aria-label="Main">`, `<nav aria-label="Footer">`.

**Vue / Nuxt:**
- Use `<header>`, `<main>`, `<footer>` in `App.vue` or default layout.
- In Nuxt, wrap `<NuxtPage />` in `<main>` inside the default layout.

## Common Mistakes

- Nesting `<main>` inside another landmark — `<main>` must be top-level.
- Using `<section>` without `aria-label` — unlabeled sections are not useful landmarks.
- Multiple `<header>`/`<footer>` elements without distinguishing labels.

## Related Rules

These follow the same principle — correct nesting and unique labels:
- `landmark-no-duplicate-main`, `landmark-banner-is-top-level`, `landmark-contentinfo-is-top-level`, `landmark-main-is-top-level`, `landmark-unique`
