# Domain Profile Catalog

Community-contributed domain profiles for Agent Kit. Each profile captures stack-specific knowledge — pitfalls, adversary questions, verification commands, and decision history — learned from real projects.

## Available Profiles

| Profile | Version | Stack | Pitfalls | Adversary Qs |
|---------|---------|-------|----------|--------------|
| [apps-sdk-mcp-lit-vite](apps-sdk-mcp-lit-vite.md) | 1.0.0 | MCP Apps + Lit + Vite + TypeScript | 23 | 17 |

## Using a profile

Create a profile link in your project's `framework/domains/` directory that extends the catalog profile:

```markdown
<!-- framework/domains/my-project.md -->

## Profile Link

extends: apps-sdk-mcp-lit-vite
catalog_version: 1.0.0

## Local Pitfalls
<!-- Project-specific pitfalls go here -->

## Local Overrides
<!-- Only sections that differ from the base -->
```

The Builder loads the base from `catalog/` and merges your local additions on top. See `framework/domains/_template.md` for the full template and `framework/domains/README.md` for merge rules.

## Promotion Protocol

Pitfalls start their life in a project's local profile link (`framework/domains/`). They graduate to the catalog base profile when they demonstrate they are stack-wide traps — not project-specific quirks. This is how the catalog grows automatically from real usage.

### Promotion triggers

A pitfall becomes a **catalog candidate** when it meets the baseline requirement **and** one of the trigger criteria:

**Baseline (required for any promotion):**

| Requirement | Rationale |
|---|---|
| `Confidence: confirmed` with a `Source:` pointing to a verification-log failure | Only runtime-proven failures graduate to the catalog. `inferred` and `heuristic` pitfalls — including those sourced from prompts, design reviews, or preventive knowledge — are not promotion-eligible, regardless of severity or count. |

**Trigger (at least one, on top of baseline):**

| Criterion | Threshold | Rationale |
|-----------|-----------|-----------|
| `occurrence_count` | `>= 3` across different projects | Three independent hits confirm the trap is stack-level, not a one-off |
| `severity: critical` + `Confidence: confirmed` | First runtime detection | Data loss, silent wrong behavior, or security failures warrant immediate candidacy — but only after a real gate proves the failure, not merely a prompt asserting it |

The GateKeeper flags `critical` pitfalls with `<!-- catalog candidate: critical severity -->` on detection (runtime only). The Builder evaluates all candidates during the Promotion Check (step 7 of `framework/BUILDER.md`), run right after Self-Review (step 6).

### Portability test (three questions)

Before promoting, answer all three:

1. **Stack-wide?** Would any project on this stack hit this failure, regardless of project-specific architecture or data shapes?
2. **Generic Detection?** Is the Detection command free of hardcoded project paths, project-specific filenames, or environment-specific assumptions? (e.g., `rg "pattern" src/` is generic; `rg "pattern" src/my-widget/` is not)
3. **General Correct approach?** Does the fix apply to any future project on this stack, not just this project's specific implementation?

If all three are yes → promote. If any is no → keep local, add a one-line note explaining why.

### How to promote

1. Copy the pitfall entry (including `Severity`, `Occurrence count`, `What goes wrong`, `Correct approach`, `Detection`) from the local profile link into the base profile in `catalog/[profile].md`.
2. In the local profile link, replace the full entry with a short reference note:
   ```
   ### Pitfall N: [Name] — promoted to catalog base
   <!-- Promoted [date]. See catalog/[profile].md -->
   ```
3. Record the promotion in the Verification Log's Domain Profile Updates section: `Promoted pitfall "[Name]" to catalog/[profile].md — occurrence_count: N, severity: X`.
4. Update the Available Profiles table at the top of this file to reflect the new pitfall count.

### Example

A project using the `apps-sdk-mcp-lit-vite` stack encounters a new pitfall with `severity: critical`. The GateKeeper adds it to the local profile with `<!-- catalog candidate: critical severity -->`. During Self-Review, the Builder runs the portability test: it is stack-wide, the Detection command uses only generic `src/` paths, and the fix applies to any MCP widget project. All three pass → the pitfall moves to `catalog/apps-sdk-mcp-lit-vite.md` and the local profile gets a reference note. The Verification Log records the promotion.

### Legacy traceability policy

Some early catalog entries may predate the current traceability standard. Those entries are allowed to remain in the catalog only if they are marked honestly:

- Use `Confidence: inferred`, not `confirmed`, when the underlying failure was real but the exact verification-log section was not preserved.
- Keep the `Source` field explicit about the legacy/pre-traceability origin.
- Upgrade an entry to `confirmed` only after a later project records the same failure with an exact verification-log reference.
- Update `Last Verified` whenever you normalize or re-audit a legacy profile.

## Contributing a profile

Domain profiles grow from real project experience. If you've built projects with a stack that isn't represented here, consider contributing your profile.

### Requirements

1. Use the template at `framework/templates/DOMAIN_PROFILE-template.md`
2. Follow the naming convention: `[domain]-[stack].md` (e.g., `web-react-nextjs.md`, `backend-python-fastapi.md`)
3. Include at minimum:
   - **Selection Metadata** — Profile ID, Match Keywords, Use When, Do Not Use When
   - **Terminology Mapping** — Stack-specific command translations
   - **Verification Commands** — Exact commands for Gates 0-4
   - **2+ Common Pitfalls** — With What/Correct/Detection pattern
4. Every pitfall and adversary question should come from a real bug or failure — not hypotheticals

### What makes a good profile

- **Specific:** "Vite uses `--k-` CSS variable prefix, not `--p-`" beats "check your CSS variables"
- **Actionable:** Each pitfall has a Detection command that can be run mechanically
- **Honest:** If a pitfall was discovered by making the mistake, say so
- **Growing:** A profile with 3 real pitfalls is more valuable than one with 10 speculative ones

### Submitting

1. Fork the repository
2. Add your profile to `catalog/`
3. Open a pull request with:
   - The stack and domain your profile covers
   - How many projects informed the profile
   - At least one example of a pitfall that prevented a real bug
