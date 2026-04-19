# Domain Profile: [Project Name]

## Profile Link

```yaml
extends: [profile-id]           # Profile ID from catalog/ (e.g., apps-sdk-mcp-lit-vite)
catalog_version: 1.0.0          # Version of the catalog profile when this link was created
artifact_schema_version: 1.1.0  # Version of the profile-link artifact schema
```

> **How it works:** The Builder loads the base profile from `catalog/[extends].md` and applies the local sections below on top. Base sections not overridden here remain active. To check if your base is outdated, compare `catalog_version` against the current profile.

---

## Local Pitfalls

Project-specific pitfalls discovered during implementation. These do NOT exist in the base profile — they are unique to this project's context.

### Pitfall L1: [Name]
- **Severity:** [critical / major / minor]
- **Occurrence count:** [number — starts at 1 when first documented]
- **Confidence:** [confirmed / inferred / heuristic]
- **Source:** [e.g., `docs/[project]-verification.md → Gate 3 FAILED 2026-04-16` | `Design review YYYY-MM-DD`]
- **What goes wrong:** [Description]
- **Correct approach:** [How to do it right]
- **Detection:** [Search pattern or verification step]

*(Add as many as needed. Use the same pitfall metadata as standalone/base profiles so Integrity Audit, promotion checks, and traceability work on profile links too.)*

## Local Adversary Questions

Project-specific questions that the base profile's adversary questions don't cover.

- [e.g., "What happens if our auth token expires mid-widget-render?"]

## Local Overrides

Sections here **replace** the corresponding section in the base profile. Only override what differs for this project — omit sections where the base is correct.

Overrides should describe real project-level differences, not temporary runner workarounds. If a command had to change only because of a specific agent shell, sandbox, or execution harness, keep that in the verification log instead of encoding it here.

### Verification Commands (overrides)

Only include gates that differ from the base profile.

Commands are written relative to the **project code root** unless the command text explicitly says otherwise.

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
