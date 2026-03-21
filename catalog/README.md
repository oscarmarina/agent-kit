# Domain Profile Catalog

Community-contributed domain profiles for Agent Kit. Each profile captures stack-specific knowledge — pitfalls, adversary questions, verification commands, and decision history — learned from real projects.

## Available Profiles

| Profile | Stack | Pitfalls | Adversary Qs |
|---------|-------|----------|--------------|
| [apps-sdk-mcp-lit-vite](apps-sdk-mcp-lit-vite.md) | MCP Apps + Lit + Vite + TypeScript | 11 | 7 |
| [web-kinu-preact-vite](web-kinu-preact-vite.md) | Kinu + Preact + Vite + TypeScript | 4 | 4 |

## Using a profile

Create a profile link in your project's `framework/domains/` directory that extends the catalog profile:

```markdown
<!-- framework/domains/my-project.md -->

## Profile Link

extends: web-kinu-preact-vite
catalog_version: 1.0.0

## Local Pitfalls
<!-- Project-specific pitfalls go here -->

## Local Overrides
<!-- Only sections that differ from the base -->
```

The Builder loads the base from `catalog/` and merges your local additions on top. See `framework/domains/_template.md` for the full template and `framework/domains/README.md` for merge rules.

## Contributing a profile

Domain profiles grow from real project experience. If you've built projects with a stack that isn't represented here, consider contributing your profile.

### Requirements

1. Use the template at `framework/templates/DOMAIN_PROFILE-template.md`
2. Follow the naming convention: `[domain]-[stack].md` (e.g., `web-react-nextjs.md`, `backend-python-fastapi.md`)
3. Include at minimum:
   - **Selection Metadata** — Profile ID, Match Keywords, Use When, Do Not Use When
   - **Terminology Mapping** — Stack-specific command translations
   - **Verification Commands** — Exact commands for Gates 0-4
   - **2+ Common Pitfalls** — With What/Correct/Detection pattern
4. Every pitfall and adversary question should come from a real bug or failure — not hypotheticals

### What makes a good profile

- **Specific:** "Vite uses `--k-` CSS variable prefix, not `--p-`" beats "check your CSS variables"
- **Actionable:** Each pitfall has a Detection command that can be run mechanically
- **Honest:** If a pitfall was discovered by making the mistake, say so
- **Growing:** A profile with 3 real pitfalls is more valuable than one with 10 speculative ones

### Submitting

1. Fork the repository
2. Add your profile to `catalog/`
3. Open a pull request with:
   - The stack and domain your profile covers
   - How many projects informed the profile
   - At least one example of a pitfall that prevented a real bug
