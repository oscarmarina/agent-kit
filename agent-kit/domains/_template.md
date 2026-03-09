# Domain Profile: [Domain - Stack]

**Domain:** [e.g., Web Frontend, PLC/Industrial Automation, Embedded Systems, Backend Services, Mobile]
**Stack:** [e.g., Angular + Lit, Siemens TIA Portal SCL, STM32 + FreeRTOS, Python FastAPI, React Native]
**Standards:** [e.g., IEC 61131-3, MISRA C, OWASP Top 10, WCAG 2.1]

## Selection Metadata (Operational Contract)

**Profile ID:** [unique id, usually filename without `.md`; e.g., `web-angular-lit`]
**Match Keywords:** [comma-separated terms used for routing; e.g., `angular, lit, web components, vite`]
**Use When:** [one sentence describing when this profile SHOULD be selected]
**Do Not Use When:** [one sentence describing when this profile MUST NOT be selected]

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

**GATE 0 (Dependencies):**
- Command: `[e.g., npm install, pip install -r requirements.txt]`
- Expected output: [what success looks like]

**GATE 1 (Scaffold):**
- Command: `[e.g., npm run build, cargo check]`
- Expected output: [what success looks like]

**GATE 2 (Feature):**
- Command: `[same as Gate 1 plus any incremental checks]`
- Expected output: [no regressions]

**GATE 3 (Tests):**
- Command: `[e.g., npm test, pytest, cargo test]`
- Expected output: [all tests pass]
- Coverage command: `[e.g., npm test -- --coverage, pytest --cov]`

**GATE 4 (Final):**
- Clean command (POSIX): `[e.g., rm -rf node_modules dist && npm install && npm run build && npm test]`
- Clean command (PowerShell): `[e.g., if (Test-Path node_modules) { Remove-Item node_modules -Recurse -Force }; if (Test-Path dist) { Remove-Item dist -Recurse -Force }; npm install; npm run build; npm test]`
- Expected output: [everything passes from clean state]

## Common Pitfalls

Domain-specific mistakes that LLMs frequently make. Each pitfall has a detection pattern for mechanical verification.

### Pitfall 1: [Name]
- **What goes wrong:** [Description of the error and why it's tempting]
- **Correct approach:** [How to do it right]
- **Detection:** [How to spot this — specific search pattern or command]

### Pitfall 2: [Name]
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

| Date | Decision | Context | Constraint |
|------|----------|---------|------------|
| [e.g., 2026-02-15] | [e.g., Browser mode only for tests] | [Why this was decided] | [MUST/MUST NOT statement] |

## Review Checklist

Items to verify during self-review, specific to this domain. Each should be verifiable by inspection or automated check.

- [ ] [Domain-specific check 1]
- [ ] [Domain-specific check 2]
- [ ] [Domain-specific check 3]

## Constraints and Standards

[Domain-specific constraints — target environments, required APIs, compliance standards, memory limits, real-time deadlines, etc.]
