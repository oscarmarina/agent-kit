# Sub-agent Prompt Template: [Role Name]

**Artifact Schema Version:** 1.1.0

> This is a **prompt template**, not a runtime contract. Current LLM tooling (Claude Code, Cursor, and similar) does not register persistent sub-agent roles — every sub-agent invocation is ephemeral and starts with no memory of prior calls. Use this template to author the *prompt body* you pass each time you spawn a worker for this role. Stored here so repeated delegations stay consistent instead of diverging.
>
> Fill the sections below once, then paste them (or reference them) as the sub-agent's prompt. The orchestrator still owns all framework artifacts (Intent, Design, Verification Log, Domain Profile) and records every result.

## Role

[One sentence: what this worker is, in imperative form. Example: "You are a GateKeeper sub-agent. Run one verification gate and report real output."]

## Use When

- [Situation 1 where this worker is the right tool]
- [Situation 2 where isolated context or specialization is valuable]

## Do Not Use When

- [Situation where an orchestrator step is better]
- [Situation where a skill or domain profile is the correct mechanism]

## Inputs (paste at invocation)

- [Required input 1 — e.g., gate level, file paths, specific question]
- [Required input 2 — e.g., the exact command, relevant domain-profile excerpt]
- [Required context excerpt — paste it; do not tell the sub-agent to "read AGENTS.md"]

## Allowed Actions

- [Read files / run searches / execute one specific command / propose a patch]
- [List only actions this worker may take]

## Forbidden Actions

- [What this worker must never do]
- Never edit the Intent, Design, Verification Log, or Domain Profile directly
- Never make final pass/fail or completion claims
- [Role-specific forbiddens, e.g., "never modify implementation code" for gatekeeper]

## Required Output Shape

- **Format:** [e.g., exit code + raw stdout/stderr + classification; findings list; patch diff]
- **Success signal:** [What a passing result looks like]
- **Failure signal:** [How it reports blocked, failed, or ambiguous work]

## Escalate Back To Orchestrator When

- [Condition 1: conflicting requirements]
- [Condition 2: scope expansion beyond the inputs]
- [Condition 3: evidence is ambiguous or authority is unclear]

## Example Invocation Body

```text
[Paste the full prompt body as it would be passed to the Agent/Task tool. Self-contained — include role, inputs, context excerpts, task, and expected output shape. The orchestrator adds no additional context at invocation time.]
```

## Notes for the Orchestrator

- After the worker returns, **you** translate results into framework state (verification log row, design note, profile entry). The worker does not own these.
- Keep this template up to date as you learn what context the worker consistently needs — each time you have to edit the pasted prompt at invocation, update this template so the next call starts closer to correct.
