---
name: scan-accessibility
description: >
  Scans a URL for WCAG 2.1 AA accessibility violations using Playwright and axe-core via the navable
  MCP server. Generates a prioritized fix plan with before/after code patterns and EN 301 549
  mapping. Use when the user asks to check, scan, audit, or test accessibility of a page, URL, or
  component.
license: MIT
compatibility: Requires the navable MCP server (npx -y @navable/mcp) and Playwright Chromium.
metadata:
  argument-hint: <url>
---

# Accessibility Scan & Fix Workflow

## Prerequisites

- navable MCP server must be configured (see Step 0)
- Playwright Chromium browser engine installed (`npx playwright install chromium`). If missing, the
  scan will return a clear error with install instructions.
- Target URL accessible on localhost (any port). External hosts require a `.navable.json` config
  with `allowedHosts`.

## Workflow

### Step 0: Ensure navable MCP Server is Configured

Check if the navable MCP server is already available by looking for an existing config:

1. **VS Code (Copilot)** — check if `.vscode/mcp.json` exists and contains a `"navable"` entry
2. **Cursor** — check if `.cursor/mcp.json` exists and contains a `"navable"` entry
3. **Claude Code** — check if `navable` appears in the MCP server list

If the navable MCP server is **not configured**, create the config file:

**For VS Code** — create or update `.vscode/mcp.json`:

```json
{
  "servers": {
    "navable": {
      "command": "npx",
      "args": ["-y", "@navable/mcp"]
    }
  }
}
```

**For Cursor** — create or update `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "navable": {
      "command": "npx",
      "args": ["-y", "@navable/mcp"]
    }
  }
}
```

**For Claude Code** — run in terminal:

```bash
claude mcp add navable -- npx -y @navable/mcp
```

After creating the config, the IDE will auto-detect and start the MCP server. The `@navable/mcp`
package is downloaded automatically via `npx` on first use — no separate install step needed.

If `.navable-plan.json` is written to the wrong directory, add the `NAVABLE_PROJECT_ROOT` env var to
the config pointing to the project root.

**Once the MCP server is available, proceed to Step 1.**

### Step 1: Check for Existing Plan (Resumability)

Check if `.navable-plan.json` exists in the project root.

- **If it exists with pending items** → skip to Step 4 (resume fixing)
- **If it exists with all items done** → ask user if they want to re-scan or verify
- **If it doesn't exist** → proceed to Step 2

### Step 2: Scan the URL

Call `run_accessibility_scan` with the user's URL:

```
run_accessibility_scan({ url: "<user-provided URL>" })
```

The URL can be any localhost address (e.g. `http://localhost:3000`,
`http://localhost:4200/checkout`, `http://127.0.0.1:8080`).

Optional parameters:

- `tags` — axe-core rule tags to run (default: `wcag2a, wcag21a, wcag2aa, wcag21aa`)
- `include` — CSS selectors to scope the scan to specific regions
- `exclude` — CSS selectors to skip (e.g. third-party widgets)

The result includes a `scanId` field — save it for Step 3.

**Note on large results:** Scan output is typically 10-20 KB. Some agents write large tool results
to temp files instead of displaying them inline. If the result is in a temp file, read it to extract
the `scanId` value. You do not need to pass the full scan object to the next step.

### Step 3: Generate Fix Plan

Use the `scanId` from Step 2 (preferred — avoids re-serializing large data):

```
generate_fix_plan({ scanId: "<scanId from step 2>" })
```

Fallback (only if the server was restarted and scanId is no longer available):

```
generate_fix_plan({ scan: <full_scan_output> })
```

This writes `.navable-plan.json` to the project root. The response includes `planPath` — the
absolute path where the file was written.

If `planPath` points to the wrong location, set `NAVABLE_PROJECT_ROOT` in the MCP server config.

The plan is pre-sorted by priority:

1. critical
2. serious
3. moderate
4. minor

**Do NOT reorder the plan.** The server applies the correct priority sort.

### Step 3b: Confirm with User Before Fixing

**STOP. Do not proceed to Step 4 automatically.**

Present the plan summary to the user and ask for explicit confirmation:

- State clearly: **"Scanned against WCAG 2.1 Level A + AA (50 criteria)."**
- Show the `summary` from the `generate_fix_plan` response (total items, critical/serious counts)
- List the `topItems` so the user can see what will be changed
- Ask: **"Do you want me to apply these fixes now?"**

Only proceed to Step 4 if the user confirms. If they decline or ask to review first, stop and wait.

> This is a hard stop — never auto-apply fixes after generating the plan.

### Step 4: Fix Items in Priority Order

Work through `plan.items` where `status === "pending"`:

For each item:

1. **Identify the violation category** from the `ruleId` and load the relevant fix guide
2. **Locate the source file** — use `item.affectedNodes[].selector` and `item.affectedNodes[].html`
   to find the component
3. **Apply the fix** following the before/after pattern from the fix guide
4. **Mark it done** — call `update_fix_status({ fixId: "fix-1", status: "done" })` to update the
   plan file. This is faster and safer than editing `.navable-plan.json` manually.
5. Move to the next pending item

Progress checklist (update as you go):

- [ ] Fix critical items
- [ ] Fix serious items
- [ ] Fix moderate items
- [ ] Fix minor items

### Step 5: Verify Fixes

Re-scan the URL with `run_accessibility_scan` using the same parameters as Step 2.

- If violations remain → report them and offer to continue fixing
- If clean → confirm all issues resolved
- Update `plan.verification` in `.navable-plan.json`:
  ```json
  { "completedAt": "<ISO 8601>", "remainingViolations": 0, "passed": true }
  ```

Always add this note:

> Automated scanning cannot detect every accessibility issue. Manual testing is still required for
> keyboard navigation flows, screen reader behavior, and cognitive load assessment.

## Fix Guides by Category

Load the relevant guide when fixing a specific violation type:

| Category             | Guide                                                                    | Rules                                                                                                                                                                                                          |
| -------------------- | ------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Images & media       | [references/fix-guide-images.md](references/fix-guide-images.md)         | image-alt, svg-img-alt, object-alt, input-image-alt, role-img-alt, area-alt, video-caption                                                                                                                     |
| Forms & labels       | [references/fix-guide-forms.md](references/fix-guide-forms.md)           | label, select-name, autocomplete-valid, input-button-name                                                                                                                                                      |
| Color & contrast     | [references/fix-guide-color.md](references/fix-guide-color.md)           | color-contrast, color-contrast-enhanced, link-in-text-block, css-orientation-lock                                                                                                                              |
| Navigation & links   | [references/fix-guide-navigation.md](references/fix-guide-navigation.md) | link-name, bypass, frame-title, button-name, label-content-name-mismatch, meta-viewport                                                                                                                        |
| Headings & structure | [references/fix-guide-headings.md](references/fix-guide-headings.md)     | heading-order, page-has-heading-one, document-title, empty-heading, p-as-heading                                                                                                                               |
| Landmarks & regions  | [references/fix-guide-landmarks.md](references/fix-guide-landmarks.md)   | region, landmark-one-main                                                                                                                                                                                      |
| ARIA usage           | [references/fix-guide-aria.md](references/fix-guide-aria.md)             | aria-required-attr, aria-valid-attr-value, aria-allowed-attr, aria-hidden-focus, aria-allowed-role, aria-required-children, aria-required-parent, aria-roles, aria-valid-attr, duplicate-id, duplicate-id-aria |
| Keyboard & focus     | [references/fix-guide-keyboard.md](references/fix-guide-keyboard.md)     | tabindex, focus-order-semantics, keyboard, target-size, accesskeys, no-autoplay-audio, scrollable-region-focusable                                                                                             |
| Language attributes  | [references/fix-guide-language.md](references/fix-guide-language.md)     | html-has-lang, html-lang-valid, valid-lang                                                                                                                                                                     |
| Tables & lists       | [references/fix-guide-tables.md](references/fix-guide-tables.md)         | td-has-header, th-has-data-cells, table-fake-caption, definition-list, dlitem, list, listitem                                                                                                                  |

## MCP Resources (load on demand)

- `navable://docs/fix-patterns/{ruleIds}` — **preferred**: before/after code for specific rules
  (e.g. `navable://docs/fix-patterns/image-alt,label,color-contrast`). Pass comma-separated rule IDs
  from the scan.
- `navable://docs/fix-patterns` — all 55 rules (~49 KB). Only load if you need the full reference.
- `navable://docs/aria-patterns/{patternSlug}` — single ARIA widget pattern detail
- `navable://docs/aria-patterns` — index of 25 patterns (compact list)
- `navable://docs/semantic-html/{element}` — single element detail
- `navable://docs/semantic-html` — index of all elements (compact list)
- `navable://docs/wcag-mapping` — WCAG 2.1 AA mapping table (compact)
- `navable://docs/bfsg-legal` — BFSG legal context, glossary, enforcement (optional, for German
  compliance)

## `.navable-plan.json` Format Reference

```json
{
  "planId": "navable-plan-<timestamp>",
  "url": "http://localhost:3000/...",
  "items": [
    {
      "id": "fix-1",
      "ruleId": "image-alt",
      "impact": "critical",
      "priority": 1,
      "wcagSc": ["1.1.1"],
      "en301549": "9.1.1.1",
      "help": "Images must have alternate text",
      "affectedNodes": [
        { "selector": "img.hero", "html": "<img src=\"...\">", "failureSummary": "..." }
      ],
      "fixDescription": "...",
      "status": "pending",
      "appliedAt": null
    }
  ],
  "manualReview": [],
  "verification": null
}
```

## Gotchas

- **Always use `scanId` for chaining.** After `run_accessibility_scan`, pass `scanId` (not the full
  scan object) to `generate_fix_plan`. This avoids serialization issues with large payloads.
- **`run_accessibility_scan` requires a running server.** The URL must be reachable. If it fails,
  ask the user to confirm their dev server is running.
- **CSS selectors may not map 1:1 to source files.** Use `html` snippets from `affectedNodes` to
  locate components. Search for unique strings (class names, text content, attributes) in the
  codebase.
- **The plan is sorted server-side.** Do not re-sort items by your own logic. Work through them in
  array order.
- **Use `update_fix_status` after each fix**, not manual file edits. This ensures correct JSON
  formatting and enables resumability if the session is interrupted.
- **`incomplete` items in the scan** are issues axe cannot determine automatically. They appear in
  `plan.manualReview`. Mention them to the user but do not auto-fix them.
- **Framework detection:** Check `package.json` for react, vue, svelte, angular to choose the right
  fix patterns from the guides.
