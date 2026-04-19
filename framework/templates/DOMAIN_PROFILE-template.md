# Domain Profile: [Domain - Stack]

**Artifact Schema Version:** 1.1.0
**Domain:** [e.g., Web Frontend, PLC/Industrial Automation, Embedded Systems, Backend Services, Mobile]
**Stack:** [e.g., Angular + Lit, Siemens TIA Portal SCL, STM32 + FreeRTOS, Python FastAPI, React Native]
**Standards:** [e.g., IEC 61131-3, MISRA C, OWASP Top 10, WCAG 2.1]

## Selection Metadata (Operational Contract)

**Profile ID:** [unique id, usually filename without `.md`; e.g., `web-angular-lit`]
**Profile Version:** 1.0.0
<!-- Bump when pitfalls, commands, or overrides change. Profile links record this as `catalog_version` to detect drift. Use semver: patch = typo/clarification, minor = additive (new pitfall), major = breaking (renamed/removed field). -->

**Match Keywords:** [comma-separated terms used for routing; e.g., `angular, lit, web components, vite`]
**Use When:** [one sentence describing when this profile SHOULD be selected]
**Do Not Use When:** [one sentence describing when this profile MUST NOT be selected]
**Last Verified:** [date when this profile was last checked against the actual codebase; e.g., `2026-03-28`]
<!-- Update this date whenever you modify this profile or re-verify pitfalls/checks against the codebase -->

## Terminology Mapping

Map generic framework terms to domain-specific terms:

| Framework Term | Domain Term | Notes |
|---|---|---|
| Build/Compile | [e.g., `npm run build` / "Download to PLC" / "Cross-compile"] | |
| Test suite | [e.g., `npm test` / "Simulation test" / "Hardware-in-loop test"] | |
| Dev server | [e.g., `npm run dev` / "PLC simulator" / "QEMU emulator"] | |
| Package/dependency | [e.g., npm package / Function Block library / HAL driver] | |
| Import/module | [e.g., ES import / USE statement / #include / Python import] | |
| Deployment | [e.g., Static hosting / Download to PLC / Flash to device] | |

## Verification Commands

Exact commands for each Verification Gate. These override any generic assumptions.

Unless a command explicitly says otherwise, write and execute these commands relative to the **project code root** (the directory that contains the project's `package.json`, build files, source tree, or equivalent entrypoint) rather than the framework repo root.

Only record command forms that are expected to be reusable for this stack or its real target operating environments. Do not encode one-off runner quirks, sandbox restrictions, or transient shell workarounds here; those belong in the verification log as intended/effective command evidence.

Each gate command carries a **Type** that controls both the gate-row outcome (`PASS` / `FAIL` / `BLOCKED`) and the default Evidence State of any claim citing that gate as Source (see `BUILDER.md → Evidence States`):

- `automated` — executable without human action; runnable in any CI shell. Gate row: `PASS` on exit 0, `FAIL` otherwise. Claims citing a `PASS` automated gate may be marked `Verified`.
- `manual` — requires a human to run a procedure (e.g., drive a simulator, execute a commissioning checklist). Gate row: `PASS` only when a log entry with human sign-off is present; otherwise `FAIL` / `BLOCKED`. Dependent claims stay `Provisional` until that sign-off is captured.
- `requires-proprietary-tooling` — requires vendor/platform tooling (e.g., TIA Portal, Xcode on macOS, hardware-in-the-loop rig). Gate row: `BLOCKED` when tooling is absent, with the intended/effective command triad. Dependent claims are `Provisional` or `Blocked`, never `Verified`, in that session.

**GATE 0 (Dependencies):**
- Type: `automated`
- Command: `[e.g., npm install, pip install -r requirements.txt]`
- Expected output: [what success looks like]

**GATE 1 (Scaffold):**
- Type: `automated`
- Command: `[e.g., npm run build, cargo check]`
- Expected output: [what success looks like]

**GATE 2 (Feature):**
- Type: `automated`
- Command: `[same as Gate 1 plus any incremental checks]`
- Expected output: [no regressions]

**GATE 3 (Tests):**
- Type: `[automated | manual | requires-proprietary-tooling]`
- Command: `[e.g., npm test, pytest, cargo test | MANUAL: run PLCSIM fault-path checklist]`
- Expected output: [all tests pass]
- Coverage command: `[e.g., npm test -- --coverage, pytest --cov]`

**GATE 4 (Final):**
- Type: `[automated | manual | requires-proprietary-tooling]`
- Clean command (POSIX): `[e.g., rm -rf node_modules dist && npm install && npm run build && npm test]`
- Clean command (PowerShell): `[e.g., if (Test-Path node_modules) { Remove-Item node_modules -Recurse -Force }; if (Test-Path dist) { Remove-Item dist -Recurse -Force }; npm install; npm run build; npm test]`
- Expected output: [everything passes from clean state]

If different command variants are needed, prefer documenting stable target-environment variants such as POSIX vs PowerShell. Do not add variants that exist only because a specific agent runner or shell session is unreliable.

## Common Pitfalls

Domain-specific mistakes that LLMs frequently make. Each pitfall has a detection pattern for mechanical verification.

**Severity levels:** `critical` (data loss, security, silent wrong behavior) / `major` (build or test failures, API breakage) / `minor` (style, performance, non-blocking warnings).

The GateKeeper increments `occurrence_count` each time it detects this pitfall during gate execution. The Builder increments it when discovering the pitfall during design-time review. When `occurrence_count` reaches 3 across different projects, the pitfall is a candidate for promotion to the catalog base profile (see `catalog/README.md` Promotion Protocol).

**Confidence levels:** every pitfall carries a confidence tier that records how the entry was validated:
- `confirmed` — originated from a gate failure with real output in a verification log; Source must reference the exact entry
- `inferred` — derived from design analysis, documentation reading, or multiple circumstantial observations; no single gate log
- `heuristic` — added proactively based on general domain knowledge; not yet validated by a real failure in this stack

A `heuristic` pitfall with zero occurrence hits after two projects is a removal candidate — it may be wrong or may target code that no longer exists.

### Pitfall 1: [Name]
- **Severity:** [critical / major / minor]
- **Occurrence count:** [number — starts at 1 when first documented; increment on each new detection]
- **Confidence:** [confirmed / inferred / heuristic]
- **Source:** [e.g., `docs/[project]-verification.md → Gate 3 FAILED 2026-03-09` | `Design review YYYY-MM-DD` | `Prompt constraint 2026-04-19`]
  <!-- Source type is implied by Confidence, and controls Promotion eligibility:
         confirmed → MUST reference a verification-log failure line (e.g., "docs/foo-verification.md → Gate 3 FAILED 2026-03-09"). Promotion-eligible.
         inferred  → references design/doc analysis (e.g., "API docs review 2026-04-11" or "Design section 4.2 review"). NOT promotion-eligible.
         heuristic → preventive origin (e.g., "Prompt constraint 2026-04-19" or "Carried from prior stack experience"). NOT promotion-eligible.
       A pitfall cannot graduate to the catalog until a real gate failure upgrades it to Confidence: confirmed with a verification-log Source. -->
- **What goes wrong:** [Description of the error and why it's tempting]
- **Correct approach:** [How to do it right]
- **Detection:** [How to spot this — specific search pattern or command]

### Pitfall 2: [Name]
- **Severity:** [critical / major / minor]
- **Occurrence count:** [number]
- **Confidence:** [confirmed / inferred / heuristic]
- **Source:** [reference]
- **What goes wrong:** [Description]
- **Correct approach:** [How to do it right]
- **Detection:** [Search pattern or verification step]

*(Add as many pitfalls as needed. Each one learned from real project experience.)*

## Adversary Questions

Domain-specific questions to answer BEFORE writing code. These target traps that generic Adversary Lens questions miss. Each question was born from a real bug in a real project.

- [e.g., "What happens if the iframe host strips the Referer header?"]
- [e.g., "What happens if the PLC watchdog timer expires during a long computation?"]
- [e.g., "What happens if the API response schema changes without a version bump?"]

*(Add questions as they emerge from real failures. A good adversary question would have caught a pitfall before it became a bug.)*

**Quality filter:** Only add questions here that an LLM would NOT generate from the generic Adversary Lens alone. Questions like "What happens if the server returns 404?" are already covered by generic adversary reasoning. Questions like "What happens when the annotation extension dispatches events to appConfig properties that were renamed between major versions?" require domain-specific experience — those belong here. If an LLM would ask it anyway without the profile, it adds noise rather than signal.

**These questions must be answered in the Design document's "Adversary Questions Applied" section — not just read. Checking pitfalls is necessary but not sufficient; adversary questions target how traps interact with a specific design.**

## Integration Rules

*(For projects combining multiple technologies/frameworks)*

### Data Flow: [Tech A] to [Tech B]
- [Method and any type/format conversions required]

### Data Flow: [Tech B] to [Tech A]
- [Method, e.g., events, callbacks, interrupts, shared memory]

### Type Boundaries
- [How to handle type declarations or format conversions across the boundary]

### Build/Compile Scoping
- [Each technology's toolchain only processes its own source files]
- [e.g., Angular compiler must only process files in `src/app/`, exclude Lit components]

### Startup/Bootstrap Order
- [Exact initialization sequence, e.g., zone.js → Lit imports → Angular bootstrap]

## Automated Checks

Detection patterns that can be executed mechanically. Each maps to a pitfall, integration rule, or review checklist item. Run these during self-review.

| Check | Command | Expected Result |
|-------|---------|-----------------|
| [e.g., No attr binding for non-strings] | `[grep command]` | [Only aria-* results, or no results] |
| [e.g., Prerequisite import order] | `[head/grep command]` | [Specific import appears first] |

## Decision History

Decisions that apply to ALL projects with this stack. Each entry is a permanent constraint derived from real project experience. New entries are added when a project discovers a stack-wide decision.

Do not store transient tooling incidents here. A decision belongs here only when it should constrain future projects on the same stack.

| Date | Decision | Context | Constraint |
|------|----------|---------|------------|
| [e.g., 2026-02-15] | [e.g., Browser mode only for tests] | [Why this was decided] | [MUST/MUST NOT statement] |

## Review Checklist

Items to verify during self-review, specific to this domain. Each item pairs an assertion with a Detection method so the agent can record an Evidence State in the verification log. Prefer executable detections (`rg ...`, file existence, automated command) over "manual review" — the latter forces Provisional status under the rules in `BUILDER.md → Evidence States`.

| Item | Detection |
|------|-----------|
| [Domain-specific check 1] | [`rg -n "pattern" src/` / file-existence test / automated check command / "manual review"] |
| [Domain-specific check 2] | [...] |
| [Domain-specific check 3] | [...] |

At Self-Review time (BUILDER.md Self-Review Protocol step 4) the agent fills in the results table in the **verification log**, not here — one row per checklist item, with Status + Evidence columns. The profile defines *what* to check; the verification log records *what was found*.

## Constraints and Standards

[Domain-specific constraints — target environments, required APIs, compliance standards, memory limits, real-time deadlines, etc.]
