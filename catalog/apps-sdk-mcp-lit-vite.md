# Domain Profile: Apps SDK + MCP + Lit + Vite + TypeScript

**Domain:** Web Apps
**Stack:** Apps SDK + MCP + Lit + Vite + TypeScript
**Standards:** MCP Apps UI bridge, JSON-RPC 2.0, OWASP input validation basics

## Selection Metadata (Operational Contract)

**Profile ID:** apps-sdk-mcp-lit-vite
**Profile Version:** 1.0.0
<!-- Bump when pitfalls, commands, or overrides change. Profile links record this as `catalog_version` to detect drift. Use semver: patch = typo/clarification, minor = additive (new pitfall), major = breaking (renamed/removed field). -->

**Match Keywords:** openai apps sdk, mcp apps, model context protocol, lit, vite, iframe, postMessage, widgetState
**Use When:** Select this profile for ChatGPT or MCP Apps widgets built with Lit and bundled with Vite behind an MCP server.
**Do Not Use When:** Do not use for React-only, Angular-only, or non-web MCP servers that do not render an iframe UI.
**Last Verified:** 2026-04-19
<!-- Update this date whenever you modify this profile or re-verify pitfalls/checks against the codebase -->

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

<!-- Pre-traceability note: pitfalls 1–11 were created during initial catalog authoring (2026-03-08/09).
     Each originated from a real project failure described in its entry, but exact verification-log
     references were not captured at the time. They are therefore recorded as Confidence: inferred,
     not confirmed. New pitfalls added after this date must include Source and Confidence fields per
     DOMAIN_PROFILE-template.md, and confirmed entries must reference the exact verification-log section. -->

### Pitfall 1: Custom postMessage bridge instead of the official SDK
- **Severity:** critical
- **Occurrence count:** 1
- **Confidence:** inferred
- **Source:** Initial catalog entry 2026-03-09 — pre-traceability; real failure: widget blank in ChatGPT sandbox due to empty `document.referrer`
- **What goes wrong:** Building a custom JSON-RPC bridge with `document.referrer` for origin detection and strict `event.origin` validation. ChatGPT's sandbox iframe (`*.web-sandbox.oaiusercontent.com`) does **not** set `document.referrer`, so the custom bridge fails to initialize and the widget stays blank. The official SDK uses `postMessage("*")` and validates only `event.source`, which works in all host environments.
- **Correct approach:** Use `App` + `PostMessageTransport` from `@modelcontextprotocol/ext-apps`. The SDK handles the initialize handshake, tool calls (`app.callServerTool`), notifications (`app.ontoolresult`), teardown, and auto-resize. Never reimplement the bridge manually.
- **Detection:** `rg "document\.referrer" src/widget` — should have **no** matches. `rg "import.*@modelcontextprotocol/ext-apps" src/widget` — should find the SDK import.

### Pitfall 2: Mixing UI state into authoritative tool payloads
- **Severity:** major
- **Occurrence count:** 1
- **Confidence:** inferred
- **Source:** Initial catalog entry 2026-03-08 — pre-traceability; real failure: sort/selected state leaked into `structuredContent`
- **What goes wrong:** Sort state, selected row state, or optimistic flags leak into `structuredContent`, making the model consume transient UI details as business truth.
- **Correct approach:** Return only authoritative task data in `structuredContent`; keep view state in local state and `window.openai.widgetState`.
- **Detection:** Search tool results for UI-only fields like `selected`, `draft`, `expanded`, or `filter`.

### Pitfall 3: Shipping Vite assets with absolute paths
- **Severity:** critical
- **Occurrence count:** 1
- **Confidence:** inferred
- **Source:** Initial catalog entry 2026-03-08 — pre-traceability; real failure: widget resources broken when served from non-root MCP path
- **What goes wrong:** The built widget references `/assets/...`, which breaks when the widget is served as an MCP resource instead of a root-hosted site.
- **Correct approach:** Set Vite `base: './'` and use stable asset names (`entryFileNames: 'assets/index.js'`) for deterministic builds. Serve index plus asset files from matching resource or HTTP paths.
- **Detection:** Inspect built HTML for `src="/` or `href="/` after `npm run build`.

### Pitfall 4: Inlining JS/CSS into MCP resource HTML
- **Severity:** critical
- **Occurrence count:** 1
- **Confidence:** inferred
- **Source:** Initial catalog entry 2026-03-08 — pre-traceability; real failure: blank/black screen due to ChatGPT iframe CSP blocking inline scripts
- **What goes wrong:** The widget HTML resource includes all JavaScript inline via `<script type="module">` tags. ChatGPT's iframe CSP blocks inline script execution, resulting in a blank/black screen.
- **Correct approach:** Generate the HTML resource with external `<script>` references pointing to the server's `/widget/` endpoint (e.g., `src="${publicOrigin}/widget/assets/index.js"`). Include the server domain in `_meta.ui.csp.connectDomains` and `resourceDomains` so CSP allows loading.
- **Detection:** Check that resource HTML contains `src="http` (external reference), not inline `<script>` blocks with bundled code.

### Pitfall 5: Empty CSP `connectDomains` in widget metadata
- **Severity:** critical
- **Occurrence count:** 1
- **Confidence:** inferred
- **Source:** Initial catalog entry 2026-03-08 — pre-traceability; real failure: ChatGPT iframe blocked script loads due to empty `connectDomains`
- **What goes wrong:** The MCP resource `_meta.ui.csp.connectDomains` is empty (`[]`), so the ChatGPT iframe cannot load scripts or make requests to the MCP server.
- **Correct approach:** Include the server's public origin in both `connectDomains` and `resourceDomains`. For ChatGPT compatibility, also set `openai/widgetCSP.connect_domains`.
- **Detection:** `rg "connectDomains" src/server` — should show arrays containing the server domain, not empty arrays.

### Pitfall 6: Missing body background in widget HTML
- **Severity:** minor
- **Occurrence count:** 1
- **Confidence:** inferred
- **Source:** Initial catalog entry 2026-03-08 — pre-traceability; real failure: transparent/black iframe on bridge initialization error
- **What goes wrong:** The widget HTML has no `<body>` or root-level background style. If the Lit component fails to initialize (bridge error, JS load failure), the iframe shows a transparent or black background.
- **Correct approach:** Always include a `<style>` in the widget HTML with `body { margin: 0; padding: 0; background: <fallback-color>; }`.
- **Detection:** Check built `index.html` and resource HTML for body style rules.

### Pitfall 7: Using jsdom for widget tests
- **Severity:** major
- **Occurrence count:** 1
- **Confidence:** inferred
- **Source:** Initial catalog entry 2026-03-08 — pre-traceability; real failure: tests passed in jsdom but missed Shadow DOM and postMessage origin issues in real browser
- **What goes wrong:** Widget tests use jsdom which lacks real browser APIs (Shadow DOM, Custom Elements, postMessage origin). Tests pass but miss real rendering and security issues.
- **Correct approach:** Use Vitest browser mode with Playwright (`@vitest/browser-playwright`). Configure a workspace with `server` (node environment) and `widget` (browser environment) projects.
- **Detection:** Check `vitest.config.ts` for `browser.enabled: true` and `provider: playwright()`. No `environment: 'jsdom'` references.

### Pitfall 8: Stateful `StreamableHTTPServerTransport` with per-request server instances
- **Severity:** critical
- **Occurrence count:** 1
- **Confidence:** inferred
- **Source:** Initial catalog entry 2026-03-08 — pre-traceability; real failure: MCP inspector "Failed to fetch" due to session ID loss across requests
- **What goes wrong:** The transport generates session IDs by default (stateful mode). Since the server creates a new transport per HTTP request, the session ID from `initialize` is lost on the next request. The MCP inspector (and any stateless HTTP client) gets "Failed to fetch" / connection errors.
- **Correct approach:** Set `sessionIdGenerator: undefined` in the `StreamableHTTPServerTransport` options to disable sessions (stateless mode) when creating a new server+transport per request.
- **Detection:** `rg "sessionIdGenerator" src/server` — should find `sessionIdGenerator: undefined`. If missing, the transport defaults to stateful mode.

### Pitfall 9: Wrong `start` script path after TypeScript server build
- **Severity:** major
- **Occurrence count:** 1
- **Confidence:** inferred
- **Source:** Initial catalog entry 2026-03-08 — pre-traceability; real failure: `MODULE_NOT_FOUND` at startup due to `rootDir: "src"` doubling the path segment
- **What goes wrong:** `tsconfig.server.json` uses `rootDir: "src"` so `src/server/index.ts` compiles to `dist/server/server/index.js`, but `package.json` has `"start": "node dist/server/index.js"` — missing the extra `server/` segment.
- **Correct approach:** Match the `start` script path to the actual build output. With `rootDir: "src"` and `outDir: "dist/server"`, the entry point is `dist/server/server/index.js`.
- **Detection:** Run `npm start` — if it fails with `MODULE_NOT_FOUND`, the path is wrong. Check `ls dist/server/` to verify the actual structure.

### Pitfall 10: Missing CORS headers on widget asset endpoints
- **Severity:** critical
- **Occurrence count:** 1
- **Confidence:** inferred
- **Source:** Initial catalog entry 2026-03-09 — pre-traceability; real failure: custom element never registered, empty shadow DOM due to silent CORS block on `/widget/*`
- **What goes wrong:** `<script type="module">` always uses CORS mode for cross-origin requests. ChatGPT loads widget HTML inside an iframe on `*.web-sandbox.oaiusercontent.com`, which fetches scripts from the MCP server origin (e.g., ngrok). Without `Access-Control-Allow-Origin` on the `/widget/*` endpoint, the browser blocks the script load silently. The custom element never registers and the `<task-board-app>` tag remains empty (no shadow DOM).
- **Correct approach:** Add `"access-control-allow-origin": "*"` to all `/widget/*` static asset responses. This is required even though the MCP `/mcp` endpoint already has CORS headers — widget assets are a separate route.
- **Detection:** `rg "access-control-allow-origin" src/server` — should match in both the MCP handler and the widget asset handler. If only the MCP handler has CORS, widget scripts will fail to load cross-origin.

### Pitfall 11: Resource HTML references a CSS asset that Vite does not emit
- **Severity:** major
- **Occurrence count:** 1
- **Confidence:** inferred
- **Source:** Initial catalog entry 2026-03-09 — pre-traceability; real failure: 404 on `assets/index.css` because Lit `static styles` keeps all CSS inside JS bundle
- **What goes wrong:** The resource HTML includes `<link rel="stylesheet" href="${publicOrigin}/widget/assets/index.css">`, but Vite does not emit a separate CSS file when all styles live inside Lit's `static styles` (or when `cssCodeSplit: true` produces no extractable CSS). The `<link>` triggers a 404, which may block rendering or cause a flash of unstyled content depending on the browser's error handling.
- **Correct approach:** Only reference CSS assets in the resource HTML if the Vite build actually emits them. After `npm run build`, check `dist/widget/assets/` for `.css` files. If Lit `static styles` handles all styling, omit the `<link>` tag entirely. If a CSS file is needed (e.g., global resets), configure Vite to emit it with a stable name (`assetFileNames: 'assets/[name].[ext]'`).
- **Detection:** After `npm run build`, run `ls dist/widget/assets/*.css 2>/dev/null || echo "NO CSS EMITTED"`. If no CSS file exists but the resource HTML references one, the widget will 404 on that asset.

### Pitfall 12: `useDefineForClassFields: true` breaks Lit reactive properties with decorators
- **Severity:** critical
- **Occurrence count:** 1
- **Confidence:** inferred
- **Source:** `openai/openai-apps-sdk reference — tsconfig.json line 12` (2026-04-18 cross-repo audit)
- **What goes wrong:** TypeScript's default for `target: ES2022+` is `useDefineForClassFields: true`, which emits class fields using native `Object.defineProperty` semantics *before* decorators run. Lit's `@property` / `@state` decorators install accessors on the prototype; the native field definition then **shadows** them on each instance, so reactive updates never trigger and the widget renders stale data silently — no error, no warning.
- **Correct approach:** Set `"useDefineForClassFields": false` in the base `tsconfig.json` (alongside `"experimentalDecorators": true`). The reference implementation (`openai-apps-sdk`) uses this exact combination.
- **Detection:** `rg "useDefineForClassFields" tsconfig.json` — must show `false`. If absent when `target` is `ES2022+`, the compiler defaults to `true` and decorators are silently broken.

### Pitfall 13: `escapeHtmlAttribute` omission allows XSS via interpolated URLs in widget HTML
- **Severity:** critical
- **Occurrence count:** 1
- **Confidence:** inferred
- **Source:** `openai/openai-apps-sdk reference — src/server/widget-resource.ts line 66` (2026-04-18 cross-repo audit)
- **What goes wrong:** The server renders widget HTML with template interpolation like `<script src="${publicOrigin}/widget/assets/index.js">`. If `publicOrigin` (or any interpolated value) contains `"`, `<`, `>`, `&`, or `'`, the attribute context is broken and arbitrary HTML/JS can be injected. `publicOrigin` typically comes from `process.env['PUBLIC_ORIGIN']` or request headers — both are attacker-controllable in deployments behind reverse proxies.
- **Correct approach:** Define and apply an `escapeHtmlAttribute` helper on every value interpolated into an HTML attribute:
  ```ts
  const escapeHtmlAttribute = (value: string) =>
    value.replace(/&/g, '&amp;').replace(/"/g, '&quot;').replace(/'/g, '&#39;').replace(/</g, '&lt;').replace(/>/g, '&gt;');
  ```
  Apply to `src=`, `href=`, `data-*`, and any other attribute that takes a server-controlled string.
- **Detection:** `rg "src=\".*\\\$\{" src/server` or similar — every `${...}` inside an attribute must pass through `escapeHtmlAttribute(...)`. Raw interpolation is a fail.

### Pitfall 14: `registerAppTool` / `registerAppResource` imported from wrong package
- **Severity:** critical
- **Occurrence count:** 1
- **Confidence:** inferred
- **Source:** `openai/openai-apps-sdk reference — src/server/server.ts line 1` (2026-04-18 cross-repo audit)
- **What goes wrong:** LLMs default to importing tool/resource registration helpers from `@modelcontextprotocol/sdk/server/mcp.js`. Those helpers do not exist there — `registerAppTool` and `registerAppResource` are exported from `@modelcontextprotocol/ext-apps/server`. Using `server.registerTool(...)` from the base SDK works for plain MCP tools but omits the `_meta.ui` / `_meta['openai/widgetCSP']` wiring that ChatGPT Apps requires, producing a tool that appears to register but renders no UI.
- **Correct approach:** `import {registerAppTool, registerAppResource} from '@modelcontextprotocol/ext-apps/server';` — these are the Apps-aware variants that accept `ui`, `csp`, and widget metadata. Use them for every tool that has an associated widget.
- **Detection:** `rg "registerAppTool|registerAppResource" src/server` — must resolve to `@modelcontextprotocol/ext-apps/server` imports. `rg "server\.registerTool\(" src/server` — if found, verify the tool is intentionally UI-less; otherwise it's the wrong helper.

### Pitfall 15: Missing `safeAreaInsets` CSS variables breaks layout on mobile ChatGPT hosts
- **Severity:** critical
- **Occurrence count:** 1
- **Confidence:** inferred
- **Source:** `openai/openai-apps-sdk reference — src/widget/mcp-task-bridge.ts lines 79–83` (2026-04-18 cross-repo audit)
- **What goes wrong:** ChatGPT's mobile host delivers `context.safeAreaInsets` ({top, right, bottom, left}) through the `ui/host-context` message. Widgets that do not consume these values render underneath the iOS home indicator, notch, or Android system bars — content is clipped or unreachable. No error is emitted; the widget just looks broken on mobile.
- **Correct approach:** On each host-context update, set CSS custom properties on `document.documentElement`:
  ```ts
  if (context.safeAreaInsets) {
    document.documentElement.style.setProperty('--safe-area-top', `${context.safeAreaInsets.top}px`);
    document.documentElement.style.setProperty('--safe-area-right', `${context.safeAreaInsets.right}px`);
    document.documentElement.style.setProperty('--safe-area-bottom', `${context.safeAreaInsets.bottom}px`);
    document.documentElement.style.setProperty('--safe-area-left', `${context.safeAreaInsets.left}px`);
  }
  ```
  Consume via `padding: var(--safe-area-top) var(--safe-area-right) ...` in component styles.
- **Detection:** `rg "safeAreaInsets" src/widget` — should find handler that sets the four CSS variables. `rg "\-\-safe-area-" src/widget` — should find consumers in component styles.

### Pitfall 16: `App.onerror` / `App.onhostcontextchanged` handlers not wired
- **Severity:** major
- **Occurrence count:** 1
- **Confidence:** inferred
- **Source:** `openai/openai-apps-sdk reference — src/widget/mcp-task-bridge.ts lines 20-24` (2026-04-19 cross-repo audit)
- **What goes wrong:** `App` from `@modelcontextprotocol/ext-apps` exposes two optional callbacks: `onerror` (bridge-level failures, e.g. malformed messages, transport drops) and `onhostcontextchanged` (host theme, locale, safeAreaInsets updates). Without `onerror`, bridge errors surface only as unhandled promise rejections in the iframe — invisible to the user and the server. Without `onhostcontextchanged`, the widget cannot react to host theme changes (dark/light mode switches) or safe-area updates after initial load.
- **Correct approach:** Wire both in the bridge constructor:
  ```ts
  this.#app.onerror = (error) => { console.error(error); /* surface via signal */ };
  this.#app.onhostcontextchanged = (context) => { /* update CSS vars, theme signal */ };
  ```
- **Detection:** `rg "app\.onerror\|onhostcontextchanged" src/widget` — both should be assigned in the bridge module.

### Pitfall 17: Tool handlers without `CallToolResult` return type annotation
- **Severity:** major
- **Occurrence count:** 1
- **Confidence:** inferred
- **Source:** `openai/openai-apps-sdk reference — src/server/server.ts` (2026-04-19 cross-repo audit)
- **What goes wrong:** `registerAppTool` does not enforce the return shape at the type level unless the handler declares `Promise<CallToolResult>` explicitly. Handlers that return bare objects `{structuredContent: {...}}` compile, but omit required fields (`content: []`) or return invalid shapes that ChatGPT silently ignores — the tool "succeeds" but produces no visible output.
- **Correct approach:** `import type {CallToolResult} from '@modelcontextprotocol/sdk/types.js';` and annotate every handler:
  ```ts
  async (args): Promise<CallToolResult> => ({ content: [...], structuredContent: {...} })
  ```
- **Detection:** `rg "async.*=>.*Promise<CallToolResult>" src/server` — every tool handler registered via `registerAppTool` must carry this annotation.

### Pitfall 18: `SignalWatcher(LitElement)` extends clause rejected by TypeScript
- **Severity:** major
- **Occurrence count:** 1
- **Confidence:** inferred
- **Source:** `openai/openai-apps-sdk reference — src/widget/task-widget-app.ts line 13` (2026-04-19 cross-repo audit)
- **What goes wrong:** `class X extends SignalWatcher(LitElement)` fails TypeScript strict mode because `SignalWatcher` returns a `Constructor<ReactiveElement & SignalWatcher>` intersection the compiler cannot narrow to a Lit-compatible base class. Results in `TS2510` or missing decorator metadata errors.
- **Correct approach:** Use the reference cast:
  ```ts
  export class TaskWidgetApp extends (SignalWatcher(LitElement) as unknown as typeof LitElement) { ... }
  ```
  The runtime behavior is identical; the cast narrows the returned mixin to a concrete `LitElement` constructor so decorators resolve.
- **Detection:** `rg "SignalWatcher\(LitElement\)" src/widget` — the expression must be followed by `as unknown as typeof LitElement`.

### Pitfall 19: Widget tests skip `await element.updateComplete` flush
- **Severity:** major
- **Occurrence count:** 1
- **Confidence:** inferred
- **Source:** `openai/openai-apps-sdk reference — src/widget/task-widget-app.test.ts lines 93, 95` (2026-04-19 cross-repo audit)
- **What goes wrong:** Lit schedules renders asynchronously. Immediately asserting against `element.shadowRoot` after a property change misses the update and produces false negatives (or worse, flaky passes when the test runner happens to yield first).
- **Correct approach:** After every property mutation or tool-call that triggers re-render, `await element.updateComplete` before asserting:
  ```ts
  element.filter = 'Italian';
  await element.updateComplete;
  expect(element.shadowRoot?.querySelectorAll('.card')).toHaveLength(2);
  ```
- **Detection:** `rg "updateComplete" src/widget` — must appear in every widget test that asserts post-mutation DOM state.

### Pitfall 20: Per-request transport and server not torn down on client disconnect
- **Severity:** major
- **Occurrence count:** 1
- **Confidence:** inferred
- **Source:** `openai/openai-apps-sdk reference — src/server/main.ts line 156` (2026-04-19 cross-repo audit)
- **What goes wrong:** The stateless pattern creates one `McpServer` + `StreamableHTTPServerTransport` per HTTP request. If the client disconnects before the response completes (timeout, tab close, ngrok drop), both remain allocated — memory leak + orphaned SSE listeners that can corrupt future requests.
- **Correct approach:** Register a cleanup hook on the response:
  ```ts
  response.on('close', () => {
    void transport.close().catch(() => undefined);
    void server.close().catch(() => undefined);
  });
  ```
- **Detection:** `rg "response\.on\(.close" src/server` — must close both `transport` and `server` in the handler.

### Pitfall 21: Widget tests mock `fetch` instead of injecting a fake bridge
- **Severity:** major
- **Occurrence count:** 1
- **Confidence:** heuristic
- **Source:** Preventive — carried from `@modelcontextprotocol/ext-apps` bridge semantics (2026-04-19 cross-repo audit)
- **What goes wrong:** Tests that mock `fetch` or `window.postMessage` to stub tool calls couple the test to the SDK's internal transport. Any SDK update that changes the message framing breaks the tests without any real regression in the widget code.
- **Correct approach:** Inject the bridge. Export the bridge dependency on the component (constructor arg, property, or context) and replace it with a fake in tests. Example: `vi.mock('./mcp-bridge.js', () => ({ McpBridge: FakeBridge }))` where `FakeBridge` exposes the same public methods (`listRecipes`, `getRecipe`, …) returning fixture data synchronously.
- **Detection:** `rg "vi\.mock.*bridge\|McpBridge.*mock" src/widget` — tests should mock the bridge module, not `fetch` or `postMessage`.

### Pitfall 22: Widget resource metadata missing dual CSP shape
- **Severity:** major
- **Occurrence count:** 1
- **Confidence:** inferred
- **Source:** `openai/openai-apps-sdk reference — src/server/widget-resource.ts` (2026-04-19 cross-repo audit)
- **What goes wrong:** The MCP Apps spec and ChatGPT's host read CSP from **two** keys that look redundant but are not: `_meta.ui.csp` (spec-standard, used by any MCP Apps host) and `_meta['openai/widgetCSP']` (ChatGPT-specific, with `connect_domains` / `resource_domains` snake_case). Setting only one works in one host and silently fails in the other.
- **Correct approach:** Emit both in the resource metadata. Keep them in sync — generate from a single source array:
  ```ts
  const connectDomains = [publicOrigin];
  const _meta = {
    ui: { csp: { connectDomains, resourceDomains: connectDomains } },
    'openai/widgetCSP': { connect_domains: connectDomains, resource_domains: connectDomains },
    'openai/widgetDescription': '...',
  };
  ```
- **Detection:** `rg "openai/widgetCSP" src/server` AND `rg "ui.*csp" src/server` — both must populate the same origin list.

### Pitfall 23: Missing `openai/widgetDescription` metadata
- **Severity:** minor
- **Occurrence count:** 1
- **Confidence:** inferred
- **Source:** `openai/openai-apps-sdk reference — src/server/widget-resource.ts line 59` (2026-04-19 cross-repo audit)
- **What goes wrong:** Without `_meta['openai/widgetDescription']`, ChatGPT's model has no summary of what the widget does, which degrades tool-selection quality and inline previews. Not an error — the widget still renders — but the model is more likely to skip the tool or surface it awkwardly.
- **Correct approach:** Set a one-sentence description on the resource metadata, e.g. `'openai/widgetDescription': 'Manage tasks from an embedded Lit widget backed by MCP tools.'`
- **Detection:** `rg "openai/widgetDescription" src/server` — must find a non-empty string on the widget resource.

## Adversary Questions

Answer these BEFORE writing code. Each was born from a real bug in this stack.

- What happens if `document.referrer` is empty inside the iframe? (ChatGPT sandbox strips it.)
- What happens if the widget script is loaded cross-origin without CORS headers? (`<script type="module">` always uses CORS mode.)
- What happens if the MCP transport creates a new session per request but the client expects session continuity?
- What happens if Vite hashes the asset filename and the server generates the resource HTML at startup?
- What happens if the iframe CSP blocks inline `<script>` tags?
- What happens if the resource HTML references a CSS asset (`assets/index.css`) that Vite never emits? (Lit `static styles` keeps CSS in JS — no extracted file.)
- Does the installed SDK already provide the feature I'm about to build? (Read the exports before implementing.)
- What happens on TypeScript `target: ES2022+` with Lit decorators if `useDefineForClassFields` is left at its default `true`? (Native field definitions shadow decorator accessors — reactivity silently breaks.)
- What happens if `publicOrigin` contains `"` or `<` when interpolated into the widget resource HTML? (HTML attribute injection → XSS unless each value is escaped.)
- Where do `registerAppTool` and `registerAppResource` come from? (`@modelcontextprotocol/ext-apps/server` — NOT the base `@modelcontextprotocol/sdk`. Using the wrong import registers a UI-less tool.)
- What happens on mobile ChatGPT if the widget ignores `context.safeAreaInsets`? (Content renders under notches/home indicators — layout is broken without any error signal.)
- What happens when the bridge emits an error (malformed message, transport drop) and `App.onerror` is not wired? (Unhandled promise rejection inside the iframe — invisible to user and server; surface via `onerror` handler.)
- What happens if the host sends a `host-context` update after initial load (theme toggle, orientation change)? (Widget ignores it unless `App.onhostcontextchanged` is wired.)
- What shape does `registerAppTool` actually require the handler to return? (`Promise<CallToolResult>` — `{content: [...], structuredContent: {...}}`. Bare objects compile but produce silently empty output.)
- What happens when the HTTP client disconnects mid-request in the stateless transport pattern? (Per-request `McpServer`+transport leak unless `response.on('close')` closes both.)
- What happens if `cssCodeSplit` is left at its default `true` with a single-entry Vite build? (Vite may still emit per-chunk CSS files with unpredictable names, breaking the resource HTML's stable reference to `assets/index.css`.)
- What happens if the deployment chooses stateful MCP sessions? (Would require a persistent `McpServer` + session store — the per-request pattern documented here is stateless-only. Use a single long-lived `McpServer` and one transport per session id if stateful mode is required.)

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
| useDefineForClassFields disabled for Lit | `rg "useDefineForClassFields" tsconfig.json` | `false` — required for Lit decorator reactivity on ES2022+ |
| HTML attribute escaping present | `rg "escapeHtmlAttribute\|escapeHtml" src/server` | Helper defined and applied to every `${...}` inside an attribute |
| Apps-aware registration imports | `rg "registerAppTool\|registerAppResource" src/server` | Imports resolve to `@modelcontextprotocol/ext-apps/server`, not base SDK |
| safeAreaInsets handled in widget | `rg "safeAreaInsets" src/widget` | Handler sets `--safe-area-*` CSS variables from host context |
| Bridge error + host-context handlers wired | `rg "app\.onerror\|onhostcontextchanged" src/widget` | Both callbacks assigned in the bridge module |
| Tool handlers annotated with CallToolResult | `rg "Promise<CallToolResult>" src/server` | Every `registerAppTool` handler carries the annotation |
| SignalWatcher cast present | `rg "SignalWatcher\(LitElement\)" src/widget` | Expression followed by `as unknown as typeof LitElement` |
| updateComplete flush in widget tests | `rg "updateComplete" src/widget` | Present in every widget test after a state mutation |
| Transport+server cleanup on disconnect | `rg "response\.on\(.close" src/server` | Closes both `transport` and `server` inside handler |
| Bridge injection pattern in widget tests | `rg "vi\.mock.*bridge\|McpBridge" src/widget` | Widget tests mock the bridge module, not `fetch` or `postMessage` |
| Dual CSP metadata present | `rg "openai/widgetCSP" src/server` + `rg "ui.*csp" src/server` | Both populated with the same origin list |
| widgetDescription metadata set | `rg "openai/widgetDescription" src/server` | Non-empty description string on widget resource |
| cssCodeSplit disabled | `rg "cssCodeSplit" vite.config.ts` | `false` — avoids unpredictable per-chunk CSS filenames |

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
| 2026-04-18 | `useDefineForClassFields: false` for Lit + ES2022 decorators | Native field definitions shadow `@property`/`@state` accessors — reactivity breaks silently with no error | MUST set `useDefineForClassFields: false` in `tsconfig.json` when Lit decorators target ES2022+ |
| 2026-04-18 | Escape every interpolated value in widget HTML attributes | `publicOrigin` and similar values can contain attacker-controlled characters; raw interpolation into attributes enables XSS | MUST apply `escapeHtmlAttribute` on every `${...}` inside an HTML attribute in server-rendered widget markup |
| 2026-04-18 | Use `@modelcontextprotocol/ext-apps/server` for tool/resource registration | Apps-aware helpers wire `_meta.ui` and `openai/widgetCSP`; base SDK `registerTool` omits widget metadata and silently registers a UI-less tool | MUST import `registerAppTool` / `registerAppResource` from `@modelcontextprotocol/ext-apps/server` for any tool with an associated widget |
| 2026-04-18 | Propagate `safeAreaInsets` from host context to CSS variables | ChatGPT mobile host delivers insets via `ui/host-context`; ignoring them clips content under notches and home indicators | SHOULD expose `--safe-area-top/right/bottom/left` on `document.documentElement` from host-context updates and consume them in component styles |
| 2026-04-19 | Wire both `App.onerror` and `App.onhostcontextchanged` in the bridge | Bridge errors become silent unhandled rejections without `onerror`; theme/orientation/safe-area updates after init are dropped without `onhostcontextchanged` | MUST assign both callbacks in the bridge constructor |
| 2026-04-19 | Annotate every tool handler with `Promise<CallToolResult>` | `registerAppTool` does not enforce shape at the type level; incorrect return shapes compile but produce silently empty tool output | MUST import `CallToolResult` from `@modelcontextprotocol/sdk/types.js` and annotate every handler |
| 2026-04-19 | Cast `SignalWatcher(LitElement)` for the extends clause | TypeScript strict mode rejects the mixin return type as a Lit base class (`TS2510` / missing decorator metadata) | MUST use `extends (SignalWatcher(LitElement) as unknown as typeof LitElement)` |
| 2026-04-19 | `await element.updateComplete` in widget tests | Lit schedules renders asynchronously — immediate assertions race the update and produce flaky results | MUST await `updateComplete` after every property/state mutation before DOM assertions |
| 2026-04-19 | Close per-request transport and server on client disconnect | Per-request stateless pattern leaks memory and leaves orphaned SSE listeners if client disconnects mid-request | MUST register `response.on('close', ...)` that closes both `transport` and `McpServer` |
| 2026-04-19 | Inject fake bridge in widget tests, do not mock `fetch`/`postMessage` | Mocking SDK internals couples tests to transport shape; SDK updates break tests without real regressions | SHOULD mock the bridge module via `vi.mock` and expose a `FakeBridge` with the same public methods |
| 2026-04-19 | Emit both `_meta.ui.csp` and `_meta['openai/widgetCSP']` on widget resource | Spec-standard key works for generic MCP Apps hosts; ChatGPT-specific key uses snake_case and is required by ChatGPT — setting only one fails silently in the other host | MUST populate both keys from the same origin list |
| 2026-04-19 | Set `openai/widgetDescription` on widget resource metadata | Without a description, the model has no summary of the widget's purpose — degrades tool-selection quality | SHOULD include a one-sentence `openai/widgetDescription` on every widget resource |
| 2026-04-19 | `cssCodeSplit: false` in Vite config | Default `true` emits per-chunk CSS with unpredictable names, breaking the resource HTML's stable reference to `assets/index.css` | SHOULD set `cssCodeSplit: false` for single-entry widget builds so CSS name is deterministic |

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
- [ ] `tsconfig.json` has `useDefineForClassFields: false` when `target` is `ES2022` or later and Lit decorators are in use.
- [ ] Every server-controlled value interpolated into widget HTML attributes passes through `escapeHtmlAttribute`.
- [ ] `registerAppTool` and `registerAppResource` are imported from `@modelcontextprotocol/ext-apps/server`, not the base SDK.
- [ ] Widget consumes `context.safeAreaInsets` and exposes them as `--safe-area-*` CSS variables on `document.documentElement`.
- [ ] Bridge wires both `App.onerror` and `App.onhostcontextchanged`.
- [ ] Every `registerAppTool` handler is annotated `async (...): Promise<CallToolResult> => ...`.
- [ ] `SignalWatcher(LitElement)` extends clause uses the `as unknown as typeof LitElement` cast.
- [ ] Widget tests `await element.updateComplete` after every state mutation before asserting DOM.
- [ ] `response.on('close', ...)` closes both the per-request `transport` and `McpServer` on client disconnect.
- [ ] Widget tests inject a fake bridge (mock the bridge module), not `fetch` or `postMessage`.
- [ ] Resource metadata sets both `_meta.ui.csp` and `_meta['openai/widgetCSP']` with the same origin list.
- [ ] Resource metadata includes a non-empty `openai/widgetDescription`.
- [ ] `vite.config.ts` sets `cssCodeSplit: false`.

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
