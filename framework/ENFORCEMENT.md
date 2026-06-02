# Enforcement

This file does **not** add new rules. It restates a subset of the obligations already defined in [`BUILDER.md`](BUILDER.md), [`GATEKEEPER.md`](GATEKEEPER.md), [`REVIEWER.md`](REVIEWER.md), and the verification-log template **in machine-checkable form**, and describes how a host can bind those checks so they fire regardless of whether the agent cooperates.

## Why this layer exists

Every other rule in the framework relies on the agent *choosing* to obey markdown. An impeccably documented run and a skipped one are indistinguishable from the outside — the framework's deepest structural weakness. Two reference points solve the same problem mechanically:

- **A host with hooks** (e.g., Claude Code) runs a check automatically after an edit or before declaring "done."
- **An orchestrator with a pipeline + CI gate** (e.g., Karajan) makes steps you cannot skip and fails the build when an invariant breaks.

Agent Kit stays tool-agnostic by separating the two halves: it defines **what** must hold (the invariants below), and leaves **how** to enforce them to the host (the binding model). The framework remains fully usable with zero automation — the manual tier is always available — but where a host offers hooks or CI, these invariants are what you wire to them.

## Invariants

Each invariant points at the rule it enforces. None is new. "Mechanical" means a script can decide it by reading the artifacts; "Human" means it needs judgment a check can only flag, not settle.

| ID | Asserts | Source rule | Check |
|----|---------|-------------|-------|
| **INV-1** | Required artifacts for the declared size exist on disk before any later phase is marked complete | `BUILDER.md → Artifact Existence Gate` | Mechanical |
| **INV-2** | The verification log has a `Progress` section | `VERIFICATION_LOG-template.md → Progress` | Mechanical |
| **INV-3** | Every gate marked `PASS`/`FAIL`/`BLOCKED` in `Progress` has a matching gate entry with recorded output — no "assumed to pass" | `VERIFICATION_LOG-template.md → Gate entry correspondence rule`; `GATEKEEPER.md §3` | Mechanical |
| **INV-4** | Every claim marked `Verified` carries a `Source:` that resolves to an existing file, gate row, or captured output | `BUILDER.md → Evidence States` | Mechanical (resolvability) |
| **INV-5** | No `Verified` claim depends on a `Blocked` gate | `BUILDER.md → Hard rule: blocked-gate dependency` | Mechanical / Human |
| **INV-6** | No `Blocking` review finding is left undispositioned (or `Fix now` and unfixed) when work is declared complete | `REVIEWER.md → Completion rule` | Mechanical / Human |

**Non-goal.** Enforcement checks the *presence and resolvability of evidence*, not its *meaning*. It cannot judge whether a test is meaningful or a Source actually proves the claim — that is what GateKeeper (execution) and Reviewer (logic) are for. INV checks raise the floor; they do not replace the roles.

## Binding model

Pick the strongest tier the host supports. Each tier enforces the same invariants; they differ only in how mechanically they fire.

### Tier 1 — Automated (hook or CI) — strongest

The host runs the checks without the agent's involvement. This is the hooks / pipeline-CI equivalent.

A minimal portable check for INV-1/2/3 — no dependencies, reads the log markdown. Treat it as a **reference example to adapt**, not a shipped tool:

```sh
# fails (exit 1) if a verification log is missing a Progress section,
# or marks a gate PASS/FAIL/BLOCKED with no matching gate entry.
for log in docs/*-verification.md; do
  [ -e "$log" ] || { echo "INV-1: no verification log"; exit 1; }
  grep -q '^## Progress' "$log" || { echo "INV-2: $log has no Progress section"; exit 1; }
  # INV-3: each gate marked in Progress must appear as a gate entry
  grep -Eo 'Gate [0-4][^|]*\| *(PASS|FAIL|BLOCKED)' "$log" | grep -Eo 'Gate [0-4]' | sort -u |
  while read -r g; do
    grep -q "## $g" "$log" || grep -q "| ${g##Gate } " "$log" || { echo "INV-3: $log marks $g without a gate entry"; exit 1; }
  done
done
```

Bind it to whatever the host exposes:

- **A hook host** (e.g., Claude Code `settings.json`): run the check on a `Stop`/pre-completion hook so a "done" claim is blocked while an invariant fails. Example shape:
  ```json
  { "hooks": { "Stop": [ { "hooks": [ { "type": "command", "command": "sh scripts/enforce.sh" } ] } ] } }
  ```
- **CI** (e.g., GitHub Actions): run the same check as a required job so a PR that breaks an invariant fails the gate — the Karajan-CI equivalent.
  ```yaml
  - name: Agent Kit invariants
    run: sh scripts/enforce.sh
  ```

INV-4/5/6 are partially mechanical (a check can flag a `Verified` row whose `Source:` path does not exist, or a `Blocking` finding with no disposition) and partially human (whether the Source truly proves the claim). Automate the resolvability flag; leave the judgment to Reviewer/GateKeeper.

### Tier 2 — Pre-commit (git)

Same checks at commit time via `.git/hooks/pre-commit` (or any git-hook manager). Catches a broken invariant before it enters history. Weaker than CI (a commit can bypass with `--no-verify`) but needs no host features.

### Tier 3 — Manual (always available, zero-install)

When the host offers no hooks and no CI, the invariants are a **pre-completion checklist** the agent runs and records before claiming the work is complete:

- [ ] INV-1 — required artifacts for this size exist (`BUILDER.md → Artifact Existence Gate`)
- [ ] INV-2 — verification log has a `Progress` section
- [ ] INV-3 — every gate status in `Progress` has a matching gate entry with real output
- [ ] INV-4 — every `Verified` claim's `Source:` resolves
- [ ] INV-5 — no `Verified` claim depends on a `Blocked` gate
- [ ] INV-6 — no `Blocking` review finding is undispositioned or unfixed

This tier is the floor that keeps the framework agnostic and zero-install: the rules still bind, just by discipline rather than by machine. Tiers 1 and 2 are how you stop depending on that discipline where the host allows.
