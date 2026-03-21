# Agent Kit — Reference

## Contents

| File | Purpose |
|------|---------|
| `BUILDER.md` | Process contract — lenses, gates, pre-implementation checkpoint, rules |
| `GATEKEEPER.md` | Verification agent — executes commands, enforces gates, updates pitfalls |
| `domains/_template.md` | Template for creating profile links that extend catalog profiles |
| `domains/README.md` | How domain profiles work — types, matching, merge rules |
| `domains/*.md` | Domain profiles — standalone or links extending catalog base profiles |
| `templates/INTENT.md` | Template for Change Intent Records |
| `templates/DESIGN.md` | Template for Design documents |
| `templates/VERIFICATION_LOG-template.md` | Template for gate execution logs with progress tracking |
| `templates/DOMAIN_PROFILE-template.md` | Template for creating standalone full domain profiles |

## Setup

1. Copy `framework/` into the target repo.
2. Create `AGENTS.md` at the repo root:
   ```markdown
   # Agent Instructions

   The framework operates on a strict Adversarial Verification Loop:
   - For design, planning, and code implementation tasks: Read and follow `framework/BUILDER.md`.
   - For execution, testing, and mechanical verification tasks: Read and follow `framework/GATEKEEPER.md`.

   Project artifacts (intent, design, verification) go in `docs/`.
   ```
3. Create `docs/` for project-specific artifacts.
4. Create a profile link in `framework/domains/` that extends a relevant profile from the [catalog](../catalog/), or let the Builder create one during the first project. See `domains/_template.md` for the link format.

Resulting structure:

```
repo/
├── AGENTS.md               ← entry point (5 lines)
├── framework/              ← framework (reusable)
│   ├── BUILDER.md
│   ├── GATEKEEPER.md
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

## Prompts

| Scenario | Prompt |
|----------|--------|
| New project | `Read AGENTS.md. Build a [description]. Stack: [technologies].` |
| New feature | `Read AGENTS.md. [Description of what you want].` |
| Bug fix | `Read AGENTS.md. [Description of the problem].` |
| Resume work | `Read AGENTS.md. Continue where the last session left off.` |

## Process sizes

| Size | Criteria | Artifacts produced |
|------|----------|--------------------|
| Quick | < 3 files, clear intent | Code + Gate 2 minimum |
| Standard | Feature-sized | Intent + Design + Code + Verification Log |
| Full | New project / major rearchitecture | Standard + ADRs + Devil's Advocate self-review |

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

"Assumed to pass" is never valid evidence. Real output must be pasted in the verification log.

### On failure

1. Record failure output in verification log (Failure History section)
2. Root cause analysis — fix the cause, not the symptom
3. Fix the code
4. Re-run the same gate from scratch
5. Record passing output
6. Update Progress section
7. Update the domain profile with the new pitfall/rule/check
8. Proceed to next phase

## Pre-implementation checkpoint

Four questions answered before writing any implementation code:

1. **Do my dependencies already solve this?** Read the public API (`.d.ts`, package docs, `npm view`). If a library provides the functionality, use it.
2. **What environment assumption could be wrong?** Identify at least one assumption about runtime, host, or deployment target.
3. **Have I checked the domain profile pitfalls?** Scan every Common Pitfall and Adversary Question against the plan.
4. **Is this still the right size?** Re-evaluate Quick vs Standard vs Full.

This is a mental gate, not a document.

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
| Acceptance | Verifiable conditions that define "done" |
| Supersedes | Reference to previous Intent if reworking a feature |

The Anti-Loop Rule requires the Intent to be produced before continued investigation. Unclear decisions are written as open questions and asked to the human.

### Design (`docs/[project]-design.md`)

Architecture + decisions + risks. Template: `templates/DESIGN.md`

| Section | Content |
|---------|---------|
| Domain Profile Selection Rationale | Candidates, scores, exclusions, selected profile |
| Stack | Technologies with verified versions |
| Structure | Code organization |
| Data Flow | How data moves, especially across technology boundaries |
| Initialization Chain | Exact startup sequence |
| Dependencies | Production and development with versions and purpose |
| Decisions | Architectural choices with rationale and rejected alternatives |
| Risks | Identified before implementation (Adversary Lens) |
| Adversary Questions Applied | Every profile adversary question answered against this design |
| Domain Pitfalls Applied | How each profile pitfall is addressed |
| Verification | Gate commands and pass criteria |
| Test Strategy | What to test, how, coverage target, what not to test |

Adversary Questions Applied and Domain Pitfalls Applied are separate mandatory sections. Checking pitfalls does not replace answering adversary questions — they serve different purposes.

### Verification Log (`docs/[project]-verification.md`)

Mechanical proof. Template: `templates/VERIFICATION_LOG-template.md`

| Section | Content |
|---------|---------|
| Progress | Summary table updated after every gate/phase transition. Resume point for interrupted work |
| Gate entries | Real command output for each gate |
| Self-Review | Domain checklist results + Devil's Advocate (Full only) |
| Failure History | Root causes and fixes |
| Domain Profile Updates | What changed in the profile and why |

One log per project. Completed logs remain as historical evidence.

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
7. If tied or below threshold: ask the human. If no clarification, create a new profile (see Creating)
8. If selected profile has `extends`: merge base + local pitfalls (appended) + local overrides (replaced) + local decisions (separate). If standalone: use directly
9. Record selected profile and reason in the Design document

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
- Adjusted commands → Verification Commands
- Missing or incorrect rules → Integration Rules
- Stack-wide decisions → Decision History (date, decision, context, constraint)
- New detection patterns → Automated Checks
- Missing review items → Review Checklist

**Project-specific discoveries → profile link in `framework/domains/`:**
- Pitfalls unique to this project's context → Local Pitfalls
- Gate overrides for this project → Local Overrides
- Decisions that only apply here → Local Decision History

## Resuming interrupted work

1. Read `AGENTS.md` → `BUILDER.md`
2. Look for existing `docs/[project]-verification.md` files
3. Read the **Progress** section at the top of the most recent verification log
4. Load the domain profile referenced in the corresponding design doc
5. Continue from the next incomplete step — do not redo completed gates

If no verification log exists, look for intent and design docs in `docs/`. If nothing exists, start fresh.

## Self-review protocol

After implementation, the Builder shifts to Adversary Lens:

1. Re-read the Intent. Does the code deliver every Behavior? Respect every Constraint?
2. Run every Automated Check from the domain profile (execute command, verify result)
3. Check every Common Pitfall against the codebase
4. Verify every Review Checklist item
5. **Full projects only:** Devil's Advocate section (3 scenarios, weakest link, attack vector) + minimum 3 genuine findings
