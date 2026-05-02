---
name: fix-accessibility
description: >
  Executes an existing accessibility fix plan from .navable-plan.json. Works through pending items
  in priority order, applying code fixes with before/after patterns. Use when a .navable-plan.json
  file exists and the user wants to fix, apply, or continue fixing accessibility issues.
license: MIT
compatibility: Requires the navable MCP server (npx -y @navable/mcp) for fix status updates.
metadata:
  argument-hint: '[item-id or all]'
---

# Fix Accessibility Plan Executor

## Prerequisites

- `.navable-plan.json` must exist in the project root
- If no plan exists, tell the user to run the `scan-accessibility` skill first

## Workflow

### Step 1: Load Plan

1. Read `.navable-plan.json` from the project root
2. Parse and display a summary:
   - Total items, pending count, done count, skipped count
   - Breakdown by impact level
3. List pending items in their current order (already sorted by priority)

### Step 2: Determine Scope

- If the user specified an item ID (e.g. "fix-3") → fix only that item
- If the user said "all" or didn't specify → fix all pending items in order
- If the user asked about a specific rule (e.g. "fix the contrast issues") → filter items by
  `ruleId`

### Step 3: Fix Each Item

**Before starting:** Group plan items by DOM element. Multiple items often target the same element
(separate axe nodes in a multi-node violation, or cross-engine flags under different WCAG criteria).
Fixing one element once avoids regressions and cuts file reads/writes.

Two items belong in the same bucket when both:

1. **Selectors match** — identical, **or** one selector is a tail of the other (the longer one has
   ` > ` immediately before where the shorter one starts; the shorter side need not contain ` > `
   itself, so `img:nth-child(1)` matches `… > img:nth-child(1)`), **or** they match after
   stripping `[attribute]` filters (e.g. `button[type="button"]` matches `button`). When you fall
   back to the attribute-stripped path, require **exact HTML equality** — distinct elements like
   `input[type="checkbox"]` and `input[type="radio"]` collapse to the same stripped selector, so
   any HTML divergence means they are different elements.
2. **`affectedNodes[0].html` snippets match** — normalize whitespace (collapse runs of spaces,
   trim) and lowercase, then compare the first ~200 characters. The 200-char window is wide enough
   to catch divergence in child content (e.g. `<option>` text inside two different `<select>`).

> If selectors look similar but HTML differs, treat as different elements. Two `<select>` in two
> forms can share the suffix `form > div:nth-child(4) > select` — their `<option>` content
> disambiguates them, but only if you compare enough of the HTML.
>
> **`:nth-child` index drift.** axe and HTMLCS occasionally disagree on `:nth-child` indices when
> there are sibling text nodes or comments. If two findings are clearly the same element but the
> indices differ by one, trust the HTML snippet over the selector and bucket them together.

For each bucket, design **one minimal edit** addressing every fix in the bucket, then call
`update_fix_status` with **all** resolved fix IDs in one call.

For each pending item (or bucket):

1. **Set status to `in_progress`** in `.navable-plan.json` and save
2. **Identify the violation category** from `item.ruleId`
3. **Load the fix guide** — use the category mapping from the `scan-accessibility` skill:
   - Images: `scan-accessibility/references/fix-guide-images.md`
   - Forms: `scan-accessibility/references/fix-guide-forms.md`
   - Color: `scan-accessibility/references/fix-guide-color.md`
   - Navigation: `scan-accessibility/references/fix-guide-navigation.md`
   - Headings: `scan-accessibility/references/fix-guide-headings.md`
   - Landmarks: `scan-accessibility/references/fix-guide-landmarks.md`
   - ARIA: `scan-accessibility/references/fix-guide-aria.md`
   - Keyboard: `scan-accessibility/references/fix-guide-keyboard.md`
   - Language: `scan-accessibility/references/fix-guide-language.md`
   - Tables: `scan-accessibility/references/fix-guide-tables.md`
4. **Locate the source file** using `item.affectedNodes`:
   - Use CSS selectors and HTML snippets to find the component
   - Search for unique class names, text content, or attributes in the codebase
   - Check `package.json` for framework (React/Vue/Svelte/Angular) to guide file search
5. **Apply the fix** following the before/after pattern from the fix guide
6. **Update `.navable-plan.json`**:
   ```json
   { "status": "done", "appliedAt": "2025-01-15T10:30:00.000Z" }
   ```
7. **Save `.navable-plan.json` after each fix** (not in a batch)

### Step 4: Summary

After fixing all targeted items:

1. Report what was fixed (count, rule IDs, impact levels)
2. If all items are done → suggest re-scanning with `run_accessibility_scan` for verification
3. If items remain → list what's still pending with their priority

### Step 5: Verification (if all items done)

If the user agrees to verify:

1. Call `run_accessibility_scan` with the same URL from `plan.url`
2. Compare results against the plan
3. Update `.navable-plan.json`:
   ```json
   {
     "verification": {
       "completedAt": "2025-01-15T11:00:00.000Z",
       "remainingViolations": 0,
       "passed": true
     }
   }
   ```

## Status Values

| Status        | Meaning                                              |
| ------------- | ---------------------------------------------------- |
| `pending`     | Not yet addressed                                    |
| `in_progress` | Currently being fixed (set at start of fix)          |
| `done`        | Fix applied successfully                             |
| `skipped`     | Intentionally skipped — add a comment explaining why |

## Skipping Items

If an item cannot be fixed (e.g. third-party content, intentional design decision):

1. Set `status: "skipped"` in `.navable-plan.json`
2. Explain to the user why it was skipped
3. Note: Critical items should not be skipped without explicit user confirmation

## Gotchas

- **Always save `.navable-plan.json` after each individual fix.** This ensures resumability if the
  session is interrupted.
- **Do not modify the `priority` or sort order of items.** The server set the correct order.
- **If a source file can't be found**, report the CSS selector and HTML snippet to the user and ask
  for guidance.
- **Multiple items may affect the same file.** Apply fixes carefully to avoid conflicts. Re-read the
  file before each fix.
- **The `manualReview` array** contains issues that need human judgment. Mention these to the user
  but do not auto-fix.
