# Verification Log: [Project Name]

**Artifact Schema Version:** 1.1.0

This log captures the actual output of every verification gate. It is the source of truth for project completion status.

**Rule:** No entry may be written without executing the command and pasting real output. "Assumed to pass" is not an entry.

**Compact format exception:** The compact table (see Gates section below) does not paste raw output — that is its purpose. It is still subject to the rule: never write a compact row without having run the command. A row with exit 0 that was not executed is fabrication, not evidence.

**Format:** This template provides both **compact** and **expanded** gate formats. Use compact for passing gates; use expanded for failures (where raw output has diagnostic value). See Context Pressure Protocol in BUILDER.md.

**Two taxonomies, one log:**
- **Gate rows** (compact and expanded) record execution events — `PASS` / `FAIL` / `BLOCKED`. A gate either ran cleanly, rejected, or could not be executed faithfully. No Evidence State goes here.
- **Self-Review tables** (Domain Checklist, Review Checklist, Pitfall Applicability) record *claims about the system* — `Verified` / `Provisional` / `Blocked` per `BUILDER.md → Evidence States`. A claim is `Verified` only when its `Evidence` column points at a resoluble Source (a gate row above, a file path, or a captured command output).

A `PASS` gate is what lets a dependent claim move from Provisional to Verified. The gate row is the Source; it is not itself Verified.

---

## Progress

**Current phase:** [e.g., "Gated build — Gate 1 passed, implementing feature for Gate 2"]
**Last updated:** [Date/Time]

| Step | Status |
|------|--------|
| Intent | — |
| Skills loaded | — |
| Design | — |
| Gate 0: Dependencies | — |
| Gate 1: Scaffold | — |
| Gate 2: Feature | — |
| Gate 3: Tests | — |
| Gate 4: Clean build | — |
| Self-Review | — |
| Domain update | — |

**Update this section after every gate or phase transition. When resuming interrupted work, read this section first.**

---

## Gates (compact format — use for passing gates)

| Gate | Command | Exit | Status | Tests | Coverage | Notes |
|------|---------|------|--------|-------|----------|-------|
| 0 | `[command]` | [0] | [PASS] | — | — | [notes] |
| 1 | `[command]` | [0] | [PASS] | — | — | [notes] |
| 2 | `[command]` | [0] | [PASS] | [X/Y] | — | [notes] |
| 3 | `[command]` | [0] | [PASS] | [X/Y] | [X%] | [notes] |
| 4 | `[command]` | [0] | [PASS] | [X/Y] | — | [notes] |

*Use the expanded format below ONLY for failed gates or when the command/output needs detailed recording (substitution reason, classification, raw output). Delete whichever format you don't use for each gate.*

---

## Gate 0: Dependencies (expanded format — use for failures)
**Executed:** [Date/Time]
**Intended command:** `[command defined by the profile or design]`
**Effective command:** `[actual command run]`
**Substitution reason:** [Only if the effective command differs from the intended command]
**Exit code:** [0 or error code]
**Status:** [PASS / FAIL / BLOCKED]
**Classification:** [Product Failure / Environment Failure / Process Failure / N/A]

<details>
<summary>Output (click to expand)</summary>

```
[Paste actual command output here. Truncate to last 50 lines if very long, but include any errors/warnings in full.]
```

</details>

**Notes:** [Any observations — warnings that are acceptable, known issues, etc.]

---

## Gate 1: Scaffold Verification
**Executed:** [Date/Time]
**Intended command:** `[command defined by the profile or design]`
**Effective command:** `[actual command run]`
**Substitution reason:** [Only if the effective command differs from the intended command]
**Exit code:** [0 or error code]
**Status:** [PASS / FAIL / BLOCKED]
**Classification:** [Product Failure / Environment Failure / Process Failure / N/A]

<details>
<summary>Output</summary>

```
[Paste actual output]
```

</details>

**Notes:**

---

## Gate 2: Feature Verification
**Executed:** [Date/Time]
**Intended command:** `[command defined by the profile or design]`
**Effective command:** `[actual command run]`
**Substitution reason:** [Only if the effective command differs from the intended command]
**Exit code:** [0 or error code]
**Status:** [PASS / FAIL / BLOCKED]
**Classification:** [Product Failure / Environment Failure / Process Failure / N/A]

<details>
<summary>Output</summary>

```
[Paste actual output]
```

</details>

**Notes:**

---

## Gate 3: Test Verification
**Executed:** [Date/Time]
**Intended command:** `[command defined by the profile or design]`
**Effective command:** `[actual command run]`
**Substitution reason:** [Only if the effective command differs from the intended command]
**Exit code:** [0 or error code]
**Status:** [PASS / FAIL / BLOCKED]
**Classification:** [Product Failure / Environment Failure / Process Failure / N/A]
**Tests passed:** [X/Y]
**Coverage:** [X% or "not measured"]

<details>
<summary>Output</summary>

```
[Paste actual test output including pass/fail counts]
```

</details>

**Notes:**

---

## Gate 4: Final Verification (Clean Build)
**Executed:** [Date/Time]
**Intended command:** `[Use the domain profile clean-build command sequence]`
**Effective command:** `[actual clean-build command(s) run]`
**Substitution reason:** [Only if the effective command differs from the intended command]
**Exit code:** [0 or error code]
**Status:** [PASS / FAIL / BLOCKED]
**Classification:** [Product Failure / Environment Failure / Process Failure / N/A]
**Tests passed:** [X/Y]

<details>
<summary>Output</summary>

```
[Paste full output from clean build + test]
```

</details>

**Notes:**

---

## Self-Review

### Domain Checklist Results
[Run every Automated Check from the domain profile. Paste command + result.]

| Check | Command | Result | Status |
|-------|---------|--------|--------|
| [e.g., No attr binding] | `[command]` | [what was found] | [Verified / Provisional / Blocked] |

### Review Checklist
[One row per item from the domain profile's Review Checklist. Apply Evidence State rules from BUILDER.md → Evidence States: `Verified` requires a resoluble Source (not just a review statement); "manual review" or a Blocked dependency gate forces `Provisional`.]

| Item | Applies? | Status | Evidence |
|------|----------|--------|----------|
| [Profile checklist item] | [Yes / No / N/A] | [Verified / Provisional / Blocked] | [`docs/[project]-verification.md → Gate 2 PASS` / `src/path/file.ts:42` / "manual review — no runtime evidence, depends on Gate 3 which is Blocked"] |

### Pitfall Applicability
[One row per Common Pitfall in the active domain profile. Same Evidence State rules apply.]

| Pitfall | Applies? | Status | Evidence |
|---------|----------|--------|----------|
| [Profile pitfall name] | [Yes / No] | [Verified / Provisional / Blocked] | [Source pointer or "manual review"] |

### Devil's Advocate (Full projects only)
1. **What happens when:** [scenario 1], [scenario 2], [scenario 3]
2. **The weakest link is:** [identification and reasoning]
3. **If I had to break this, I would:** [attack vector]

### Findings (Full projects only)
[List genuine vulnerabilities or logic flaws. If none found, document the critical attack vectors investigated and why they are not exploitable. Do not fabricate findings to meet a quota.]

| # | Severity | Finding / Investigation | Impact / Conclusion |
|---|----------|------------------------|---------------------|
| 1 | [P0/P1/P2 or "Investigated"] | [Description or attack vector checked] | [What it affects, or why it's secure] |

---

## Failure History

*(Record any gate failures and their resolutions here)*

### [Date] Gate [N] FAILED
**Classification:** [Product Failure / Environment Failure / Process Failure]
**Error:** [What went wrong]
**Root Cause:** [Why — not symptoms, actual cause]
**Fix:** [What was done]
**Re-run result:** [PASS — see Gate N above for passing output]

---

## Domain Profile Updates

*(List every change made to the domain profile during this project. This section closes the learning loop: failures and discoveries above flow into the profile for the next project.)*

| What Changed | Section Updated | Trigger |
|---|---|---|
| [e.g., Added Pitfall 10: CORS on widget assets] | Common Pitfalls | [e.g., Gate 2 failure — script blocked cross-origin] |
| [e.g., Added adversary question about sandbox referrer] | Adversary Questions | [e.g., Widget blank in ChatGPT due to empty document.referrer] |

**If this section is empty at the end of the project, ask: did we really learn nothing? Review Failure History for missed updates.**
