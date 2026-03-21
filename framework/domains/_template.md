# Domain Profile: [Project Name]

## Profile Link

```yaml
extends: [profile-id]           # Profile ID from catalog/ (e.g., apps-sdk-mcp-lit-vite)
catalog_version: 1.0.0          # Version of the catalog profile when this link was created
```

> **How it works:** The Builder loads the base profile from `catalog/[extends].md` and applies the local sections below on top. Base sections not overridden here remain active. To check if your base is outdated, compare `catalog_version` against the current profile.

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
