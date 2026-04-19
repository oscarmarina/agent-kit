# Intent: [Short Name]

**Artifact Schema Version:** 1.1.0
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

## Adversary Questions Applied

*Include this section when no separate Design document is produced (Standard projects with straightforward architecture). If a Design document exists, these tables belong there instead. See BUILDER.md Standard step 5.*

| Question | Answer for This Design |
|----------|----------------------|
| [Profile adversary question] | [How this design addresses it] |

**If the profile has no Adversary Questions, write "No adversary questions in profile."**

## Domain Pitfalls Applied

[Each row needs `Status` + `Evidence` per `BUILDER.md → Evidence States`. At intent time most rows are `Provisional` — they become `Verified` when the verification log cites a gate output or file path as Source.]

| Pitfall | Applies? | How Addressed | Status | Evidence |
|---------|----------|---------------|--------|----------|
| [Profile pitfall name] | [Yes/No] | [How this design handles it, or why it doesn't apply] | [Provisional / Verified / Blocked] | [Source pointer or "manual review — Verified after Gate N"] |

**If no domain profile is loaded yet, write "No profile loaded — will be created during this project."**

## Acceptance

[Specific, verifiable conditions that define "done"]

- [e.g., `npm install && npm run build && npm test` passes from clean state]
- [e.g., User can create, complete, edit, and delete habits]
- [e.g., Stats page shows streak, completion rate, and most consistent habit]
