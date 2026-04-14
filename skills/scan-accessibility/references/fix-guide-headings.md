# Headings & Structure — Fix Guide

> For complete before/after code patterns, load:
> `navable://docs/fix-patterns/heading-order,page-has-heading-one,document-title,empty-heading,p-as-heading`

## Rules

| Rule ID | Summary | Impact |
|---|---|---|
| `heading-order` | Heading levels are skipped (e.g. h1 → h3) | moderate |
| `page-has-heading-one` | Page missing an `<h1>` | moderate |
| `document-title` | Document missing `<title>` | serious |
| `empty-heading` | Heading element has no text content | minor |
| `p-as-heading` | `<p>` styled to look like heading but uses no heading tag | serious |

## Framework Patterns

**React / Next.js:**
- Heading levels must be sequential across the component tree. Consider a `HeadingLevel` context for dynamic heading levels in reusable components.
- Ensure exactly one `<h1>` per page. In Next.js App Router, typically in the page component, not the layout.
- Export `metadata` from `layout.tsx` or `page.tsx` for document title. In Pages Router, use `next/head`.
- Watch for conditional rendering that can leave empty headings.

**Vue / Nuxt:**
- Components rendering headings should accept a `level` prop to stay sequential.
- Place one `<h1>` per page view. In Nuxt, typically in the page component.
- Use `useHead({ title: '...' })` for document title.
- Check `v-if` conditions that may render empty headings — provide fallback text.

## Common Mistakes

- Choosing heading level for visual size instead of document structure — use CSS for styling.
- Nested components each rendering `<h2>` when one should be `<h3>` — headings form a tree.
- Missing `<title>` in SPAs — each route change should update the document title.
