# Agent Instructions

**Normative surface:** `framework/BUILDER.md`, `framework/GATEKEEPER.md`, and `framework/REVIEWER.md` are the operational sources of truth — the Builder, GateKeeper, and Reviewer role contracts. `framework/ENFORCEMENT.md` adds no new rules; it restates a subset of those obligations in machine-checkable form. All other docs (`README.md`, `GUIDE.md`, `framework/README.md`, `.github/copilot-instructions.md`, `catalog/README.md`) are derived views for humans. If they disagree with the role contracts, those files win.

**Activation:** Before writing any code, determine project size (Quick / Standard / Full) and state it explicitly. This is the activation signal — if you skip it, the framework is not active.

## Reading order

1. **Read `framework/BUILDER.md`** — the process contract for design and implementation. It defines Mode Detection (single-agent vs single-agent-with-delegation), the domain-profile selection algorithm, and when to load profiles relative to the process steps. Do not pre-select a profile by filename guess — the selection algorithm is deterministic and lives in `BUILDER.md` step 2.
2. **Read `framework/GATEKEEPER.md`** — the verification standard for gates (including retry budget).
3. **Read `framework/REVIEWER.md`** — the independent-review role contract: who reads the diff's logic (as opposed to running commands), how independence scales by execution mode, and how findings are arbitrated so only blocking issues gate completion.
4. **Skim `framework/ENFORCEMENT.md`** — the machine-checkable invariants and how to bind them to your host (hook / CI / git pre-commit / manual). Optional to read up front; required when you wire enforcement.
5. **Check `framework/domains/`** for candidate profiles only when BUILDER step 2 tells you to — it will route you to a standalone profile, a profile link, or instruct you to create one.

**Single-agent environments** (CLI agents, IDE assistants): You fulfill the Builder, GateKeeper, and Reviewer roles yourself. `BUILDER.md` defines both dual-agent and single-agent verification protocols. The `GATEKEEPER.md` contract still applies as your verification standard, but you execute gates directly instead of handing off. The `REVIEWER.md` independence is procedural here (review the diff under the "someone else wrote this" stance) — see its mode rules.

**Multi-agent environments** (Claude Code with the Agent tool, orchestrated systems): If you have access to a tool that spawns sub-agents, you may delegate bounded work. When you do:

1. **You remain the owner** of the Intent, Design, Verification Log, domain profile updates, and all phase decisions. Sub-agents return results; you record them.
2. **Pass context explicitly** in the sub-agent prompt — never assume the sub-agent has access to this session. Include the relevant domain profile pitfalls, the specific design section, file paths, and the exact task. A sub-agent prompt that starts with "read AGENTS.md" is wrong — it will load the full framework and behave as an orchestrator, not as a scoped worker.
3. **Receive and translate the result** — after the sub-agent returns, you decide what becomes durable framework state (profile entry, verification log row, design note) and write it yourself.
4. **Patterns and anti-patterns**: see `framework/README.md` → Sub-agents and Skills.

Use the `Agent` tool to spawn sub-agents. Set `description` to the sub-agent role name (e.g., `"gatekeeper-subagent"`, `"research-subagent"`). Write the `prompt` as a self-contained briefing: role, context excerpt, task, expected output shape. Example structure for a GateKeeper sub-agent:

```
You are a GateKeeper sub-agent. Your only job is to run verification commands and report real output.

Context:
- Project: [name]
- Gate: 2 (Feature)
- Domain profile gate command: [command from profile]

Run the command. Return: exit code, raw stdout/stderr, and failure classification (Product / Environment / Process). Do not edit any files.
```

The same pattern is the cleanest way to get an **independent** review: spawn a Reviewer sub-agent, give it only the diff, the Intent, and the active profile's pitfalls, and ask for findings with a Severity (`Blocking` / `Non-blocking`) and Disposition (`Fix now` / `Defer` / `Reject as noise`). You own the dispositions it returns. See `framework/REVIEWER.md`.

Project artifacts (intent, design, verification) go in `docs/`.
