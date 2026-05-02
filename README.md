# @navable/skills

Agent skills that teach AI coding agents how to scan, fix, audit, and review WCAG 2.1 Level A + AA
accessibility issues using the
[navable MCP server](https://github.com/web-DnA/navable-web-accessibility-mcp)
([npm](https://www.npmjs.com/package/@navable/mcp)).

Learn more about what we're building at [navable.io](https://navable.io/).

## Skills

| Skill                  | Description                                                            |
| ---------------------- | ---------------------------------------------------------------------- |
| `scan-accessibility`   | Scan a URL, generate a fix plan, apply fixes, and verify               |
| `fix-accessibility`    | Resume an existing `.navable-plan.json` and work through pending items |
| `review-component`     | Review a component's source code for accessibility issues (no browser) |
| `audit-page-structure` | Audit landmarks, heading hierarchy, navigation, and document metadata  |

## Scan engines

**By default, only axe-core runs.** A plain "scan this URL" request uses axe-core only — fast, low
token cost, suitable for iterative fix loops.

Pa11y/HTMLCS is a second engine that can be enabled for higher-confidence scans. Use it when:

- Running a one-shot compliance audit (BFSG, EN 301 549 sign-off)
- The user explicitly asks for a thorough or dual-engine scan

The `audit-page-structure` skill enables both engines automatically. All other skills use axe-only
by default.

### How to enable Pa11y for all scans

Add to `.navable.json` in your project root:

```json
{
  "engines": ["axe", "htmlcs"]
}
```

Or pass it per scan:

```
run_accessibility_scan({ url: "http://localhost:3000", engines: ["axe", "htmlcs"] })
```

When both engines run, crossover findings are deduped server-side — the same bug found by both
engines appears once, tagged `alsoFlaggedBy: ["htmlcs"]`. HTMLCS-only findings include a `helpUrl`
and `developerNote` so agents know how to act on them. Adds ~2–4 s wall-clock per scan.

> **Heads up:** the same DOM element is often flagged by both engines under different WCAG criteria
> (e.g. a `<select>` without a label: SC 3.3.2 _and_ SC 4.1.2 _and_ SC 1.3.1). These are real,
> separate audit findings and are intentionally kept distinct — collapsing them would lose
> compliance traceability. The `scan-accessibility` and `fix-accessibility` skills include guidance
> to **group plan items by DOM element** before editing, so one HTML change resolves all related fix
> IDs in a single pass instead of re-editing the same element repeatedly.

## Setup

### 1. Install the navable MCP server

The skills require the navable MCP server. See the
[MCP repo](https://github.com/web-DnA/navable-web-accessibility-mcp) for setup instructions, or
install directly from npm:

```bash
npm install @navable/mcp
```

### 2. Copy skills to your project

Copy the `skills/` directory into your project's IDE-specific skills folder:

```bash
# VS Code (Copilot)
cp -r skills/* your-project/.github/skills/

# Claude Code
cp -r skills/* your-project/.claude/skills/

# Cursor
cp -r skills/* your-project/.agents/skills/
```

## Skill structure

```
skills/
├── scan-accessibility/
│   ├── SKILL.md              # Full scan → fix → verify workflow
│   └── references/
│       ├── fix-guide-aria.md
│       ├── fix-guide-color.md
│       ├── fix-guide-forms.md
│       ├── fix-guide-headings.md
│       ├── fix-guide-images.md
│       ├── fix-guide-keyboard.md
│       ├── fix-guide-landmarks.md
│       ├── fix-guide-language.md
│       ├── fix-guide-navigation.md
│       └── fix-guide-tables.md
├── fix-accessibility/
│   └── SKILL.md              # Resume and execute existing fix plans
├── review-component/
│   └── SKILL.md              # Source code review (no browser needed)
└── audit-page-structure/
    └── SKILL.md              # Landmarks, headings, navigation audit
```

## Requirements

- [navable MCP server](https://github.com/web-DnA/navable-web-accessibility-mcp) configured in your
  IDE
- Playwright Chromium (`npx playwright install chromium`) for scan/audit skills
- A running dev server for URL-based scanning

## Contributing

We'd love your feedback, ideas, and contributions! If you run into issues, have suggestions, or want
to add new fix guides — feel free to
[open an issue](https://github.com/web-DnA/navable-web-accessibility-skills/issues) or submit a pull
request. Every bit helps make the web more accessible for everyone.

Check out [navable.io](https://navable.io/) to see what else we're working on.

## License

MIT
