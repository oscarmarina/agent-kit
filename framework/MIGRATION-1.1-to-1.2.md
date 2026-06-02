# Artifact Schema Migration: 1.1.0 → 1.2.0

This guide covers the artifact-contract change from `ARTIFACT_SCHEMA_VERSION` **1.1.0** to **1.2.0**. It is required reading only if you maintain verification logs created under 1.1.0; new artifacts created from the current templates already follow 1.2.0.

## Why the bump

The framework added the **Reviewer** role (`REVIEWER.md`) and the **Sharded Execution** breadth-axis discipline (`BUILDER.md`). Both write their output into the existing Verification Log rather than into new artifact files. That changed the shape of the `VERIFICATION_LOG` template, so the artifact contract moved from 1.1.0 to 1.2.0. Per `framework/README.md` → Versioning, a shape change to any artifact template requires an `ARTIFACT_SCHEMA_VERSION` bump and this migration note — otherwise two structurally different logs would carry the same schema id and an audit could not tell them apart.

## What changed

**Only `templates/VERIFICATION_LOG-template.md` changed shape.** Two **additive, optional** sections were introduced:

1. **`### Independent Review`** — a subsection under `## Self-Review`. Records the independent reviewer's findings (Severity, Evidence State, Disposition). Per `BUILDER.md → Independent Review Protocol`: optional for Quick, a blocking-findings pass for Standard, **mandatory for Full**.
2. **`## Shard Manifest (broad multi-file tasks only)`** — a section after `## Progress`. Records the partition contract for `Sharded Execution`. Present only for broad, repetitive multi-file tasks; omit it otherwise.

`templates/INTENT.md`, `templates/DESIGN.md`, `templates/DOMAIN_PROFILE-template.md`, and `templates/SUBAGENT_PROMPT_TEMPLATE.md` are **structurally unchanged**. Their stamped `Artifact Schema Version` advances to `1.2.0` so new artifacts declare the current schema, but no field was added or removed.

## Backward compatibility

- **1.1.0 artifacts remain readable** and are valid evidence of the work they recorded. Do **not** rewrite completed logs — historical logs stay under the schema they were created with (consistent with "completed logs remain as historical evidence").
- The only real gap a reader should watch for: under 1.2.0 a **Full-project** verification log is expected to contain an `Independent Review` subsection. A 1.1.0 Full log predates that expectation; treat its absence as "review not yet run under the new contract," not as a violation of 1.1.0.

## Migrating a log you are continuing under 1.2.0

If you resume an active 1.1.0 verification log and keep working under 1.2.0:

1. Update the header line `**Artifact Schema Version:** 1.1.0` → `1.2.0`.
2. If the project is **Full** (or you run an independent review for any size), add the `### Independent Review` subsection under `## Self-Review`. Copy the table shape from the current `VERIFICATION_LOG-template.md`.
3. Add `## Shard Manifest` **only** if the remaining work uses Sharded Execution; otherwise leave it out.

No changes are required to existing Intent, Design, or domain-profile artifacts beyond stamping `1.2.0` on anything newly created.

## One-line summary

`1.1.0 → 1.2.0`: the Verification Log gained two optional sections (`Independent Review`, `Shard Manifest`); all other templates are unchanged. Old logs stay valid; only continued Full-project logs should add the `Independent Review` subsection.
