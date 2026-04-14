# Images & Media — Fix Guide

> For complete before/after code patterns, load:
> `navable://docs/fix-patterns/image-alt,area-alt,input-image-alt,object-alt,role-img-alt,svg-img-alt,video-caption`

## Rules

| Rule ID | Summary | Impact |
|---|---|---|
| `image-alt` | `<img>` missing `alt` attribute | critical |
| `area-alt` | `<area>` in image map missing `alt` | critical |
| `input-image-alt` | `<input type="image">` missing `alt` | critical |
| `object-alt` | `<object>` missing accessible text | serious |
| `role-img-alt` | `role="img"` element missing accessible name | serious |
| `svg-img-alt` | `<svg role="img">` missing accessible name | serious |
| `video-caption` | `<video>` missing `<track kind="captions">` | critical |

## Framework Patterns

**React / Next.js:**
- Use `alt` prop on `<img>` and Next.js `<Image>`. For decorative images use `alt=""`.
- SVG icons: add `aria-label` to `<svg>` or include `<title>` as first child. Decorative SVGs get `aria-hidden="true"`.
- For video, use a player component that supports VTT captions.

**Vue / Nuxt:**
- Use `:alt` for dynamic alt text. Decorative images: `alt=""` and `role="presentation"`.
- Replace image maps with styled links or SVG navigation.
- Provide `.vtt` caption files with `<track>`.

## Common Mistakes

- Using `alt="image"` or `alt="photo"` — describe the content, not the element type.
- Forgetting alt on decorative images — use `alt=""`, not omitting the attribute entirely.
- Using CSS background images for meaningful content — these have no text alternative.
- Assuming video platforms auto-generate accurate captions — always provide authored `.vtt` files.
