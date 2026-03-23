# Verification Log: [Project Name]

This log captures the actual output of every verification gate. It is the source of truth for project completion status.

**Rule:** No entry may be written without executing the command and pasting real output. "Assumed to pass" is not an entry.

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

## Gate 0: Dependencies
**Executed:** [Date/Time]
**Command:** `[actual command run]`
**Exit code:** [0 or error code]
**Status:** [PASS / FAIL]

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
**Command:** `[actual command run]`
**Exit code:** [0 or error code]
**Status:** [PASS / FAIL]

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
**Command:** `[actual command run]`
**Exit code:** [0 or error code]
**Status:** [PASS / FAIL]

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
**Command:** `[actual command run]`
**Exit code:** [0 or error code]
**Status:** [PASS / FAIL]
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
**Clean command:** `[Use the domain profile POSIX or PowerShell clean command variant]`
**Exit code:** [0 or error code]
**Status:** [PASS / FAIL]
**Tests passed:** [X/Y]

<details>
<summary>Output</summary>

```
[Paste full output from clean build + test]
```

</details>

**Notes:**

---

## Self-Review (Full projects only)

### Domain Checklist Results
[Run every Automated Check from the domain profile. Paste command + result.]

| Check | Command | Result | Pass? |
|-------|---------|--------|-------|
| [e.g., No attr binding] | `[command]` | [what was found] | [YES/NO] |

### Devil's Advocate
1. **What happens when:** [scenario 1], [scenario 2], [scenario 3]
2. **The weakest link is:** [identification and reasoning]
3. **If I had to break this, I would:** [attack vector]

### Findings
[Minimum 3 genuine findings for Full projects. If fewer, document what you checked.]

| # | Severity | Finding | Impact |
|---|----------|---------|--------|
| 1 | [P0/P1/P2] | [Description] | [What it affects] |

---

## Failure History

*(Record any gate failures and their resolutions here)*

### [Date] Gate [N] FAILED
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
