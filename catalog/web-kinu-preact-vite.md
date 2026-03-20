# Domain Profile: Web App — Kinu + Preact + Vite

**Domain:** Web Frontend
**Stack:** Kinu UI toolkit + Preact + Vite + TypeScript
**Standards:** Semantic HTML, CSS custom properties, WCAG 2.1 basics

## Selection Metadata (Operational Contract)

**Profile ID:** web-kinu-preact-vite
**Match Keywords:** kinu, preact, vite, dashboard, ui toolkit, css variables, attribute selectors
**Use When:** Select this profile for web apps built with kinu (Preact UI toolkit) and bundled with Vite.
**Do Not Use When:** Do not use for React, Angular, Lit, or projects not using the kinu component library.

## Terminology Mapping

| Framework Term | Domain Term | Notes |
|---|---|---|
| Build/Compile | `pnpm run build` | Vite production build |
| Test suite | `pnpm run build` (type-check + build) | No test framework by default |
| Dev server | `pnpm run dev` | Vite dev server with HMR |
| Package/dependency | npm/pnpm package | kinu is a peer-dep library |
| Import/module | ES module / TypeScript module | Tree-shakeable kinu imports |
| Deployment | Static file hosting | Vite outputs to `dist/` |

## Verification Commands

**GATE 0 (Dependencies):**
- Command: `pnpm install`
- Expected output: lockfile created or updated, exit code 0, no unresolved dependency errors

**GATE 1 (Scaffold):**
- Command: `pnpm run build`
- Expected output: Vite outputs assets to `dist/`, exit code 0

**GATE 2 (Feature):**
- Command: `pnpm run build`
- Expected output: build passes, no TypeScript errors, output artifacts exist

**GATE 3 (Tests):**
- Command: `pnpm run build` (type-check is the minimum verification)
- Expected output: clean build with no errors

**GATE 4 (Final):**
- Clean command (POSIX): `rm -rf dist node_modules && pnpm install && pnpm run build`
- Expected output: clean install and build pass from scratch

## Common Pitfalls

### Pitfall 1: Using `p` attribute instead of `k` attribute
- **What goes wrong:** The AGENTS.md summary mentions `[p="button"]` but kinu actually uses `[k="button"]` as the component identifier attribute. Using `p` produces unstyled elements.
- **Correct approach:** All kinu components use the `k` attribute. CSS selectors target `[k="component-name"]`.
- **Detection:** `rg 'p="' src/` — should have no matches for component identifiers using `p`.

### Pitfall 2: Using `--p-` CSS variable prefix instead of `--k-`
- **What goes wrong:** CSS variables are prefixed `--k-` (e.g., `--k-primary`, `--k-background`), not `--p-`.
- **Correct approach:** Always use `--k-` prefix: `hsl(var(--k-primary))`.
- **Detection:** `rg '\-\-p\-' src/` — should have no matches.

### Pitfall 3: Importing kinu CSS files incorrectly
- **What goes wrong:** Forgetting to import `kinu/style.css` (which includes variables.css, base.css, and all component styles) results in unstyled components.
- **Correct approach:** Import `kinu/style.css` once in the app entry point.
- **Detection:** `rg "kinu/style" src/` — should find at least one import.

### Pitfall 4: Using JavaScript for variant defaults instead of CSS
- **What goes wrong:** Destructuring props to add fallback variant values in JS. Kinu's philosophy is CSS-driven: defaults are handled by `:not([variant])` selectors.
- **Correct approach:** Let CSS handle defaults. Pass variant props only when overriding.
- **Detection:** Review component code for `variant = "default"` destructuring patterns.

## Adversary Questions

- Does the app import `kinu/style.css` before rendering any kinu components?
- Are CSS custom properties (`--k-*`) used consistently instead of hardcoded colors?
- Are compound components (Dialog, Tabs, etc.) used with their sub-components correctly?
- Does the Vite config include the `@preact/preset-vite` plugin?

## Integration Rules

### Kinu Component Usage
- Import components directly: `import { Button, Card } from 'kinu'`
- Components forward all HTML attributes to the underlying element
- Variants are set via props that map to HTML attributes: `<Button variant="outline">`
- Compound components use dot notation: `<Dialog.Trigger>`, `<Dialog.Content>`

### CSS Theming
- Override `--k-*` variables on `:root` or a parent element for custom themes
- Dark mode: use `@media (prefers-color-scheme: dark)` or `.dark` class
- All colors use HSL format: `hsl(var(--k-primary))`

### Build/Compile Scoping
- Vite builds the app; kinu is consumed as a pre-built npm package
- `@preact/preset-vite` handles JSX transformation and Preact aliasing

## Automated Checks

| Check | Command | Expected Result |
|-------|---------|-----------------|
| Kinu style imported | `rg "kinu/style" src/` | At least one import of kinu/style.css |
| No hardcoded colors | `rg "#[0-9a-fA-F]{3,8}" src/ --glob "*.css"` | Minimal matches (only in custom overrides if any) |
| Preact preset configured | `rg "preset-vite\|preact()" vite.config.ts` | Plugin present |

## Decision History

| Date | Decision | Context | Constraint |
|------|----------|---------|------------|
| 2026-03-19 | Use pnpm as package manager | Consistent with kinu's own development setup | SHOULD use pnpm for kinu projects |
| 2026-03-19 | Import kinu/style.css in entry point | All component styles bundled in one CSS entry | MUST import kinu/style.css before rendering |

## Review Checklist

- [ ] `kinu/style.css` is imported in the app entry point
- [ ] All kinu components imported from `'kinu'`
- [ ] CSS custom properties use `--k-` prefix
- [ ] Vite config includes `@preact/preset-vite`
- [ ] No JavaScript-based variant defaults where CSS handles it
- [ ] Compound components use proper sub-component API

## Constraints and Standards

- Peer dependency: preact ^10.26.8
- kinu components use `k` attribute for CSS targeting
- CSS-first approach: prefer attribute selectors and CSS custom properties over JS logic
- Tree-shakeable: import only needed components
