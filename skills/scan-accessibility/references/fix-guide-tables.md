# Tables & Lists — Fix Guide

> For complete before/after code patterns, load:
> `navable://docs/fix-patterns/td-has-header,th-has-data-cells,table-fake-caption,definition-list,dlitem,list,listitem`

## Rules

| Rule ID | Summary | Impact |
|---|---|---|
| `td-has-header` | `<td>` not associated with a `<th>` header | critical |
| `th-has-data-cells` | `<th>` has no associated data cells | serious |
| `table-fake-caption` | Table uses colspan cell as fake caption | serious |
| `definition-list` | `<dl>` contains invalid direct children | serious |
| `dlitem` | `<dt>`/`<dd>` not inside a `<dl>` | serious |
| `list` | `<ul>`/`<ol>` contains non-`<li>` children | serious |
| `listitem` | `<li>` not inside `<ul>` or `<ol>` | serious |

## Framework Patterns

**React / Next.js:**
- Use `<th>` for header cells with `scope="col"` or `scope="row"` for complex tables.
- Use `<caption>` as first child of `<table>` — never simulate with colspan cells.
- Components inside `<ul>`/`<ol>` must render `<li>` as root element — watch for Fragment wrappers.
- Components inside `<dl>` must render `<dt>` or `<dd>` as root.

**Vue / Nuxt:**
- Use `<thead>`/`<th>` for header rows. Dynamic tables should match header count to data columns.
- `<dl>` children must be `<dt>`, `<dd>`, or `<div>` wrapping dt/dd pairs.
- Child components in lists must render `<li>` as root element.

## Common Mistakes

- Layout tables without `role="presentation"` — screen readers interpret them as data tables.
- Component boundaries breaking list/dl nesting — parent renders `<ul>`, child renders `<div>` instead of `<li>`.
- Extra `<th>` columns without matching data cells in dynamically generated tables.
