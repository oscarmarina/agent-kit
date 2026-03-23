# Domain Profiles

Domain profiles provide stack-specific knowledge that the Builder loads based on the project's technology stack. They are the framework's learning mechanism — each project can enrich them with new pitfalls, decisions, and automated checks.

## Two Types of Profile

This directory can contain two types of profile:

**Standalone profiles** — full profiles with all sections (Selection Metadata, Terminology, Verification Commands, Pitfalls, etc.). Created when no base profile exists in `catalog/`. Template: `framework/templates/DOMAIN_PROFILE-template.md`.

**Profile links** — lightweight files that extend a base profile from `catalog/` with project-specific additions. Created when a matching base exists in `catalog/`. Template: `_template.md` in this directory.

```
framework/domains/                    ← Profiles for this project
├── _template.md                      ← Template for profile links
├── README.md                         ← This file
├── web-react-nextjs.md               ← Example: standalone (no catalog base)
└── my-widget.md                      ← Example: link (extends a catalog base)
```

## How It Works

### When a catalog profile exists for your stack

1. Create a profile link in this directory using `_template.md`
2. Set `extends:` to the Profile ID of a catalog profile
3. Add only project-specific pitfalls, overrides, and decisions
4. The Builder loads the base from `catalog/` and applies your local additions on top

**Resolution rules for links:**
- **Local Pitfalls** and **Local Adversary Questions** are **appended** to the base profile's lists
- **Local Overrides** **replace** the corresponding section in the base profile
- **Local Decision History** is kept separate from the base profile's Decision History
- Sections not mentioned locally inherit from the base unchanged

### When no catalog profile exists

1. Create a standalone profile in this directory using `framework/templates/DOMAIN_PROFILE-template.md`
2. Fill in at minimum: Terminology Mapping + Verification Commands + 2 Common Pitfalls
3. The profile grows as the project discovers new pitfalls and decisions
4. When you want to reuse it across projects, move it to `catalog/` and replace it here with a link

### Example: Profile Link

```markdown
# Domain Profile: My Widget App

## Profile Link

extends: apps-sdk-mcp-lit-vite
catalog_version: 1.0.0

## Local Pitfalls

### Pitfall L1: Auth token expiry during render
- **What goes wrong:** Widget crashes if token expires between init and first paint
- **Correct approach:** Check token TTL before render, refresh if < 30s remaining
- **Detection:** `grep -r "getToken" src/ | grep -v "checkExpiry"`

## Local Overrides

### Verification Commands (overrides)

**GATE 3 (Tests):**
- Command: `npm test -- --project=widget`
- Expected output: all widget tests pass
```

This profile inherits all 11 pitfalls, 7 adversary questions, terminology, integration rules, and automated checks from `apps-sdk-mcp-lit-vite` — and adds one local pitfall plus a gate override.

## How Profiles Are Used

1. The Builder reads the user's prompt and identifies the technology stack
2. The Builder applies the operational matching contract (see below)
3. If the selected profile has `extends`:
   - Loads the base profile from `catalog/[extends].md`
   - Applies local overrides and additions
   - Uses the merged result for the project
4. If the selected profile is standalone, uses it directly
5. The profile overrides generic assumptions:
   - **Verification commands** replace generic "run build" instructions
   - **Common pitfalls** (base + local, or standalone) are checked against the design before coding
   - **Adversary questions** (base + local, or standalone) must be answered against the specific design
   - **Automated checks** are executed mechanically during self-review
   - **Decision history** applies permanent constraints from past projects
   - **Integration rules** define data flow between technologies

## Operational Matching Contract

Selection must be deterministic and auditable:

1. Candidate set: all `.md` files in this directory except `_template.md` and `README.md`
2. For each candidate, check for an `extends` field:
   - If `extends` exists: load the base profile's selection metadata from `catalog/`
   - If no `extends`: read the selection metadata directly from the profile
3. Read selection metadata: `Profile ID`, `Match Keywords`, `Use When`, `Do Not Use When`
4. Remove any profile whose `Do Not Use When` matches explicit user constraints
5. Score remaining profiles by keyword overlap with prompt/stack (`+1` per keyword hit)
6. Select only if highest score is unique and `>= 2`
7. If tied or below threshold: ask the human to choose; if no clarification is available, create a new profile (see below)
8. Record the selected profile and reason in the project design doc

## Keeping Profiles Updated

- For profile links: compare `catalog_version` against the current catalog profile to detect updates
- **Local pitfalls** that prove useful across projects should be contributed back to the catalog profile via PR
- **Stack-wide decisions** discovered locally belong in the catalog profile's Decision History, not in the local one
- Standalone profiles that prove reusable should be moved to `catalog/` and replaced here with a link

## Who Creates Profiles

1. **The human (proactively)** — Create profiles for your project's stack using the appropriate template
2. **The Builder (during a project)** — When a matching catalog profile exists, creates a profile link using `_template.md`. When no catalog profile exists, creates a standalone profile using `framework/templates/DOMAIN_PROFILE-template.md`
