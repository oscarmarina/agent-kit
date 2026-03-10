# Verification Log: MCP Task Widget

This log captures the actual output of every verification gate. It is the source of truth for project completion status.

**Rule:** No entry may be written without executing the command and pasting real output. "Assumed to pass" is not an entry.

---

## Progress

**Current phase:** Complete
**Last updated:** 2026-03-09 16:16 local

| Step | Status |
|------|--------|
| Intent | PASS |
| Design | PASS |
| Gate 0: Dependencies | PASS |
| Gate 1: Scaffold | PASS |
| Gate 2: Feature | PASS |
| Gate 3: Tests | PASS |
| Gate 4: Clean build | PASS |
| Self-Review | PASS |
| Domain update | PASS |

**Update this section after every gate or phase transition. When resuming interrupted work, read this section first.**

---

## Gate 0: Dependencies
**Executed:** 2026-03-09 16:15 local
**Command:** `npm install`
**Exit code:** 0
**Status:** PASS

<details>
<summary>Output</summary>

```text
removed 11 packages, and audited 159 packages in 632ms

44 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
```

</details>

**Notes:** This gate was re-run after migrating the HTTP runtime from Express to `node:http`; the install removed direct Express packages from the root manifest while keeping the rest of the dependency graph healthy.

---

## Gate 1: Scaffold Verification
**Executed:** 2026-03-09 16:15 local
**Command:** `npm run build`
**Exit code:** 0
**Status:** PASS

<details>
<summary>Output</summary>

```text
> mcp-task-widget@1.0.0 build
> tsc --noEmit && vite build && tsc -p tsconfig.server.json

vite v7.3.1 building client environment for production...
✓ 243 modules transformed.
dist/widget/index.html          0.45 kB │ gzip:   0.27 kB
dist/widget/assets/index.css    0.21 kB │ gzip:   0.16 kB
dist/widget/assets/index.js   546.29 kB │ gzip: 131.15 kB

(!) Some chunks are larger than 500 kB after minification. Consider:
- Using dynamic import() to code-split the application
- Use build.rollupOptions.output.manualChunks to improve chunking: https://rollupjs.org/configuration-options/#output-manualchunks
- Adjust chunk size limit for this warning via build.chunkSizeWarningLimit.
✓ built in 555ms
```

</details>

**Notes:** Revalidated after replacing Express with `node:http`. Build output remains deterministic and still emits both `assets/index.js` and `assets/index.css` for the MCP resource HTML.

---

## Gate 2: Feature Verification
**Executed:** 2026-03-09 16:16 local
**Command:** `npm run build && npm test`
**Exit code:** 0
**Status:** PASS

<details>
<summary>Output</summary>

```text
> mcp-task-widget@1.0.0 build
> tsc --noEmit && vite build && tsc -p tsconfig.server.json

vite v7.3.1 building client environment for production...
✓ 243 modules transformed.
dist/widget/index.html          0.45 kB │ gzip:   0.27 kB
dist/widget/assets/index.css    0.21 kB │ gzip:   0.16 kB
dist/widget/assets/index.js   546.29 kB │ gzip: 131.15 kB

(!) Some chunks are larger than 500 kB after minification. Consider:
- Using dynamic import() to code-split the application
- Use build.rollupOptions.output.manualChunks to improve chunking: https://rollupjs.org/configuration-options/#output-manualchunks
- Adjust chunk size limit for this warning via build.chunkSizeWarningLimit.
✓ built in 552ms

> mcp-task-widget@1.0.0 test
> vitest run


 RUN  v4.0.18 /Users/oscarmarina/Projects/AGENTS/openai-apps-sdk

 ✓  server  src/server/task-service.test.ts (1 test) 7ms
 ✓  server  src/server/widget-resource.test.ts (2 tests) 1ms
stderr | unknown test
Lit is in dev mode. Not recommended for production! See https://lit.dev/msg/dev-mode for more information.
 ✓  widget (chromium)  src/widget/task-widget-app.test.ts (2 tests) 133ms

 Test Files  3 passed (3)
		Tests  5 passed (5)
	Start at  16:16:23
	Duration  1.39s (transform 47ms, setup 0ms, import 833ms, tests 141ms, environment 0ms)
```

</details>

**Notes:** Revalidated on the `node:http` implementation. The Lit dev-mode stderr is expected in test runs and did not affect correctness.

---

## Gate 3: Test Verification
**Executed:** 2026-03-09 16:15 local
**Command:** `npm test`
**Exit code:** 0
**Status:** PASS
**Tests passed:** 5/5
**Coverage:** not measured

<details>
<summary>Output</summary>

```text
> mcp-task-widget@1.0.0 test
> vitest run


 RUN  v4.0.18 /Users/oscarmarina/Projects/AGENTS/openai-apps-sdk

4:15:35 PM [vite] (client) Re-optimizing dependencies because lockfile has changed
 ✓  server  src/server/task-service.test.ts (1 test) 16ms
 ✓  server  src/server/widget-resource.test.ts (2 tests) 1ms
stderr | unknown test
Lit is in dev mode. Not recommended for production! See https://lit.dev/msg/dev-mode for more information.
 ✓  widget (chromium)  src/widget/task-widget-app.test.ts (2 tests) 149ms

 Test Files  3 passed (3)
		Tests  5 passed (5)
	Start at  16:15:35
	Duration  1.56s (transform 84ms, setup 0ms, import 1.11s, tests 166ms, environment 0ms)
```

</details>

**Notes:** Browser-mode widget tests executed in Chromium through `@vitest/browser-playwright` after the lockfile and runtime migration.

---

## Gate 4: Final Verification (Clean Build)
**Executed:** 2026-03-09 16:16 local
**Clean command:** `rm -rf dist node_modules && npm install && npm run build && npm test`
**Exit code:** 0
**Status:** PASS
**Tests passed:** 5/5

<details>
<summary>Output</summary>

```text
added 158 packages, and audited 159 packages in 1s

44 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities

> mcp-task-widget@1.0.0 build
> tsc --noEmit && vite build && tsc -p tsconfig.server.json

vite v7.3.1 building client environment for production...
✓ 243 modules transformed.
dist/widget/index.html          0.45 kB │ gzip:   0.27 kB
dist/widget/assets/index.css    0.21 kB │ gzip:   0.16 kB
dist/widget/assets/index.js   546.29 kB │ gzip: 131.15 kB

(!) Some chunks are larger than 500 kB after minification. Consider:
- Using dynamic import() to code-split the application
- Use build.rollupOptions.output.manualChunks to improve chunking: https://rollupjs.org/configuration-options/#output-manualchunks
- Adjust chunk size limit for this warning via build.chunkSizeWarningLimit.
✓ built in 602ms

> mcp-task-widget@1.0.0 test
> vitest run


 RUN  v4.0.18 /Users/oscarmarina/Projects/AGENTS/openai-apps-sdk

 ✓  server  src/server/task-service.test.ts (1 test) 25ms
 ✓  server  src/server/widget-resource.test.ts (2 tests) 5ms
stderr | unknown test
Lit is in dev mode. Not recommended for production! See https://lit.dev/msg/dev-mode for more information.
 ✓  widget (chromium)  src/widget/task-widget-app.test.ts (2 tests) 118ms

 Test Files  3 passed (3)
		Tests  5 passed (5)
	Start at  16:16:31
	Duration  1.58s (transform 81ms, setup 0ms, import 1.05s, tests 148ms, environment 0ms)
```

</details>

**Notes:** Clean verification confirms the `node:http` server bootstrap, start script target, and full reinstall path all work from scratch.

---

## Self-Review (Full projects only)

### Domain Checklist Results

| Check | Command | Result | Pass? |
|-------|---------|--------|-------|
| Uses official SDK bridge | `grep -R "@modelcontextprotocol/ext-apps" -n src/widget ; grep -R "document\.referrer" -n src/widget ; true` | `src/widget/mcp-task-bridge.ts:7` imports the SDK; `document.referrer` had no matches | YES |
| No UI-only fields in structured content | `grep -R "selected\|expanded\|draft" -n src/server ; true` | No matches in `src/server` | YES |
| Relative Vite assets configured | `grep -n "base:\|entryFileNames" vite.config.ts` | Found `base: './'` and `entryFileNames: 'assets/index.js'` | YES |
| CSP includes server domain | `grep -R "connectDomains\|resourceDomains\|sessionIdGenerator\|access-control-allow-origin" -n src/server` | Found CSP domains in `src/server/widget-resource.ts`, CORS headers in `src/server/main.ts`, and stateless transport config in `src/server/main.ts` | YES |
| Widget tests use browser env | `grep -n "browser:" vitest.config.ts && grep -n "jsdom" vitest.config.ts` | Browser config present at `vitest.config.ts`; `jsdom` had no matches | YES |
| External script in resource HTML | `grep -R "widget/assets/index.js\|widget/assets/index.css" -n src/server` | Absolute asset references present in `src/server/widget-resource.ts` | YES |
| CSS asset exists when linked | `test -f dist/widget/assets/index.css && test -f dist/widget/assets/index.js` | Both files exist after build | YES |
| Startup path matches build output | `test -f dist/server/server/main.js && node -e "console.log(require('./package.json').scripts.start)"` | Built entry exists and script is `node dist/server/server/main.js` | YES |

### Devil's Advocate
1. **What happens when:** `PUBLIC_ORIGIN` is not set in a remote deployment, ChatGPT loads the widget over a slower network with the current 546 kB JS bundle, or multiple users share the same server-side JSON task file.
2. **The weakest link is:** runtime deployment configuration. The local fallback origin is correct for development, but remote ChatGPT usage still depends on the operator setting a public HTTPS origin correctly.
3. **If I had to break this, I would:** deploy the server behind a tunnel without setting `PUBLIC_ORIGIN`, then let the resource HTML point back to `http://localhost:3001`, which would make widget asset loading fail in the host iframe.

### Findings

| # | Severity | Finding | Impact |
|---|----------|---------|--------|
| 1 | P2 | The built widget bundle is `546.29 kB` minified, which triggers Vite's chunk-size warning. | Embedded widget startup may be slower than necessary in ChatGPT and other MCP hosts. |
| 2 | P2 | Remote deployments still require `PUBLIC_ORIGIN` to be set correctly; the default fallback is development-only. | A misconfigured deployment can return valid MCP responses while the iframe fails to load widget assets. |
| 3 | P3 | Task persistence is global and file-backed rather than user-scoped. | A multi-user deployment would share one task list across all users unless a per-user storage layer is added. |

---

## Failure History

### 2026-03-09 Gate 0 FAILED
**Error:** `npm install` failed with `ETARGET` for `@modelcontextprotocol/ext-apps@1.27.1`.
**Root Cause:** Initial version verification was contaminated by a shared terminal that had drifted into a cloned `/tmp` repo, so the package versions recorded in `package.json` were not the actual npm registry versions.
**Fix:** Re-verified package versions in fresh shells against `https://registry.npmjs.org`, updated the manifest to `@modelcontextprotocol/ext-apps@1.2.0` and `@modelcontextprotocol/sdk@1.27.1`, and re-ran `npm install` successfully.
**Re-run result:** PASS — see Gate 0 above for passing output.

---

## Domain Profile Updates

| What Changed | Section Updated | Trigger |
|---|---|---|
| Added Pitfall 11 for fixed stylesheet links without an emitted CSS asset | Common Pitfalls | Resource HTML referenced `assets/index.css` before the build emitted that file |
| Added a CSS asset existence automated check | Automated Checks | Needed a repeatable way to verify that a fixed stylesheet link is valid after build |
| Added a decision to import a real global stylesheet when linking `/widget/assets/index.css` | Decision History / Review Checklist | Final widget resource design now depends on a stable emitted CSS asset |