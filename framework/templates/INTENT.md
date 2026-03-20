# Intent: [Short Name]

**Date:** [Date]
**Size:** [Quick / Standard / Full]
**Domain Profile:** [Reference to framework/domains/ profile, or "New — will be created"]
**Supersedes:** [Reference to previous Intent if this reworks an existing feature, or "—"]

## Goal

[1-2 sentences: What we're building/changing and why it matters]

## Behavior

Observable behaviors that define this change. Use given/when/then format for clarity.

- **Given** [precondition], **when** [action], **then** [observable outcome]
- **Given** [precondition], **when** [action], **then** [observable outcome]

<!-- Examples:
- Given a user is on the habit list, when they tap "Mark Done" on a habit card, then the card disables its controls and shows a "done" visual state
- Given a habit has been completed 3+ consecutive days, when the habit list renders, then a streak badge appears on the card
- Given the app loads for the first time, when no habits exist in localStorage, then an empty state with a "Create your first habit" prompt appears
-->

<!-- Only include behaviors that DEFINE the change — not exhaustive specs.
     If a behavior is obvious from the Goal, skip it. -->

## Decisions

Key choices made by the human or agreed during design discussion.

| Decision | Choice | Rejected | Why |
|----------|--------|----------|-----|
| [e.g., Test environment] | [Vitest browser mode] | [jsdom, happy-dom] | [Web Components need real browser APIs] |
| [e.g., CSS approach] | [CSS custom properties] | [Tailwind, CSS modules] | [User constraint: no external CSS frameworks] |

## Constraints

**MUST:**
- [Hard requirements — e.g., "Standalone Angular components only, no NgModules"]
- [Technical constraints — e.g., "localStorage only, no backend"]

**MUST NOT:**
- [Explicit rejections — e.g., "No Tailwind CSS"]
- [Anti-patterns — e.g., "Never use jsdom for Web Component testing"]

**SHOULD:**
- [Preferences — e.g., "Signals over RxJS where possible"]
- [Quality targets — e.g., "≥70% test coverage on core service"]

## Scope

**IN:**
- [Feature/capability 1]
- [Feature/capability 2]

**OUT:**
- [Explicitly excluded — e.g., "No multi-user sync"]
- [Deferred — e.g., "No PWA offline support in v1"]

## Acceptance

[Specific, verifiable conditions that define "done"]

- [e.g., `npm install && npm run build && npm test` passes from clean state]
- [e.g., User can create, complete, edit, and delete habits]
- [e.g., Stats page shows streak, completion rate, and most consistent habit]
