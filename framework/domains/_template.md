# Domain Profile: [Project Name]

## Profile Link

```yaml
extends: [profile-id]           # Profile ID from catalog/ (e.g., apps-sdk-mcp-lit-vite)
catalog_version: 1.0.0          # Must match the Profile Version field in the catalog base profile
```

> **How it works:** The Builder loads the base profile from `catalog/[extends].md` and applies the local sections below on top. Base sections not overridden here remain active.
>
> **Version check:** When loading this link, compare `catalog_version` above against the `Profile Version` field in `catalog/[extends].md`:
> - **Same version** → base profile is current, proceed normally.
> - **Patch difference** (e.g., `1.0.0` vs `1.0.1`) → new pitfalls or adversary questions were added to the base. Read the additions before proceeding; update `catalog_version` here after re-verifying.
> - **Minor or major difference** (e.g., `1.0.0` vs `1.1.0` or `2.0.0`) → verification commands or integration rules changed. Treat inherited sections as potentially stale. Re-read the full base profile and update Local Overrides if needed before writing any code. Update `catalog_version` after reconciling.

---

## Local Pitfalls

Project-specific pitfalls discovered during implementation. These do NOT exist in the base profile — they are unique to this project's context.

### Pitfall L1: [Name]
- **What goes wrong:** [Description]
- **Correct approach:** [How to do it right]
- **Detection:** [Search pattern or verification step]

*(Add as many as needed. Each one is a candidate for contributing back to the catalog profile.)*

## Local Adversary Questions

Project-specific questions that the base profile's adversary questions don't cover.

- [e.g., "What happens if our auth token expires mid-widget-render?"]

## Local Overrides

Sections here **replace** the corresponding section in the base profile. Only override what differs for this project — omit sections where the base is correct.

Overrides should describe real project-level differences, not temporary runner workarounds. If a command had to change only because of a specific agent shell, sandbox, or execution harness, keep that in the verification log instead of encoding it here.

### Verification Commands (overrides)

Only include gates that differ from the base profile.

**GATE 3 (Tests):**
- Command: `[e.g., npm test -- --project=widget]`
- Expected output: [what success looks like]

### Integration Rules (overrides)

[Only if this project has integration patterns that differ from the base profile]

## Local Decision History

Decisions specific to THIS project (not stack-wide). Stack-wide decisions belong in the catalog profile.

| Date | Decision | Context | Constraint |
|------|----------|---------|------------|
| [e.g., 2026-03-20] | [e.g., Skip server tests in CI] | [Why] | [MUST/MUST NOT] |
