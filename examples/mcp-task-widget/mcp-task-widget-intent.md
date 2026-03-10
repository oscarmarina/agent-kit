# Intent: MCP Task Widget

**Date:** 2026-03-09
**Size:** Full
**Domain Profile:** agent-kit/domains/mcp-apps-lit-vite.md
**Supersedes:** —

## Goal

Build a ChatGPT-compatible MCP App that turns the Lit todo UI into an embedded widget backed by MCP tools for task CRUD operations. The widget should use Lit 3, signals, Material Web, and the MCP Apps bridge so task management works directly inside the conversation UI.

## Behavior

- **Given** ChatGPT renders the widget resource for the task tool, **when** the widget loads, **then** it connects through the official MCP Apps bridge and renders the authoritative task list returned by the server.
- **Given** a user enters a new task in the widget, **when** they submit it, **then** the widget calls an MCP server tool and re-renders from the returned task snapshot.
- **Given** a rendered task, **when** the user edits its title or completion state, **then** the widget updates the task through an MCP tool and reflects the updated server state.
- **Given** a rendered task, **when** the user deletes it, **then** the widget calls the delete MCP tool and removes the task from the rendered list using the returned snapshot.
- **Given** the MCP server is exposed over HTTP, **when** ChatGPT fetches the widget resource and assets, **then** the resource HTML and asset routes load correctly under the sandbox CSP and cross-origin rules.

## Decisions

| Decision | Choice | Rejected | Why |
|----------|--------|----------|-----|
| MCP UI bridge | `App` from `@modelcontextprotocol/ext-apps` | Custom `postMessage` bridge | The domain profile explicitly requires the official SDK and it handles host lifecycle safely |
| Server transport | Streamable HTTP plus stdio entrypoint | HTTP-only or stdio-only | HTTP is needed for ChatGPT embedding and stdio remains useful for local MCP testing |
| HTTP runtime | `node:http` | Express | This project already chose the native HTTP server and the reference implementation style fits the required routes |
| Task storage | File-backed JSON repository with Zod validation | Pure in-memory state | File-backed storage preserves tasks across server restarts without adding external infrastructure |
| Widget state model | Signals-based controller feeding Lit components | Ad hoc element-local state only | Matches the referenced Lit repo patterns and keeps server data authoritative |
| Widget asset delivery | External Vite-built assets served from `/widget/*` | Inline bundled HTML/JS | ChatGPT iframe CSP blocks inline bundled scripts |

## Constraints

**MUST:**
- Use MCP Apps, Lit, Vite, and TypeScript.
- Use components and patterns from the `lit-signals-material` repo as the UI base.
- Expose MCP tools for task CRUD operations.
- Have the widget communicate through the MCP Apps bridge.
- Produce project artifacts in `docs/`.

**MUST NOT:**
- Implement a custom iframe bridge instead of the official MCP Apps SDK.
- Inline bundled widget JavaScript into the MCP resource HTML.
- Leak UI-only transient state into tool `structuredContent`.

**SHOULD:**
- Keep the widget visually close to the referenced Lit + signals + Material style.
- Preserve deterministic build output for widget asset URLs.
- Include automated tests for both server and widget flows.

## Scope

**IN:**
- MCP server with task list, create, update, and delete tools
- Embedded Lit widget for creating, editing, completing, filtering, and deleting tasks
- File-backed task persistence and schema validation
- Vite widget build, TypeScript server build, and browser-based widget tests
- Verification log entries for all required gates

**OUT:**
- Multi-user synchronization
- Authentication and per-user task separation
- Rich due dates, labels, reminders, or notifications
- Remote database integration

## Acceptance

- `npm install`, `npm run build`, and `npm test` pass.
- The MCP server serves `/mcp` and widget assets with a resource HTML document that ChatGPT can load.
- The widget uses MCP tool calls to create, read, update, and delete tasks.
- The widget styling and component approach reflect Lit 3 + signals + Material Web patterns from the referenced repo.