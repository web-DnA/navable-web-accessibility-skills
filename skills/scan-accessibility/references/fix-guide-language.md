# Language Attributes — Fix Guide

> For complete before/after code patterns, load:
> `navable://docs/fix-patterns/html-has-lang,html-lang-valid,valid-lang`

## Rules

| Rule ID | Summary | Impact |
|---|---|---|
| `html-has-lang` | `<html>` missing `lang` attribute | serious |
| `html-lang-valid` | `lang` attribute is not a valid BCP 47 tag | serious |
| `valid-lang` | Element-level `lang` attribute invalid | serious |

## Common BCP 47 Codes

`de`, `en`, `fr`, `es`, `it`, `nl`, `de-AT`, `de-CH`, `en-US`, `en-GB`

## Framework Patterns

**React / Next.js:**
- Set `lang` in root layout: `<html lang="de">`.
- In Next.js, can also set via `next.config.js` i18n config.
- Use `lang` attribute on elements with mixed-language content.

**Vue / Nuxt:**
- Set in `nuxt.config.ts`: `app: { head: { htmlAttrs: { lang: 'de' } } }`.
- Use `:lang` for dynamic language switching on elements.

## Common Mistakes

- Using full language names (`deutsch`) instead of BCP 47 codes (`de`).
- Forgetting to set `lang` entirely — screen readers cannot determine pronunciation.
- Multi-language pages without element-level `lang` on foreign-language sections.
