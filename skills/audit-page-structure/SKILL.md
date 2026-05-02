---
name: audit-page-structure
description: >
  Audits page structure for accessibility: landmark regions, heading hierarchy, navigation
  patterns, skip links, and document metadata. Use ONLY when the user explicitly asks about
  landmarks, heading order, page outline, or navigation structure — NOT for general accessibility
  scans or fix workflows (use scan-accessibility for those).
license: MIT
compatibility:
  Requires the navable MCP server (npx -y @navable/mcp) and Playwright Chromium for URL scanning.
metadata:
  argument-hint: <url or file-path>
---

# Page Structure Audit

## What This Checks

1. **Landmark regions** — header, nav, main, footer, aside (correct nesting and labeling)
2. **Heading hierarchy** — single h1, no skipped levels, meaningful text
3. **Navigation patterns** — skip links, consistent navigation, labeled nav regions
4. **Document metadata** — lang attribute, title element, viewport meta

## Procedure

### With Browser (URL provided)

1. Call `run_accessibility_scan` with the URL. For an audit (one-shot, high-confidence review),
   prefer dual-engine: `run_accessibility_scan({ url, engines: ["axe", "htmlcs"] })`. The extra
   ~2–4 s is acceptable for audits and surfaces structural issues axe alone can miss
   (e.g. deprecated `align` attributes, missing iframe titles).
2. Filter results for structural rules:
   - `region`, `landmark-one-main`, `landmark-no-duplicate-main`, `landmark-banner-is-top-level`,
     `landmark-contentinfo-is-top-level`, `landmark-main-is-top-level`, `landmark-unique`
   - `heading-order`, `page-has-heading-one`, `empty-heading`, `p-as-heading`
   - `document-title`, `html-has-lang`, `html-lang-valid`
   - `bypass`
3. Present findings grouped by category (landmarks, headings, navigation, metadata)

### Without Browser (source files provided)

1. Identify the layout/page component files (layout.tsx, App.vue, index.html, etc.)
2. Check for landmark elements per `navable://docs/semantic-html` reference:
   - Is there a `<header>` / `<footer>` / `<main>` / `<nav>`?
   - Are there multiple `<nav>` elements? Each needs a unique `aria-label`.
3. Trace heading hierarchy across components:
   - Is there exactly one `<h1>`?
   - Are levels sequential (h1 → h2 → h3, no skips)?
4. Check the `<html>` tag for `lang` attribute
5. Check `<head>` for `<title>` element
6. Check for skip link as first focusable element

## Expected Page Structure

```
<html lang="de">                         ← Document language
<head>
  <title>Page Title — Site Name</title>  ← Document title
  <meta name="viewport" content="width=device-width, initial-scale=1">
</head>
<body>
  <a href="#main" class="skip-link">     ← Skip navigation
    Skip to content
  </a>
  <header>                               ← banner landmark
    <nav aria-label="Main">              ← navigation landmark (labeled)
      ...
    </nav>
  </header>
  <main id="main">                       ← main landmark (skip link target)
    <h1>Page Title</h1>                  ← Single h1
    <section aria-label="...">           ← region landmark (labeled)
      <h2>Section Title</h2>            ← Sequential heading
      <h3>Subsection</h3>              ← No skipped levels
    </section>
  </main>
  <footer>                               ← contentinfo landmark
    <nav aria-label="Footer">            ← labeled to distinguish from main nav
      ...
    </nav>
  </footer>
</body>
</html>
```

## Common Issues

| Issue                      | Rule                 | Fix                                                                                  |
| -------------------------- | -------------------- | ------------------------------------------------------------------------------------ |
| No `<main>` landmark       | landmark-one-main    | Wrap primary content in `<main>`                                                     |
| Content outside landmarks  | region               | Ensure all content is inside `<header>`, `<main>`, `<nav>`, `<footer>`, or `<aside>` |
| Multiple unlabeled `<nav>` | landmark-unique      | Add `aria-label` to each `<nav>`                                                     |
| No skip link               | bypass               | Add skip link as first focusable element                                             |
| Missing `<h1>`             | page-has-heading-one | Add one `<h1>` per page                                                              |
| Skipped heading levels     | heading-order        | Use sequential levels (h1 → h2 → h3)                                                 |
| Missing `lang` attribute   | html-has-lang        | Add `lang` to `<html>` element                                                       |
| No `<title>`               | document-title       | Add `<title>` to `<head>`                                                            |

## Fix Guides

- Landmarks: see `scan-accessibility/references/fix-guide-landmarks.md`
- Headings: see `scan-accessibility/references/fix-guide-headings.md`
- Navigation: see `scan-accessibility/references/fix-guide-navigation.md`
- Language: see `scan-accessibility/references/fix-guide-language.md`

## MCP Resources

- `navable://docs/semantic-html` — HTML elements → implicit ARIA roles
- `navable://docs/wcag-mapping` — WCAG 2.1 AA criteria for structural elements

## Gotchas

- **`<header>` and `<footer>` only map to banner/contentinfo landmarks when they are direct children
  of `<body>`.** Nested inside `<article>` or `<section>`, they have no implicit landmark role.
- **SPA frameworks** may render heading hierarchy across multiple components. Trace the full
  component tree to audit heading levels.
- **Multiple `<main>` elements** are allowed if only one is visible (others have `hidden`
  attribute). But only one should be visible at a time.
