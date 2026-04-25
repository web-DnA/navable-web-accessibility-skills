# @navable/skills

[navable.io](https://navable.io/) |
[MCP Server](https://github.com/web-DnA/navable-web-accessibility-mcp) |
[npm](https://www.npmjs.com/package/@navable/mcp)

Agent skills that teach AI coding agents how to scan, fix, audit, and review WCAG 2.1 Level A + AA
accessibility issues using the
[navable MCP server](https://github.com/web-DnA/navable-web-accessibility-mcp).

## Skills

| Skill                  | Description                                                            |
| ---------------------- | ---------------------------------------------------------------------- |
| `scan-accessibility`   | Scan a URL, generate a fix plan, apply fixes, and verify               |
| `fix-accessibility`    | Resume an existing `.navable-plan.json` and work through pending items |
| `review-component`     | Review a component's source code for accessibility issues (no browser) |
| `audit-page-structure` | Audit landmarks, heading hierarchy, navigation, and document metadata  |

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

## License

MIT
