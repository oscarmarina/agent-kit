# Agent Kit — Reference

> **Non-normative view.** Reference index for humans. Operational truth for agents lives in `BUILDER.md` and `GATEKEEPER.md`. If this page disagrees with either, those files win.

## Contents

| File | Purpose |
|------|---------|
| `BUILDER.md` | Process contract — lenses, gates, pre-implementation checkpoint, single/dual-agent verification, rules |
| `GATEKEEPER.md` | Verification agent contract — executes commands, enforces gates, updates pitfalls |
| `ARTIFACT_SCHEMA_VERSION` | Artifact schema version for Intent/Design/Verification/Profile templates |
| `domains/_template.md` | Template for creating profile links that extend catalog profiles |
| `domains/README.md` | How domain profiles work — types, matching, merge rules |
| `domains/*.md` | Domain profiles — standalone or links extending catalog base profiles |
| `templates/INTENT.md` | Template for Change Intent Records |
| `templates/DESIGN.md` | Template for Design documents |
| `templates/VERIFICATION_LOG-template.md` | Template for gate execution logs with progress tracking |
| `templates/DOMAIN_PROFILE-template.md` | Template for creating standalone full domain profiles |
| `templates/SUBAGENT_PROMPT_TEMPLATE.md` | Prompt template for repeat sub-agent delegations (ephemeral per-call; not a runtime role registry) |

## Setup

1. Copy `framework/` into the target repo.
2. Copy `AGENTS.md` from the repo root into the target repo (or create it — see `AGENTS.md` for the current format). It defines the reading order: `BUILDER.md`, then `GATEKEEPER.md`, then `framework/domains/` only when BUILDER step 2 routes there.
3. Create `docs/` for project-specific artifacts.
4. Create a profile link in `framework/domains/` that extends a relevant profile from the [catalog](../catalog/), or let the Builder create one during the first project. See `domains/_template.md` for the link format.

Resulting structure:

```
repo/
├── AGENTS.md               ← entry point (reading order)
├── framework/              ← framework (reusable)
│   ├── BUILDER.md
│   ├── GATEKEEPER.md
│   ├── ARTIFACT_SCHEMA_VERSION
│   ├── VERSION
│   ├── domains/
│   └── templates/
├── docs/                   ← project artifacts (intent, design, verification)
└── my-project-name/        ← generated project code (named by the Builder)
    ├── package.json
    ├── src/
    └── ...
```

Project code goes in its own directory — never at the repo root.

## Versioning

- `framework/VERSION` tracks the framework release.
- `framework/ARTIFACT_SCHEMA_VERSION` tracks the artifact contract used by the templates (`INTENT`, `DESIGN`, `VERIFICATION_LOG`, standalone profiles, and profile links).
- The two values may move together, but they are not the same thing. A documentation/process release can leave the artifact schema unchanged, and a schema bump requires migration guidance.
- When the artifact schema changes, a migration guide is provided at `framework/MIGRATION-[old]-to-[new].md` (e.g., `framework/MIGRATION-1.0-to-1.1.md`). Artifacts created under the previous schema remain readable but should not be assumed to satisfy the new audit contract until migrated.

## Prompts

| Scenario | Prompt |
|----------|--------|
| New project | `Read AGENTS.md. Build a [description]. Stack: [technologies].` |
| New feature | `Read AGENTS.md. [Description of what you want].` |
| Bug fix | `Read AGENTS.md. [Description of the problem].` |
| Resume work | `Read AGENTS.md. Continue where the last session left off.` |

**What the framework needs from your prompt:**

- Always start with `Read AGENTS.md` — without it, the LLM has no framework
- Name the stack explicitly (`Lit 3, Vite, TypeScript`) — these keywords drive domain profile selection
- State constraints as MUST / MUST NOT — they get captured in the Intent and enforced throughout
- If you have reference code, say "study for patterns, do not copy" — otherwise the LLM may copy verbatim
- Everything you don't say, the LLM decides alone. Be explicit about what matters to you

## Process sizes

| Size | Criteria | Artifacts produced |
|------|----------|--------------------|
| Quick | < 3 files, clear intent | Code + Gate 2 minimum |
| Standard | Feature-sized | Intent + Code + Verification Log (Design optional — see Standard step 5) |
| Full | New project / major rearchitecture | Research Summary + Intent + Design (mandatory, with ADRs) + Code + Verification Log + Devil's Advocate self-review |

All sizes load relevant skills when they exist (see Skills section below). For Full, skill loading is mandatory — document that none were found if that's the case.

Quick escalates to Standard if it touches > 3 files or uncovers bugs beyond the original scope.

When in doubt, the Builder goes one size up.

## Verification gates

| Gate | Trigger | What runs | Pass criteria |
|------|---------|-----------|---------------|
| 0 | After dependency install | Install command | Exit 0, no unresolved dependency errors |
| 1 | After scaffold | Build/compile command | Exit 0, output artifacts exist |
| 2 | After each feature phase | Build + existing tests | Build succeeds, no regressions |
| 3 | After tests written | Full test suite | All tests pass, coverage meets target |
| 4 | Before declaring done | Clean build from scratch | All gates pass from clean state |

Commands come from the domain profile. If no profile exists, they are defined in the Design document.

Default execution context: run profile/design verification commands from the **project code root** (the project directory that contains the code, build files, and test config), not from the framework repo root, unless the command explicitly sets a different path.

If the exact command cannot be executed because of environment or tooling restrictions, the GateKeeper may use a mechanically equivalent command that preserves verification intent. In that case, the verification log must record:

- the intended command
- the effective command actually executed
- why the substitution was necessary

"Assumed to pass" is never valid evidence. The verification log must record real execution evidence: compact rows are allowed for passing gates, while failures, blocked gates, and command substitutions must include expanded raw output.

### On failure

1. Record failure output in verification log (Failure History section)
2. Root cause analysis — fix the cause, not the symptom
3. Fix the code
4. Re-run the same gate from scratch
5. Record passing output
6. Update Progress section
7. Update the domain profile with the new pitfall/rule/check
8. Proceed to next phase

Failures should be classified when possible as:

- Product Failure
- Environment Failure
- Process Failure

## Pre-implementation checkpoint

Four questions inlined into Standard step 4 (and applicable to all sizes). See [BUILDER.md](BUILDER.md) Standard step 4 for the canonical list and enforcement rule — do not duplicate here to avoid drift.

## Artifacts

### Intent (`docs/[project]-intent.md`)

Source of truth for scope. Template: `templates/INTENT.md`

| Section | Content |
|---------|---------|
| Goal | What and why (1-2 sentences) |
| Behavior | Observable input/output pairs in given/when/then format |
| Decisions | Choices with rejected alternatives and rationale |
| Constraints | MUST / MUST NOT / SHOULD rules |
| Scope | IN and OUT boundaries |
| Adversary Questions Applied | Answers to profile questions (when no Design doc is produced) |
| Domain Pitfalls Applied | How each pitfall is addressed (when no Design doc is produced) |
| Acceptance | Verifiable conditions that define "done" |
| Supersedes | Reference to previous Intent if reworking a feature |

The Anti-Loop Rule requires the Intent to be produced before continued investigation. Unclear decisions are written as open questions and asked to the human.

Every artifact created from the templates records `Artifact Schema Version` so future audits know which contract the file follows.

### Design (`docs/[project]-design.md`)

Architecture + decisions + risks. **Required for Full; optional for Standard** (see Standard step 5 in BUILDER.md). Template: `templates/DESIGN.md`

| Section | Content |
|---------|---------|
| Domain Profile Selection Rationale | Candidates, scores, exclusions, selected profile |
| Skills Loaded | Skills matched and loaded, or "none" if no skills apply |
| Design Status | Which sections or decisions are Provisional, Verified, Blocked, or Revised After Runtime Validation |
| Stack | Technologies with verified versions |
| Structure | Code organization |
| Data Flow | How data moves, especially across technology boundaries |
| Initialization Chain | Exact startup sequence |
| Dependencies | Production and development with versions and purpose |
| Decisions | Architectural choices with rationale and rejected alternatives |
| ADR Summary | Durable record of major Full-project architecture decisions |
| Risks | Identified before implementation (Adversary Lens) |
| Adversary Questions Applied | Every profile adversary question answered against this design |
| Domain Pitfalls Applied | How each profile pitfall is addressed — each row carries an Evidence State (see below) |
| Verification | Gate commands and pass criteria |
| Test Strategy | What to test, how, coverage target, what not to test |

Adversary Questions Applied and Domain Pitfalls Applied are separate mandatory sections. Checking pitfalls does not replace answering adversary questions — they serve different purposes.

Claims across the framework (design decisions, pitfall applicability, review checklist items) carry one of three **Evidence States** — `Verified`, `Provisional`, `Blocked`. `Verified` requires a resoluble `Source:` pointer; `Provisional` is the honest default for review-only reasoning; `Blocked` is used when the tooling needed to verify is absent. Canonical definitions live in [BUILDER.md](BUILDER.md) → Evidence States. Design sections may additionally be marked **Revised After Runtime Validation** when runtime contradicted the initial plan.

**Gate rows in the verification log are a separate taxonomy** — `PASS` / `FAIL` / `BLOCKED` — recording execution events. Claims elsewhere *cite* those gate rows as their Source when asserting `Verified`.

### Verification Log (`docs/[project]-verification.md`)

Mechanical proof. Template: `templates/VERIFICATION_LOG-template.md`

| Section | Content |
|---------|---------|
| Progress | Summary table updated after every gate/phase transition. Resume point for interrupted work |
| Gate entries | Real command output for each gate |
| Self-Review | Intent behavior coverage + domain checklist results + Devil's Advocate (Full only) |
| Failure History | Root causes and fixes |
| Domain Profile Updates | What changed in the profile and why |

One log per project. Completed logs remain as historical evidence.

## Knowledge authority hierarchy

| Priority | Source |
|----------|--------|
| 1 (highest) | Local profile overrides (`framework/domains/` link file) |
| 2 | Base catalog profile (`catalog/`) |
| 3 | Skills (`.github/skills/`, `.agents/skills/`, `.claude/skills/`) |
| 4 (lowest) | Builder defaults in `BUILDER.md` |

Canonical resolution rules and same-level conflict handling live in [BUILDER.md](BUILDER.md) Rules section. Do not restate here.

## Sub-agents and Skills

Sub-agents are an **execution mechanism**, skills are **loadable guidance**, domain profiles are **stack memory**. Their responsibility split, conflict resolution, and anti-patterns are defined in [BUILDER.md](BUILDER.md) (Mode Detection, step 3 skill loading, Rules). Recommended sub-agent patterns and prompt examples live in [../GUIDE.md](../GUIDE.md#step-65-if-your-environment-supports-sub-agents). Prompt template for bounded workers: [templates/SUBAGENT_PROMPT_TEMPLATE.md](templates/SUBAGENT_PROMPT_TEMPLATE.md).

Quick decision table:

| Need | Mechanism |
|------|-----------|
| Reusable stack knowledge across projects | Domain profile |
| Reusable procedure loaded on demand | Skill |
| Isolated multi-step work or parallel execution | Sub-agent |
| Isolated worker following a reusable procedure | Sub-agent + Skill |

## Domain profile selection contract

Profiles in `framework/domains/` can be standalone or links that extend base profiles from `catalog/`:

1. Build candidates from `framework/domains/*.md`, excluding `_template.md` and `README.md`
2. For each candidate, check for an `extends` field:
   - If `extends` exists: load the base profile's metadata from `catalog/`
   - If no `extends`: read the metadata directly from the profile
3. Read selection metadata: `Profile ID`, `Match Keywords`, `Use When`, `Do Not Use When`
4. Exclude profiles where `Do Not Use When` matches explicit user constraints
5. Score remaining profiles by keyword overlap with prompt/stack (`+1` per keyword hit)
6. Select only if highest score is unique and `>= 2`
7. If tied: ask the human; if unavailable, select the profile with the most Common Pitfalls. If below threshold: ask the human; if unavailable, create a new profile (see Creating)
8. If selected profile has `extends`: merge base + local pitfalls (appended) + local overrides (replaced) + local decisions (separate). If standalone: use directly
9. Record selected profile and reason in the Design document (or Intent's Decisions table if no Design is produced)

## Domain profile sections

| Section | Purpose |
|---------|---------|
| Selection Metadata | `Profile ID`, `Match Keywords`, `Use When`, `Do Not Use When` |
| Terminology Mapping | Translates generic "build/test/deploy" to stack-specific commands |
| Verification Commands | Exact commands for each gate (POSIX and PowerShell variants) |
| Common Pitfalls | Frequent errors with What/Correct/Detection pattern |
| Adversary Questions | Domain-specific questions to answer before coding — born from real bugs |
| Integration Rules | Data flow between technologies, type boundaries, startup order |
| Automated Checks | Executable commands for mechanical verification during self-review |
| Decision History | Permanent stack-wide constraints from real project experience |
| Review Checklist | Items to verify during self-review |
| Constraints and Standards | Target environments, compliance requirements, API standards |

### Creating a new profile

When a matching base profile exists in `catalog/`, the Builder creates a profile link in `domains/` using `domains/_template.md` with the `extends` field pointing to the catalog profile.

When no base profile exists, the Builder creates a standalone full profile in `domains/` using `templates/DOMAIN_PROFILE-template.md`. Minimum viable profile: Terminology Mapping + Verification Commands + 2 Common Pitfalls. When the profile proves reusable across projects, move it to `catalog/` and replace it with a link.

### Updating a profile

**Trigger:** any code fix, gate failure, or new discovery — not just end-of-project.

Updates happen immediately, as part of the same change that discovered the gap. A fix without a profile update means the same mistake can happen again.

**Where updates go** depends on their scope:

**Stack-wide discoveries → base profile in `catalog/`:**
- New pitfalls → Common Pitfalls (What/Correct/Detection)
- Reusable command variants required by the stack or target operating environment → Verification Commands
- Missing or incorrect rules → Integration Rules
- Stack-wide decisions → Decision History (date, decision, context, constraint)
- New detection patterns → Automated Checks
- Missing review items → Review Checklist

Do not promote one-off runner quirks, sandbox restrictions, or transient shell workarounds into the profile. Those belong in the verification log as intended/effective command evidence.

**Project-specific discoveries → profile link in `framework/domains/`:**
- Pitfalls unique to this project's context → Local Pitfalls (using the same metadata schema as base pitfalls)
- Gate overrides for this project → Local Overrides
- Decisions that only apply here → Local Decision History

## Resuming interrupted work

1. Read `AGENTS.md` → `BUILDER.md`
2. Look for existing `docs/[project]-verification.md` files
3. Read the **Progress** section at the top of the most recent verification log
4. Load the domain profile referenced in the corresponding design doc
5. Continue from the next incomplete step — do not redo completed gates

If no verification log exists, look for intent and design docs in `docs/`. If nothing exists, start fresh.

## Skills

Skills are optional guidance loaded from `.github/skills/`, `.agents/skills/`, or `.claude/skills/`. Each is a `SKILL.md` with frontmatter `name`/`description`.

Discovery, match rules, precedence, and boundary rules (skill vs profile authority) are defined in [BUILDER.md](BUILDER.md) step 3. See there — do not duplicate.

## Self-review, context pressure, debug sprints

All three are defined canonically in [BUILDER.md](BUILDER.md):

- **Self-Review Protocol** — adversarial re-read, automated checks, Devil's Advocate for Full, Integrity Audit, Promotion Check.
- **Context Pressure Protocol** — artifact degradation order, compact gate format, post-compaction rules.
- **Debug Sprint** (incl. Integration Discovery variant) — when to enter, what to prioritize, exit conditions.

This README intentionally does not restate them to avoid wording drift.
