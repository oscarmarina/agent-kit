# Domain Profiles

Domain profiles provide stack-specific knowledge that the Builder loads based on the project's technology stack. They are the framework's learning mechanism — each project can enrich them with new pitfalls, decisions, and automated checks.

## How Profiles Are Used

1. The Builder reads the user's prompt and identifies the technology stack
2. The Builder applies the operational matching contract in this directory
3. If found, the profile overrides generic assumptions:
   - **Verification commands** replace generic "run build" instructions
   - **Common pitfalls** are checked against the design before coding
   - **Adversary questions** must be answered against the specific design (documented in the Design doc)
   - **Automated checks** are executed mechanically during self-review
   - **Decision history** applies permanent constraints from past projects
   - **Integration rules** define data flow between technologies

## Operational Matching Contract

Selection must be deterministic and auditable:

1. Candidate set: all `.md` files in this directory except `_template.md` and `README.md`
2. Metadata required per profile:
   - `Profile ID`
   - `Match Keywords`
   - `Use When`
   - `Do Not Use When`
3. Remove any profile whose `Do Not Use When` matches explicit user constraints
4. Score remaining profiles by keyword overlap with prompt/stack (`+1` per keyword hit)
5. Select only if highest score is unique and `>= 2`
6. If tied or below threshold: ask the human to choose; if no clarification is available, create a new profile from `_template.md` instead of forcing a weak match
7. Record the selected profile and reason in the project design doc

## Naming Convention

`[domain]-[stack].md`

The filenames below are naming examples, not bundled profiles in this starter pack.

Examples:
- `web-angular-lit.md`
- `web-react-nextjs.md`
- `plc-siemens-scl.md`
- `embedded-stm32-freertos.md`
- `backend-python-fastapi.md`

## Who Creates Profiles

1. **The human (proactively)** — Create profiles for stacks you use frequently. Use `_template.md`.
2. **The Builder (during a project)** — When no profile exists for the stack, the Builder creates one during the Design phase. Pitfalls discovered during implementation enrich it.

## Profile Sections

| Section | Purpose |
|---------|---------|
| Selection Metadata | Deterministic profile routing (`Profile ID`, keywords, use/do-not-use rules) |
| Terminology Mapping | Translates generic "build/test/deploy" to stack language |
| Verification Commands | Exact commands for each gate (0-4) |
| Common Pitfalls | Frequent errors with detection patterns |
| Adversary Questions | Domain-specific questions to answer BEFORE coding — born from real bugs. Must be answered in the Design doc's "Adversary Questions Applied" section |
| Integration Rules | Data flow between technologies, build scoping, init order |
| Automated Checks | Executable commands for mechanical verification |
| Decision History | Permanent constraints from real project experience |
| Review Checklist | Items to verify during self-review |

## When No Profile Exists

The Builder creates one using `_template.md` during the Design phase. Minimum viable profile: Terminology Mapping + Verification Commands + 2 Common Pitfalls.
