# Design: [Project Name]

**Artifact Schema Version:** 1.1.0
**Intent:** docs/[project]-intent.md
**Domain Profile:** framework/domains/[profile].md
**Date:** [Date]

## Domain Profile Selection Rationale

Document why this profile was selected so routing is auditable and repeatable.

| Candidate Profile | Keyword Score | Excluded? | Reason |
|-------------------|---------------|-----------|--------|
| [e.g., web-angular-lit] | [e.g., 4] | [No] | [Matched angular, lit, vite, vitest] |
| [e.g., web-lit-from-design] | [e.g., 1] | [Yes/No] | [Excluded by Do Not Use When, or low match] |

**Selected Profile:** [profile id]
**Selection Basis:** [Unique highest score >= 2, or human-selected after tie]

## Skills Loaded

| Skill | Location | Why loaded |
|-------|----------|------------|
| [e.g., frontend-design] | [.agents/skills/frontend-design/] | [Task involves UI — skill description matches] |

[If no skills exist or none match, write "None — no matching skills found."]

## Design Status

[Use this section when runtime behavior may invalidate early assumptions. Mark sections, decisions, or assumptions as Provisional, Verified, or Revised After Runtime Validation. If the design is fully stable before implementation, write "All sections currently provisional until runtime verification" or explain why explicit markers are unnecessary.]

| Item | Status | Evidence / Trigger |
|------|--------|--------------------|
| [e.g., worker loading strategy] | [Provisional / Verified / Revised After Runtime Validation] | [e.g., pending runtime validation, verified by Gate 3, revised after browser test failure] |
| [e.g., thumbnail generation strategy] | [Provisional / Verified / Revised After Runtime Validation] | [e.g., design decision only, verified by stress test, changed after memory issue] |

## Research Summary

*(Full projects only — required before writing any other section. For Standard projects, omit this section entirely.)*

What the existing codebase contains that is relevant to this project. This is an investigation result, not a plan.

**Existing modules surveyed:**
- [Module/directory] — [what it does, what patterns it uses]
- [Module/directory] — [relevant constraints or dependencies]

**Patterns already in use:**
- [Pattern name] — [where it appears, what it implies for this project]

**Constraints imposed by existing code:**
- [What cannot change without breaking downstream consumers]
- [APIs, events, or data shapes that must remain compatible]

**Greenfield note:** If no prior code exists, write "Greenfield — no prior architecture to survey."

---

## Architecture

### Stack

| Technology | Version | Verified Via | Purpose |
|-----------|---------|-------------|---------|
| [e.g., Angular] | [21.1.4] | [`npm view @angular/core version`] | [App framework] |
| [e.g., Lit] | [3.3.2] | [`npm view lit version`] | [Web Components] |

*"Verified Via" accepts any auditable source: package manager query (`npm view`, `pip show`), file inspection ("Copied from [path]", "Header in [file]"), vendor documentation ("Datasheet rev. X"), or runtime query ("PLC firmware version read via diagnostic tool"). The point is traceability, not a specific command.*

### Structure

[Component/module organization — how the code is laid out. List key directories and what each contains.]

### Data Flow

[How data moves through the system. For multi-technology projects: how data crosses boundaries, what type conversions happen, and which direction events flow.]

### Initialization Chain

[The exact startup sequence. Every link must be explicit — missing links cause runtime errors.]

1. [Entry point] → loads [prerequisites]
2. [Prerequisites] → registered before [framework bootstrap]
3. [Framework bootstrap] → renders [application]

### Dependencies

**Production:**
- [package@version] — [purpose]

**Development:**
- [package@version] — [purpose]
- [package@version] — peer dep of [parent package]

## Decisions

Every significant architectural choice, with rationale and rejected alternatives.

| # | Decision | Choice | Alternatives Considered | Rationale |
|---|----------|--------|------------------------|-----------|
| 1 | [e.g., State management] | [Signals] | [RxJS BehaviorSubjects, NgRx] | [Simpler for single-component state at this scale] |
| 2 | [e.g., DOM rendering] | [Light DOM] | [Shadow DOM] | [CSS custom properties need to cascade to child elements] |

## ADR Summary

*(Required for Full projects. Use one ADR row per major architectural decision. Standard projects may omit this section if no Design document is produced.)*

| ADR | Status | Decision | Why it exists |
|-----|--------|----------|---------------|
| ADR-001 | [Accepted / Superseded / Revised] | [Short decision title] | [Why this needed a durable record beyond the generic decisions table] |
| ADR-002 | [Accepted / Superseded / Revised] | [Short decision title] | [What tradeoff or constraint it preserves for future work] |

## Risks (Adversary Lens)

Identified BEFORE implementation. Each risk has impact assessment and mitigation strategy.

| Risk | Impact | Likelihood | Mitigation |
|------|--------|------------|------------|
| [e.g., localStorage quota exceeded] | [Silent data loss] | [Low] | [Try-catch on write, warn user on failure] |
| [e.g., Multiple tabs open] | [State desync] | [Medium] | [Accept for v1, document as known limitation] |
| [e.g., Malformed data in storage] | [Runtime crash] | [Low] | [Validate on load, fallback to empty state] |

## Adversary Questions Applied

[If the domain profile has an Adversary Questions section, answer EVERY question against this specific design. These are domain-specific traps born from real bugs — checking pitfalls is not enough.]

| Question | Answer for This Design |
|----------|----------------------|
| [e.g., What happens if `document.referrer` is empty?] | [e.g., Not applicable — using SDK `App` class which uses `postMessage("*")` and validates `event.source` only] |
| [e.g., Does the SDK already provide this feature?] | [e.g., Yes — `App.callServerTool()` handles tool calls; no custom bridge needed] |

**If the profile has no Adversary Questions, write "No adversary questions in profile" and move on.**

## Domain Pitfalls Applied

> **CRITICAL:** Do not omit this section. Every pitfall from the domain profile must be evaluated against this design — even if the answer is "does not apply." Silent omission is the most common audit failure.

[For each relevant pitfall from the domain profile: which ones apply and how this design addresses them.]

| Pitfall | Applies? | How Addressed |
|---------|----------|---------------|
| [Attribute binding for non-strings] | Yes | [All Lit bindings use property binding `[prop]="expr"`] |
| [zone.js import order] | Yes | [First import in main.ts, before any Angular code] |
| [Angular compiler scope] | Yes | [`include: ['src/app/**/*.ts']` excludes Lit files] |
| [jsdom for Web Components] | Yes | [Vitest browser mode with playwright, no jsdom] |

## Verification

[Reference domain profile gate commands, or define custom ones.]

| Gate | Command | Pass Criteria |
|------|---------|---------------|
| 0 | `npm install` | Exit 0, no unresolved deps |
| 1 | `npm run build` | Exit 0, output in dist/ |
| 3 | `npm test` | All tests pass in real browser |
| 4 | `[Use domain profile clean command: POSIX or PowerShell variant]` | Everything passes clean |

## Test Strategy

- **What to test:** [Core logic, component rendering, integration points]
- **How:** [Test framework, browser mode, coverage tool]
- **Coverage target:** [e.g., ≥70% on core service, ≥80% on critical paths]
- **What NOT to test:** [e.g., Third-party library internals, simple pass-through methods]
