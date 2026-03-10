# Example: Global Logistics Control Tower

## Context

This example reverse-engineers an existing project built with an earlier version of Agent Kit. The original was a multi-tenant cold-chain monitoring system with a Lit dashboard, risk engine, incident management, and audit trail.

The prompt below is designed for the current version of the framework. It describes the same functional requirements but pushes for a bolder visual direction — moving from the original's corporate blue/teal palette to something more distinctive.

No domain profile exists for this stack yet. The LLM will create one from `_template.md` during the Design phase.

## The prompt

```
Read AGENTS.md. Build a Global Logistics Control Tower — a multi-tenant,
event-driven backend for real-time monitoring of temperature-sensitive
shipments (cold-chain logistics).

Stack: Node.js (>=22), TypeScript, Lit 3 (CDN via esm.sh), node:http,
node:test. No external frameworks (no Express, no Vitest, no React).

Core domain:
- Shipments with lifecycle: planned → in_transit → at_risk → incident → delivered
- IoT telemetry ingest (temperature, humidity, door open, battery, location)
- Risk engine: score 0-100 based on sensor thresholds + delivery delay
- Automatic incident opening when risk score crosses threshold (default: 70)
- Incident resolution with notes and audit trail
- Multi-tenant isolation on every read/write path
- Idempotent ingest (dedup by event ID and idempotency key)
- Out-of-order safe (timeline sorted by sensor timestamp, not arrival)
- Immutable audit trail for every lifecycle action
- Structured error envelope on all API responses (code, message, traceId)

REST API:
- POST /api/shipments (create, idempotent)
- POST /api/telemetry (ingest, triggers risk recalc + status transition)
- POST /api/incidents/resolve
- GET /api/dashboard/{tenantId} (shipments, open incidents, KPIs)
- GET /api/audit/{tenantId}
- GET /api/alerts/{tenantId} (in-app + email outbox)
- GET /api/metrics (ingestCount, duplicateCount, p95 latency)

Dashboard KPIs: on-time rate %, cold-chain breaches count, MTTR minutes.

Frontend: Lit custom element <control-tower-app> served as static HTML.
Responsive 12-column CSS grid. Forms for creating shipments and ingesting
telemetry. Live lists for shipments, incidents, audit trail, and alerts.
Auto-refresh after every action.

Visual direction: I want something visually bold and modern — not the
typical corporate dashboard with safe blues and grays. Think dark mode
with high-contrast accent colors, glassmorphism cards, gradient mesh
backgrounds, and strong typographic hierarchy. The UI should feel like
a mission control center, not a spreadsheet. Surprise me with the
palette but keep it readable and professional.

Storage: in-memory maps (tenant-scoped). Include a PostgreSQL schema
artifact in db/schema.sql for future persistence. Include a
docker-compose.yml with PostgreSQL 16 and RabbitMQ 3.13 for
future event streaming.

Testing: node:test runner. Cover risk scoring, status transitions,
full ingest flow with incident opening, error envelope contracts,
duplicate/out-of-order resilience, and a frontend smoke test.
Target: minimum 9 tests, all passing.

Constraints:
- MUST: tenant isolation on every path
- MUST: idempotent ingest (no state mutation on duplicates)
- MUST: immutable audit trail
- MUST NOT: use Express, Fastify, or any HTTP framework
- MUST NOT: use jsdom or external test runners
- SHOULD: keep ingest p95 < 500ms
- SHOULD: keep architecture ready for RabbitMQ/PostgreSQL adapters
```

## What the framework should produce

- **Size:** Full (new project, major architecture)
- **Domain profile:** New — no existing profile matches this backend-only Node.js stack. The LLM creates one from `_template.md`
- **Artifacts:** Intent, Design, Verification Log (with all gates), new domain profile

## What to look for when reviewing the output

1. **Domain profile creation** — The LLM should create a new profile (e.g., `backend-node-ts-event-driven.md`) with pitfalls specific to this stack: in-memory map tenant isolation, node:test quirks, ESM/CJS boundaries, idempotency edge cases.

2. **Risk engine design** — The Design document should show scoring rules, threshold logic, and status state machine before any code is written.

3. **Adversary Questions** — Even without an existing profile, the LLM should generate adversary questions in the Design: "What happens if two telemetry events arrive with the same timestamp?", "What happens if a tenant ID is missing from a request?", etc.

4. **Gate failures becoming pitfalls** — If any gate fails (it likely will on first pass), watch for the LLM adding the root cause to the new domain profile. This is the flywheel starting.

5. **Visual execution** — The prompt asks for a bold departure from typical dashboards. The Design should document the visual direction as an architectural decision with rationale.

## Original project reference

The original project (built with an earlier framework version) used:
- 438-line ControlTowerService with in-memory tenant-scoped maps
- 68-line risk engine with threshold-based scoring
- 206-line HTTP server with structured error handling
- 364-line Lit component with 12-column grid layout
- 9 tests covering unit, integration, contract, resilience, and smoke
- Corporate blue/teal palette (professional but conventional)
- All 4 verification gates passing

The prompt above preserves all functional requirements while pushing for a more distinctive visual identity and letting the current framework version guide the process.
