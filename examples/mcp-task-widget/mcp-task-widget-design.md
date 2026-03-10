# Design: MCP Task Widget

**Intent:** docs/mcp-task-widget-intent.md
**Domain Profile:** agent-kit/domains/mcp-apps-lit-vite.md
**Date:** 2026-03-09

## Domain Profile Selection Rationale

| Candidate Profile | Keyword Score | Excluded? | Reason |
|-------------------|---------------|-----------|--------|
| `mcp-apps-lit-vite` | 6 | No | Matches `mcp apps`, `lit`, `vite`, `widget`, `iframe`, and the ChatGPT embedded UI requirement |

**Selected Profile:** `mcp-apps-lit-vite`
**Selection Basis:** Unique highest score `>= 2`

## Architecture

### Stack

| Technology | Version | Verified Via | Purpose |
|-----------|---------|-------------|---------|
| `lit` | `3.3.2` | `npm view lit version` | Widget component model |
| `vite` | `7.3.1` | `npm view vite version --json` | Widget bundling |
| `typescript` | `5.9.3` | `npm view typescript version` | Shared typing and compilation |
| `@modelcontextprotocol/sdk` | `1.27.1` | `npm view @modelcontextprotocol/sdk version --registry=https://registry.npmjs.org` | MCP server transports and core server APIs |
| `@modelcontextprotocol/ext-apps` | `1.2.0` | `npm view @modelcontextprotocol/ext-apps version --registry=https://registry.npmjs.org` and the published package source | Widget bridge and app resource registration |
| `@material/web` | `2.4.1` | Referenced UI base package manifest | Material Web UI components |
| `@lit-labs/signals` | `0.2.0` | `npm view @lit-labs/signals version` | Signals-based widget state |
| `zod` | `4.x` | Package install lock verification during Gate 0 | Tool and persistence validation |

### Structure

- `src/server/` contains the MCP server entrypoint, HTTP bootstrap, app resource registration, task repository, and Zod schemas.
- `src/widget/` contains the Lit widget entrypoint, signal-based controller, Material Web custom elements, and CSS.
- `src/shared/` contains task schemas and types shared across server and widget boundaries.
- `docs/` contains intent, design, and verification artifacts.
- `data/tasks.json` stores persisted task state for local development and runtime.

### Data Flow

1. The HTTP server receives a request on `/mcp` and creates a stateless `StreamableHTTPServerTransport` plus a fresh `McpServer` instance.
2. MCP tools validate input with Zod, mutate the file-backed task repository, and return an authoritative `{ tasks, summary }` snapshot in `structuredContent`.
3. The widget initializes `App` from `@modelcontextprotocol/ext-apps`, receives tool results, and calls additional server tools with `app.callServerTool(...)`.
4. A widget controller stores authoritative task data in signals and derives counts, progress, and filtered lists via computed signals.
5. UI-only state such as draft text, edit mode, and active filter remains in widget-local signals and never crosses into server payloads.

### Initialization Chain

1. `src/server/main.ts` starts either stdio or the HTTP server.
2. The HTTP server mounts `/mcp` and `/widget/*`, enabling CORS on both routes.
3. The MCP tool registration links task tools to a `ui://tasks/task-board.html` resource.
4. ChatGPT fetches the resource HTML, which references externally served widget assets under `/widget/assets/*`.
5. `src/widget/main.ts` creates an `App`, registers lifecycle callbacks, applies host theme variables, connects to the host, and triggers the initial `list_tasks` call.
6. The Lit root component renders from signals and dispatches CRUD actions back through the bridge client.

### Dependencies

**Production:**
- `@lit-labs/signals` — signal and computed state for the widget controller
- `@material/web` — Material Web inputs, buttons, list, checkbox, divider, and progress components
- `@modelcontextprotocol/ext-apps` — widget `App`, host theming helpers, and server-side app registration helpers
- `@modelcontextprotocol/sdk` — MCP server implementation and transports
- `node:http` — HTTP server for `/mcp` and `/widget/*`
- `zod` — schema validation at MCP and persistence boundaries

**Development:**
- `@types/node` — TypeScript types for runtime APIs
- `@vitest/browser-playwright` — browser-mode widget tests
- `playwright` — real browser runtime for widget verification
- `vitest` — server and widget tests
- `typescript` — compilation
- `vite` — widget build

## Decisions

| # | Decision | Choice | Alternatives Considered | Rationale |
|---|----------|--------|------------------------|-----------|
| 1 | Server framework | `node:http` with manual route handling | Express, Fastify | Matches the project decision from the previous session and keeps the HTTP surface minimal for this embedded MCP server |
| 2 | Tool surface | `list_tasks`, `create_task`, `update_task`, `delete_task` | Single monolithic task mutation tool | Explicit CRUD tools are easier for both models and widget code to reason about |
| 3 | Persistence | JSON file repository | In-memory only, SQLite | JSON is sufficient for a local MCP sample and avoids infrastructure overhead |
| 4 | Widget composition | Lit custom element plus signals controller | Monolithic imperative DOM app | Keeps the architecture close to the referenced `lit-signals-material` project |
| 5 | Styling | Material Web components plus custom CSS variables and gradient surfaces | Plain HTML controls | Preserves the requested UI base and provides a stronger embedded-widget presentation |
| 6 | Asset resolution | Stable Vite output names and server-generated resource HTML | Parsing Vite manifest at runtime | Stable names reduce moving parts and align with the selected domain profile |

## Risks (Adversary Lens)

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| Public origin is wrong behind a tunnel or proxy | Widget assets fail to load in ChatGPT | Medium | Resolve origin from env first, then forwarded headers, and include it in CSP and asset URLs |
| Persisted JSON is malformed or manually edited | Tool calls can crash or return invalid data | Medium | Parse storage through Zod and fall back to an empty list on invalid content |
| Widget attempts calls before bridge connection finishes | Initial render stalls or errors | Medium | Controller tracks connection state and defers actions until `app.connect()` resolves |
| Cross-origin module asset requests are blocked | Widget stays blank | Medium | Add `Access-Control-Allow-Origin: *` to `/widget/*` responses |
| Future changes leak UI-only fields into tool payloads | Model-visible data becomes polluted | Low | Centralize snapshot construction on the server and keep UI state in the controller only |

## Domain Pitfalls Applied

| Pitfall | Applies? | How Addressed |
|---------|----------|---------------|
| Official SDK bridge only | Yes | Widget uses `App` from `@modelcontextprotocol/ext-apps`; no custom bridge code is written |
| UI state must not enter `structuredContent` | Yes | Server snapshots contain only task entities and summary counts |
| Relative/portable widget assets | Yes | Vite uses `base: './'` and stable output names |
| No inline bundled scripts | Yes | Resource HTML points to `/widget/assets/index.js` and `/widget/assets/index.css` |
| CSP must include server domain | Yes | Resource metadata includes the resolved public origin in `connectDomains` and `resourceDomains` |
| Widget fallback background | Yes | Resource HTML includes an explicit body background style |
| Stateless transport | Yes | `/mcp` creates `StreamableHTTPServerTransport({ sessionIdGenerator: undefined })` |
| Widget tests must use a real browser | Yes | Vitest uses a browser project backed by Playwright |
| Widget assets need CORS headers | Yes | `/widget/*` responses include `Access-Control-Allow-Origin: *` |

## Verification

| Gate | Command | Pass Criteria |
|------|---------|---------------|
| 0 | `npm install` | Exit 0 and no unresolved dependency errors |
| 1 | `npm run build` | Server build succeeds and widget assets are emitted under `dist/widget/` |
| 2 | `npm run build && npm test` | Build succeeds and all tests pass |
| 3 | `npm test` | All server and widget tests pass |
| 4 | `rm -rf dist node_modules && npm install && npm run build && npm test` | Clean build and tests pass from scratch |

## Test Strategy

- **What to test:** task repository behavior, CRUD tool behavior, resource HTML generation, widget initial load, and widget CRUD interactions against a mocked bridge client
- **How:** Vitest workspace with a Node project for server tests and a Playwright browser project for widget tests
- **Coverage target:** At least core CRUD paths and the widget’s create/update/delete flow
- **What NOT to test:** Material Web internals, MCP SDK internals, or browser rendering details already owned by dependencies