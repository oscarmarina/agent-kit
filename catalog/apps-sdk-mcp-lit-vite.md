# Domain Profile: Apps SDK + MCP + Lit + Vite + TypeScript

**Domain:** Web Apps
**Stack:** Apps SDK + MCP + Lit + Vite + TypeScript
**Standards:** MCP Apps UI bridge, JSON-RPC 2.0, OWASP input validation basics

## Selection Metadata (Operational Contract)

**Profile ID:** apps-sdk-mcp-lit-vite
**Match Keywords:** openai apps sdk, mcp apps, model context protocol, lit, vite, iframe, postMessage, widgetState
**Use When:** Select this profile for ChatGPT or MCP Apps widgets built with Lit and bundled with Vite behind an MCP server.
**Do Not Use When:** Do not use for React-only, Angular-only, or non-web MCP servers that do not render an iframe UI.

## Terminology Mapping

| Framework Term | Domain Term | Notes |
|---|---|---|
| Build/Compile | `npm run build` | Produces widget assets and server output |
| Test suite | `npm test` | Vitest with two projects: `server` (node) and `widget` (browser via Playwright) |
| Dev server | `npm run dev` | Runs Vite watch and the MCP HTTP server together |
| Package/dependency | npm package | Runtime and build-time dependencies |
| Import/module | ES module / TypeScript module | Node uses ESM output |
| Deployment | MCP HTTP server deployment | Expose `/mcp` to ChatGPT via HTTPS tunnel or hosting |

## Verification Commands

**GATE 0 (Dependencies):**
- Command: `npm install`
- Expected output: lockfile created or updated, exit code 0, no unresolved dependency errors

**GATE 1 (Scaffold):**
- Command: `npm run build`
- Expected output: TypeScript server build succeeds, Vite outputs widget assets with stable names (`assets/index.js`), `dist/` exists

**GATE 2 (Feature):**
- Command: `npm run build && npm test`
- Expected output: build passes, server tests (node) and widget tests (browser) pass with no regressions

**GATE 3 (Tests):**
- Command: `npm test`
- Expected output: all tests pass across both projects (server + widget)
- Coverage command: `npm test -- --coverage`

**GATE 4 (Final):**
- Clean command (POSIX): `rm -rf dist node_modules && npm install && npm run build && npm test`
- Clean command (PowerShell): `if (Test-Path dist) { Remove-Item dist -Recurse -Force }; if (Test-Path node_modules) { Remove-Item node_modules -Recurse -Force }; npm install; npm run build; npm test`
- Expected output: clean install, build, and tests all pass from scratch

## Common Pitfalls

### Pitfall 1: Custom postMessage bridge instead of the official SDK
- **What goes wrong:** Building a custom JSON-RPC bridge with `document.referrer` for origin detection and strict `event.origin` validation. ChatGPT's sandbox iframe (`*.web-sandbox.oaiusercontent.com`) does **not** set `document.referrer`, so the custom bridge fails to initialize and the widget stays blank. The official SDK uses `postMessage("*")` and validates only `event.source`, which works in all host environments.
- **Correct approach:** Use `App` + `PostMessageTransport` from `@modelcontextprotocol/ext-apps`. The SDK handles the initialize handshake, tool calls (`app.callServerTool`), notifications (`app.ontoolresult`), teardown, and auto-resize. Never reimplement the bridge manually.
- **Detection:** `rg "document\.referrer" src/widget` — should have **no** matches. `rg "import.*@modelcontextprotocol/ext-apps" src/widget` — should find the SDK import.

### Pitfall 2: Mixing UI state into authoritative tool payloads
- **What goes wrong:** Sort state, selected row state, or optimistic flags leak into `structuredContent`, making the model consume transient UI details as business truth.
- **Correct approach:** Return only authoritative task data in `structuredContent`; keep view state in local state and `window.openai.widgetState`.
- **Detection:** Search tool results for UI-only fields like `selected`, `draft`, `expanded`, or `filter`.

### Pitfall 3: Shipping Vite assets with absolute paths
- **What goes wrong:** The built widget references `/assets/...`, which breaks when the widget is served as an MCP resource instead of a root-hosted site.
- **Correct approach:** Set Vite `base: './'` and use stable asset names (`entryFileNames: 'assets/index.js'`) for deterministic builds. Serve index plus asset files from matching resource or HTTP paths.
- **Detection:** Inspect built HTML for `src="/` or `href="/` after `npm run build`.

### Pitfall 4: Inlining JS/CSS into MCP resource HTML
- **What goes wrong:** The widget HTML resource includes all JavaScript inline via `<script type="module">` tags. ChatGPT's iframe CSP blocks inline script execution, resulting in a blank/black screen.
- **Correct approach:** Generate the HTML resource with external `<script>` references pointing to the server's `/widget/` endpoint (e.g., `src="${publicOrigin}/widget/assets/index.js"`). Include the server domain in `_meta.ui.csp.connectDomains` and `resourceDomains` so CSP allows loading.
- **Detection:** Check that resource HTML contains `src="http` (external reference), not inline `<script>` blocks with bundled code.

### Pitfall 5: Empty CSP `connectDomains` in widget metadata
- **What goes wrong:** The MCP resource `_meta.ui.csp.connectDomains` is empty (`[]`), so the ChatGPT iframe cannot load scripts or make requests to the MCP server.
- **Correct approach:** Include the server's public origin in both `connectDomains` and `resourceDomains`. For ChatGPT compatibility, also set `openai/widgetCSP.connect_domains`.
- **Detection:** `rg "connectDomains" src/server` — should show arrays containing the server domain, not empty arrays.

### Pitfall 6: Missing body background in widget HTML
- **What goes wrong:** The widget HTML has no `<body>` or root-level background style. If the Lit component fails to initialize (bridge error, JS load failure), the iframe shows a transparent or black background.
- **Correct approach:** Always include a `<style>` in the widget HTML with `body { margin: 0; padding: 0; background: <fallback-color>; }`.
- **Detection:** Check built `index.html` and resource HTML for body style rules.

### Pitfall 8: Stateful `StreamableHTTPServerTransport` with per-request server instances
- **What goes wrong:** The transport generates session IDs by default (stateful mode). Since the server creates a new transport per HTTP request, the session ID from `initialize` is lost on the next request. The MCP inspector (and any stateless HTTP client) gets "Failed to fetch" / connection errors.
- **Correct approach:** Set `sessionIdGenerator: undefined` in the `StreamableHTTPServerTransport` options to disable sessions (stateless mode) when creating a new server+transport per request.
- **Detection:** `rg "sessionIdGenerator" src/server` — should find `sessionIdGenerator: undefined`. If missing, the transport defaults to stateful mode.

### Pitfall 9: Wrong `start` script path after TypeScript server build
- **What goes wrong:** `tsconfig.server.json` uses `rootDir: "src"` so `src/server/index.ts` compiles to `dist/server/server/index.js`, but `package.json` has `"start": "node dist/server/index.js"` — missing the extra `server/` segment.
- **Correct approach:** Match the `start` script path to the actual build output. With `rootDir: "src"` and `outDir: "dist/server"`, the entry point is `dist/server/server/index.js`.
- **Detection:** Run `npm start` — if it fails with `MODULE_NOT_FOUND`, the path is wrong. Check `ls dist/server/` to verify the actual structure.

### Pitfall 10: Missing CORS headers on widget asset endpoints
- **What goes wrong:** `<script type="module">` always uses CORS mode for cross-origin requests. ChatGPT loads widget HTML inside an iframe on `*.web-sandbox.oaiusercontent.com`, which fetches scripts from the MCP server origin (e.g., ngrok). Without `Access-Control-Allow-Origin` on the `/widget/*` endpoint, the browser blocks the script load silently. The custom element never registers and the `<task-board-app>` tag remains empty (no shadow DOM).
- **Correct approach:** Add `"access-control-allow-origin": "*"` to all `/widget/*` static asset responses. This is required even though the MCP `/mcp` endpoint already has CORS headers — widget assets are a separate route.
- **Detection:** `rg "access-control-allow-origin" src/server` — should match in both the MCP handler and the widget asset handler. If only the MCP handler has CORS, widget scripts will fail to load cross-origin.

### Pitfall 11: Resource HTML references a CSS asset that Vite does not emit
- **What goes wrong:** The resource HTML includes `<link rel="stylesheet" href="${publicOrigin}/widget/assets/index.css">`, but Vite does not emit a separate CSS file when all styles live inside Lit's `static styles` (or when `cssCodeSplit: true` produces no extractable CSS). The `<link>` triggers a 404, which may block rendering or cause a flash of unstyled content depending on the browser's error handling.
- **Correct approach:** Only reference CSS assets in the resource HTML if the Vite build actually emits them. After `npm run build`, check `dist/widget/assets/` for `.css` files. If Lit `static styles` handles all styling, omit the `<link>` tag entirely. If a CSS file is needed (e.g., global resets), configure Vite to emit it with a stable name (`assetFileNames: 'assets/[name].[ext]'`).
- **Detection:** After `npm run build`, run `ls dist/widget/assets/*.css 2>/dev/null || echo "NO CSS EMITTED"`. If no CSS file exists but the resource HTML references one, the widget will 404 on that asset.

### Pitfall 7: Using jsdom for widget tests
- **What goes wrong:** Widget tests use jsdom which lacks real browser APIs (Shadow DOM, Custom Elements, postMessage origin). Tests pass but miss real rendering and security issues.
- **Correct approach:** Use Vitest browser mode with Playwright (`@vitest/browser-playwright`). Configure a workspace with `server` (node environment) and `widget` (browser environment) projects.
- **Detection:** Check `vitest.config.ts` for `browser.enabled: true` and `provider: playwright()`. No `environment: 'jsdom'` references.

## Adversary Questions

Answer these BEFORE writing code. Each was born from a real bug in this stack.

- What happens if `document.referrer` is empty inside the iframe? (ChatGPT sandbox strips it.)
- What happens if the widget script is loaded cross-origin without CORS headers? (`<script type="module">` always uses CORS mode.)
- What happens if the MCP transport creates a new session per request but the client expects session continuity?
- What happens if Vite hashes the asset filename and the server generates the resource HTML at startup?
- What happens if the iframe CSP blocks inline `<script>` tags?
- What happens if the resource HTML references a CSS asset (`assets/index.css`) that Vite never emits? (Lit `static styles` keeps CSS in JS — no extracted file.)
- Does the installed SDK already provide the feature I'm about to build? (Read the exports before implementing.)

## Integration Rules

### Data Flow: Widget to MCP server
- The widget calls `tools/call` over the MCP Apps JSON-RPC bridge.
- Arguments must be JSON-serializable and validated again by the server.

### Data Flow: MCP server to Widget
- Tools return authoritative task snapshots in `structuredContent` only.
- The widget re-renders from the returned task list and reapplies local widget state.

### Widget HTML Resource Delivery
- The server generates minimal HTML with external script references to `/widget/assets/index.js`.
- The script tag uses the server's public origin (resolved from request headers or env vars).
- The `_meta.ui.csp` includes the server origin in `connectDomains` and `resourceDomains`.
- Widget assets are served statically from `dist/widget/` via the `/widget/*` HTTP route.

### Type Boundaries
- Validate tool input with Zod before mutating task state.
- Parse persisted storage through schemas before use.
- Treat JSON-RPC `event.data` as unknown until validated.

### Build/Compile Scoping
- Vite only builds widget code under `src/widget/`.
- TypeScript server build only compiles `src/server/` and shared types.
- Vite outputs stable asset names for deterministic resource HTML generation.

### Testing Configuration
- Vitest workspace with two projects:
  - `server` — node environment, tests in `src/server/**/*.test.ts`
  - `widget` — browser environment via Playwright, tests in `src/widget/**/*.test.ts`
- Never use jsdom for widget tests; always use real browser via `@vitest/browser-playwright`.
- Widget bridge tests validate message source, origin, JSON-RPC compliance, and error handling.

### Startup/Bootstrap Order
- HTTP server starts first and serves widget assets plus `/mcp`.
- ChatGPT loads the registered UI resource in an iframe.
- Widget initializes the JSON-RPC bridge with `ui/initialize` before calling tools.

## Automated Checks

| Check | Command | Expected Result |
|-------|---------|-----------------|
| Uses official SDK bridge | `rg "import.*@modelcontextprotocol/ext-apps" src/widget` | SDK import exists — no custom postMessage bridge |
| No document.referrer usage | `rg "document\.referrer" src/widget` | No matches — referrer is unreliable in sandbox iframes |
| No UI-only fields in structured content | `rg "selected\|expanded\|draft\|filter" src/server` | No UI-only state inside tool result payload builders |
| Relative Vite assets configured | `rg "base:\s*['\"]\./['\"]" vite.config.ts` | One matching `base: './'` config |
| Stable asset names configured | `rg "entryFileNames" vite.config.ts` | Stable `assets/index.js` entry name |
| CSP includes server domain | `rg "connectDomains" src/server` | Arrays contain the server domain, not empty |
| Widget tests use browser env | `rg "browser:" vitest.config.ts` | Browser mode enabled for widget project |
| No jsdom in test config | `rg "jsdom" vitest.config.ts` | No matches |
| External script in resource HTML | `rg "src=.*publicOrigin\|src=.*widgetDomain" src/server` | Script references use full server URL |
| Stateless transport | `rg "sessionIdGenerator" src/server` | `sessionIdGenerator: undefined` is set |
| Start script matches build output | `node -e "require('child_process').execSync('node dist/server/server/index.js --help', {timeout:2000})"` | No MODULE_NOT_FOUND error |
| CORS on widget assets | `rg "access-control-allow-origin" src/server` | Header present in both MCP handler and widget asset handler |
| CSS asset references match build output | `ls dist/widget/assets/*.css 2>/dev/null \|\| echo "NO CSS"` then `rg "index\.css" src/server` | If server references `index.css`, the file must exist in build output |

## Decision History

| Date | Decision | Context | Constraint |
|------|----------|---------|------------|
| 2026-03-08 | `structuredContent` is the only UI data contract | MCP Apps widgets should remain portable and model-visible data should be authoritative only | MUST keep widget rendering data in `structuredContent` and keep UI-only state out of tool results |
| 2026-03-08 | Use external script references in widget resource HTML, not inline JS | ChatGPT iframe CSP blocks inline scripts; external references via `src="${origin}/widget/..."` with CSP `connectDomains` work | MUST NOT inline JS/CSS into resource HTML; MUST include server domain in CSP meta |
| 2026-03-08 | Use Vitest browser mode (Playwright) for widget tests, not jsdom | jsdom lacks real Shadow DOM, Custom Elements, and postMessage origin validation; browser tests catch real rendering issues | MUST use `@vitest/browser-playwright` for widget tests; MUST NOT use jsdom |
| 2026-03-08 | Use stable Vite asset names for deterministic builds | Hashed filenames change per build; stable names (`assets/index.js`) allow the server to generate resource HTML without probing the filesystem | SHOULD configure `rollupOptions.output.entryFileNames` for stable names |
| 2026-03-08 | Use stateless `StreamableHTTPServerTransport` (`sessionIdGenerator: undefined`) | Per-request server+transport pattern is incompatible with stateful sessions; MCP inspector and ChatGPT both work with stateless mode | MUST set `sessionIdGenerator: undefined` when creating transport per request |
| 2026-03-08 | Add CORS headers to `/widget/*` asset responses | `<script type="module">` uses CORS mode; ChatGPT sandbox origin differs from MCP server origin; without CORS the script is blocked silently | MUST include `Access-Control-Allow-Origin` on widget asset endpoints, not just `/mcp` |
| 2026-03-09 | Use official SDK `App` + `PostMessageTransport` instead of custom bridge | Custom bridge used `document.referrer` for origin detection which fails in ChatGPT sandbox (referrer is empty). The SDK uses `postMessage("*")` + `event.source` validation which works in all hosts | MUST use `@modelcontextprotocol/ext-apps` App class for widget-host communication; MUST NOT use `document.referrer` or implement custom JSON-RPC bridge |
| 2026-03-09 | Only reference CSS assets in resource HTML if Vite actually emits them | Lit `static styles` keeps all CSS inside JS bundles — Vite produces no separate `.css` file. A `<link>` to a non-existent CSS asset causes a 404 | MUST verify `dist/widget/assets/` contains a `.css` file before adding a `<link>` tag to the resource HTML |

## Review Checklist

- [ ] Widget uses official SDK `App` + `PostMessageTransport` — no custom JSON-RPC bridge or `document.referrer`.

- [ ] Every task mutation tool returns a fresh authoritative task snapshot.
- [ ] Built widget HTML uses relative asset references compatible with MCP resources.
- [ ] Widget resource HTML references external scripts, not inline JS.
- [ ] CSP `connectDomains` and `resourceDomains` include the server's public origin.
- [ ] Widget HTML includes a `<body>` background style for graceful fallback.
- [ ] Widget tests run in a real browser environment (Playwright), not jsdom.
- [ ] Widget asset endpoint (`/widget/*`) includes `Access-Control-Allow-Origin` header for cross-origin script loading.
- [ ] `StreamableHTTPServerTransport` uses `sessionIdGenerator: undefined` (stateless mode).
- [ ] `npm start` script path matches the actual TypeScript build output directory.
- [ ] Resource HTML only references CSS assets that Vite actually emits (check `dist/widget/assets/` after build).

## Reference Implementation

*(Optional — this section references project-specific example code. If the folder does not exist in your repo, rely on the patterns documented below and the SDK's published API.)*

The official OpenAI MCP Apps example (server.js + todo-widget.html) demonstrates these key patterns:
- Widget bridge uses `postMessage('*')` — never validates `event.origin`, only `event.source === window.parent`.
- No `document.referrer` usage anywhere.
- Inline HTML works for simple widgets; external script refs are needed for bundled frameworks (Lit, React).
- `structuredContent.tasks` is the authoritative data contract in tool results.
- `sessionIdGenerator: undefined` for stateless per-request transport.

## Constraints and Standards

- Prefer MCP Apps standard keys and `ui/*` bridge methods over ChatGPT-only APIs.
- Use `window.openai` only for optional ChatGPT-specific capabilities such as widget state persistence.
- Avoid browser-only globals in server-side tests.
- Never inline bundled JS/CSS into the MCP resource HTML; use external script references when using a bundled framework (Lit, React). Simple widgets without build tooling can use inline scripts.
- Always include the server domain in CSP `connectDomains` and `resourceDomains` when using external script references.
- Use Vitest browser mode with Playwright for widget tests; never use jsdom.
- Never use `document.referrer` for parent origin detection — ChatGPT sandbox does not set it. Use the official SDK (`App` + `PostMessageTransport`) or manual `postMessage('*')` with `event.source` validation only.
