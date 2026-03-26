# Agent Kit — LLM Context

This repository **is** the Agent Kit framework. It is not a project built with the framework — it is the framework itself.

## What this repository contains

Agent Kit is a process framework for LLM-assisted software development. It provides a structured process (lenses, gates, artifacts) and a learning mechanism (domain profiles) that accumulates knowledge across projects.

When someone copies `framework/` into their own repository and points `AGENTS.md` at `BUILDER.md`, any LLM can follow the process to build software with verification and domain-specific knowledge.

## Repository structure

```
.
├── AGENTS.md                    # Entry point — reading order: domain profile → BUILDER.md → GATEKEEPER.md
├── README.md                    # Explanation: what the framework is and why it exists
├── GUIDE.md                     # Tutorial: step-by-step first project walkthrough
├── LICENSE                      # MIT
│
├── framework/                   # THE FRAMEWORK (this is what gets copied to target repos)
│   ├── BUILDER.md               # Process contract — the LLM reads and follows this
│   ├── GATEKEEPER.md            # Verification agent — executes commands, enforces gates
│   ├── README.md                # Technical reference — gates, artifacts, contracts
│   ├── VERSION                  # Framework version for tracking updates
│   ├── domains/
│   │   ├── _template.md         # Template for creating profile links that extend catalog profiles
│   │   └── README.md            # How domain profiles work
│   └── templates/
│       ├── INTENT.md            # Template: what and why
│       ├── DESIGN.md            # Template: how (architecture, decisions, risks)
│       ├── VERIFICATION_LOG-template.md  # Template: proof (gate output, progress)
│       └── DOMAIN_PROFILE-template.md    # Template: standalone full domain profile
│
├── catalog/                     # Community-contributed base profiles
│   ├── README.md                # How to use and contribute profiles
│   ├── apps-sdk-mcp-lit-vite.md # Real profile (11 pitfalls, 7 adversary Qs)
│
└── docs/                        # Would hold generated artifacts in a real project
```

## Framework concepts

- **Domain profiles** accumulate stack knowledge (pitfalls, adversary questions, checks, decisions) across projects
- **Gates** (0-4) are verification checkpoints with real command output — "assumed to pass" is never valid
- **Four lenses**: User, Architecture, Adversary, Domain — thinking modes, not sequential phases
- **Anti-Loop Rule**: produce the Intent document before continuing to investigate
- **Pre-code checkpoint** (step 4): 4 questions inlined into the process before any code
- **Debug Sprint**: bounded mode for dense debugging loops — verification log stays current, design doc reconciled when sprint ends. **Integration Discovery variant** for undocumented dependencies.
- **Context Pressure Protocol**: artifact degradation order when context is scarce (Progress > Profile > Intent > Design)
- **Resume**: verification log Progress section enables continuing interrupted work

### The process (BUILDER.md)

The LLM determines project size (Quick / Standard / Full), then follows a structured process:

1. **Intent** — Capture what and why before doing anything (`docs/[project]-intent.md`)
2. **Domain profile** — Load accumulated stack knowledge, read every pitfall and adversary question
3. **Skills** — Load relevant skills from `.github/skills/` and `.agents/skills/` as design guidance
4. **Pre-code checkpoint** — 4 questions: deps solve this? wrong assumption? pitfalls checked? right size?
5. **Design** — Architecture, decisions, risks, pitfalls applied, adversary questions answered. **Optional for Standard; mandatory for Full** with ADRs.
6. **Gated build** — Gates 0→1→2 per feature phase
7. **Tests + verification** — Gates 3→4 with real command output recorded
8. **Self-review** — Adversary lens + domain checklist
9. **Domain learning** — Update the profile with new discoveries

In single-agent environments (most current tooling), the LLM fulfills both Builder and GateKeeper roles with the same mechanical discipline.

### Domain profiles (the differentiator)

Domain profiles use an **inheritance model**. Base profiles in `catalog/` contain stack-wide knowledge (pitfalls, adversary questions, automated checks, decision history). Profile links in `framework/domains/` extend a base profile with project-specific additions. Every gate failure becomes a new pitfall. Every project makes the next one better.

Community-contributed base profiles live in `catalog/`. Create a profile link in `framework/domains/` that extends the relevant catalog profile — see `framework/domains/_template.md` for the format.

A base profile contains: Selection Metadata, Terminology Mapping, Verification Commands, Common Pitfalls, Adversary Questions, Integration Rules, Automated Checks, Decision History, Review Checklist. A profile link contains: `extends` reference, Local Pitfalls, Local Overrides, Local Decision History.

### Verification gates

Gates are mandatory checkpoints with real command output. "Assumed to pass" is never valid. Gates 0 (deps) → 1 (scaffold) → 2 (features) → 3 (tests) → 4 (clean build).

### Artifacts

- **Intent** — Scope anchor. Given/when/then behaviors, MUST/MUST NOT constraints, IN/OUT scope.
- **Design** — Single document replacing PRD + tech spec + implementation plan. **Required for Full; optional for Standard.** Includes Adversary Questions Applied and Domain Pitfalls Applied as separate mandatory sections (in Design or Intent, depending on size).
- **Verification Log** — Gate evidence + Progress section for resuming interrupted work.

### Anti-Loop Rule

The LLM produces the Intent before continuing to investigate. Unclear decisions become open questions asked to the human — not reasons to keep researching.

### Resume mechanism

Each verification log has a Progress table at the top. When a session is interrupted, the next session reads Progress and continues from the last completed step.

## Documentation follows Diátaxis

| Document | Type | Audience | Serves |
|----------|------|----------|--------|
| `README.md` | Explanation | Human | Understanding — what and why |
| `GUIDE.md` | Tutorial | Human | Learning — step-by-step first project |
| `framework/README.md` | Reference | Human | Information — specs, contracts, definitions |
| `catalog/README.md` | How-to guide | Human | How to use and contribute domain profiles |
| `BUILDER.md` | Agent contract | LLM | Process instructions the LLM follows |
| `GATEKEEPER.md` | Agent contract | LLM | Verification instructions the LLM follows |

## When modifying the framework

- `BUILDER.md` is the source of truth for the process. Changes here affect how every LLM behaves.
- Profile link template (`framework/domains/_template.md`) defines what new profile links look like. Full profile template (`framework/templates/DOMAIN_PROFILE-template.md`) defines standalone profiles and catalog contributions. Changes propagate to all future profiles.
- Template changes (`templates/*.md`) affect artifact structure for all future projects.
- `README.md`, `GUIDE.md`, and `framework/README.md` must stay aligned with `BUILDER.md`. If the process changes, the docs must reflect it.
- Examples in `examples/` are historical artifacts — do not modify them to match framework changes.
- Catalog profiles in `catalog/` are contributed by the community — review for quality but preserve the contributor's learnings.

## Conventions

- All framework documentation is in English.
- Standalone/base profiles use the structure in `framework/templates/DOMAIN_PROFILE-template.md`. Profile links use `framework/domains/_template.md`. Do not deviate.
- Verification logs are per-project: `docs/[project]-verification.md`, not a shared file.
- Project code goes in its own directory, never at the repo root.
- The AGENTS.md entry point defines a 3-step reading order (domain profile → BUILDER.md → GATEKEEPER.md). Process logic lives in BUILDER.md.
- Skills are guidance, not process. They live in `.github/skills/` (repo-level) or `.agents/skills/` (agent-level/external). Skills cannot override gates, skip artifacts, or replace domain profile correctness. Technical learnings go in domain profiles; process learnings go in skills; project-specific learnings go in `docs/`.
