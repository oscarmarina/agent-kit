Read AGENTS.md. [Quick | Standard | Full] project: <project-name>.

## Goal
<One or two sentences: what must exist when this is done, and for whom.>

## Deliverable shape
<What the output looks like — a repo, a document, a PLC program, a config,
a diagram. Name concrete files/artifacts if you can.>

## Stack / environment
<Languages, frameworks, tools, hardware, standards. Versions when they matter.
For non-software domains: platform, target runtime, norms (e.g. IEC 61131-3,
Siemens TIA Portal V18, ISA-88 batch model).>

## Constraints (MUST)
- <Hard rule 1 — invariant that cannot be violated>
- <Hard rule 2>

## Constraints (MUST NOT)
- <Anti-pattern 1 — things that look reasonable but break this domain>
- <Anti-pattern 2>

## Domain profile
- Do not pre-select a profile by filename guess.
- Follow the deterministic profile-selection algorithm in `framework/BUILDER.md` step 2.
- If BUILDER step 2 selects or creates a profile, apply every pitfall before writing code.

## Verification expectation
<How "done" is proven. A command, a test suite, a simulation run,
a checklist, a sign-off. If you don't know yet, say so and ask the
gate to propose one.>

## Out of scope
<What this project does NOT cover. Prevents drift.>