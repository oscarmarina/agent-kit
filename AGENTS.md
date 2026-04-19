# Agent Instructions

**Activation:** Before writing any code, determine project size (Quick / Standard / Full) and state it explicitly. This is the activation signal — if you skip it, the framework is not active.

## Reading order

1. **Read `framework/BUILDER.md`** — the process contract for design and implementation. It defines Mode Detection (single-agent vs single-agent-with-delegation), the domain-profile selection algorithm, and when to load profiles relative to the process steps. Do not pre-select a profile by filename guess — the selection algorithm is deterministic and lives in `BUILDER.md` step 2.
2. **Read `framework/GATEKEEPER.md`** — the verification standard for gates (including retry budget).
3. **Check `framework/domains/`** for candidate profiles only when BUILDER step 2 tells you to — it will route you to a standalone profile, a profile link, or instruct you to create one.

**Single-agent environments** (CLI agents, IDE assistants): You fulfill both Builder and GateKeeper roles. `BUILDER.md` defines both dual-agent and single-agent verification protocols. The `GATEKEEPER.md` contract still applies as your verification standard, but you execute gates directly instead of handing off.

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

Project artifacts (intent, design, verification) go in `docs/`.
