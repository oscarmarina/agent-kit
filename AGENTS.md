# Agent Instructions

## Reading order

1. **Check `framework/domains/` for an existing domain profile** matching the task's stack. If exactly one profile matches by filename or obvious keyword overlap, read it first — its pitfalls, adversary questions, and verification commands are the highest-value-per-token knowledge available. If multiple candidates exist, defer selection to `BUILDER.md` step 2 (which defines the full scoring algorithm). If none exists, continue to step 2.
2. **Read `framework/BUILDER.md`** — the process contract for design and implementation.
3. **Read `framework/GATEKEEPER.md`** — the verification standard for gates.

**Single-agent environments** (CLI agents, IDE assistants): You fulfill both Builder and GateKeeper roles. `BUILDER.md` defines both dual-agent and single-agent verification protocols. The `GATEKEEPER.md` contract still applies as your verification standard, but you execute gates directly instead of handing off.

Project artifacts (intent, design, verification) go in `docs/`.
