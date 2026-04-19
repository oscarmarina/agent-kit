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
- Check `framework/domains/` and `catalog/` for a matching profile.
- If one exists, load it and apply every pitfall.
- If none exists, create a minimal skeleton profile before writing code
  (see BUILDER.md step 2).

## Verification expectation
<How "done" is proven. A command, a test suite, a simulation run,
a checklist, a sign-off. If you don't know yet, say so and ask the
gate to propose one.>

## Out of scope
<What this project does NOT cover. Prevents drift.>