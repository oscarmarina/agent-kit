# Agent Kit

A process framework for LLM-assisted software development. Works with any LLM and any technology stack.

## The problem

LLMs make the same mistakes across projects. They skip verification, forget past lessons, lose context between sessions, and build confidently on wrong assumptions. Each new conversation starts from zero — even when the same stack was used yesterday.

Agent Kit solves this with an **Adversarial Verification Loop**:

1. **Adversarial Verification**: A `Builder` (writes code) and `GateKeeper` (executes tests) with strict separation of concerns. In dual-agent environments, these are separate agents. In single-agent environments (CLI agents, IDE assistants), the same agent fulfills both roles but maintains the same mechanical discipline — real commands, real output, recorded in the verification log. The principle is proof over claims, regardless of how many agents execute the work.
2. **Domain profiles** — living documents that accumulate stack-specific knowledge across projects. Every bug fix, every gate failure, every discovery enriches the profile. The next project on the same stack starts knowing what the last one learned.

## How the process works

```mermaid
graph TB
    %% ── Entry & Reading Order ──
    AGENTS["AGENTS.md<br/><i>entry point</i>"]
    HAS_PROFILE{"Domain profile<br/>exists?"}
    AGENTS --> HAS_PROFILE
    HAS_PROFILE -->|"Yes"| READ_PROFILE["Read domain profile first<br/><i>highest value per token</i>"]
    HAS_PROFILE -->|"No"| BUILDER
    READ_PROFILE --> BUILDER["BUILDER.md<br/><i>process contract</i>"]

    %% ── Sizing ──
    BUILDER --> SIZE{Determine size}
    SIZE -->|"< 3 files,<br/>clear scope"| QUICK["<b>Quick</b><br/>Code → Gate 2 minimum"]
    SIZE -->|"feature-sized"| STANDARD
    SIZE -->|"new project /<br/>major refactor"| FULL["<b>Full</b><br/>Standard + ADRs +<br/>Devil's Advocate"]

    %% ── Standard flow (steps match BUILDER.md) ──
    subgraph STANDARD ["<b>Standard / Full Process</b>"]
        direction TB
        INTENT["1  Capture Intent<br/><i>docs/[project]-intent.md</i>"]
        LOAD_PROFILE["2  Load domain profile"]
        LOAD_SKILLS["3  Load relevant skills<br/><i>.github/skills/ · .agents/skills/</i>"]
        CHECKPOINT["4  Pre-code checkpoint<br/><i>deps? assumptions? pitfalls? size?</i>"]
        DESIGN["5  Design<br/><i>optional Standard · mandatory Full</i>"]
        WRITE_CODE["6  Gated build<br/><i>Gate 0 → 1 → 2</i>"]
        HANDOFF["7  Tests + verification<br/><i>Gate 3 → Gate 4</i>"]

        INTENT --> LOAD_PROFILE
        LOAD_PROFILE --> LOAD_SKILLS
        LOAD_SKILLS --> CHECKPOINT
        CHECKPOINT --> DESIGN
        DESIGN --> WRITE_CODE
        WRITE_CODE --> HANDOFF
    end

    %% ── Verification Loop ──
    subgraph VERIFICATION ["<b>Verification Authority</b><br/><i>dual-agent: GateKeeper · single-agent: self-verify</i>"]
        direction TB
        GATES["Run gate commands"]
        TESTS["Record real output"]
        GATES --> TESTS
    end

    HANDOFF --> VERIFICATION

    %% ── Domain profile (central) ──
    DP[("<b>Domain Profile</b><br/><i>pitfalls · adversary questions<br/>integration rules · checks<br/>decision history</i>")]

    %% ── Connections to/from profile ──
    LOAD_PROFILE -. "load" .-> DP
    DP -. "pitfalls &<br/>adversary Qs" .-> CHECKPOINT

    %% ── Gate failure loop ──
    GATES --> FAIL{"Gate<br/>fails?"}
    FAIL -->|"Yes"| FIX["Raw log → root cause<br/>→ fix → re-run"]
    FIX --> UPDATE_PROFILE["Update profile<br/><i>new pitfall / rule</i>"]
    UPDATE_PROFILE --> DP
    FIX --> GATES
    FAIL -->|"No"| TESTS

    %% ── Learning cycle ──
    TESTS --> REVIEW["8  Self-review<br/><i>Adversary Lens + domain checklist</i>"]
    REVIEW --> LEARNING["9  Domain learning<br/><i>verify profile was updated</i>"]
    LEARNING --> DP

    %% ── Verification log ──
    VLOG["docs/[project]-verification.md<br/><i>Progress · Gates · Failures<br/>Domain Profile Updates</i>"]
    GATES -. "real output" .-> VLOG
    TESTS -. "real output" .-> VLOG
    FIX -. "failure history" .-> VLOG

    %% ── Resume ──
    RESUME(["Resume interrupted work"])
    RESUME -. "read Progress section" .-> VLOG
    VLOG -. "continue from<br/>last completed step" .-> GATES

    %% ── Quick also feeds profile ──
    QUICK --> QUICK_GATE["Gate 2 minimum"]
    QUICK_GATE --> ESCALATE{"Touches > 3 files<br/>or new bugs?"}
    ESCALATE -->|"Yes"| INTENT
    ESCALATE -->|"No"| QUICK_DONE["Done"]
    QUICK_GATE -. "if bug found" .-> DP

    %% ── Next project ──
    DP -. "<b>next project</b><br/>starts with all<br/>accumulated knowledge" .-> LOAD_PROFILE

    %% ── Styles ──
    style DP fill:#4CAF50,color:#fff,stroke:#2E7D32,stroke-width:2px
    style VLOG fill:#2196F3,color:#fff,stroke:#1565C0,stroke-width:2px
    style REVIEW fill:#FF9800,color:#fff,stroke:#E65100,stroke-width:2px
    style CHECKPOINT fill:#FF9800,color:#fff,stroke:#E65100,stroke-width:2px
    style FAIL fill:#f44336,color:#fff,stroke:#c62828
    style FIX fill:#f44336,color:#fff,stroke:#c62828
    style UPDATE_PROFILE fill:#4CAF50,color:#fff,stroke:#2E7D32
    style LOAD_SKILLS fill:#CE93D8,color:#000,stroke:#8E24AA,stroke-width:1px
    style HAS_PROFILE fill:#4CAF50,color:#fff,stroke:#2E7D32
    style READ_PROFILE fill:#4CAF50,color:#fff,stroke:#2E7D32
    style RESUME fill:#9C27B0,color:#fff,stroke:#6A1B9A
    style QUICK fill:#78909C,color:#fff,stroke:#37474F
    style QUICK_DONE fill:#78909C,color:#fff,stroke:#37474F
    style ESCALATE fill:#FF9800,color:#fff,stroke:#E65100
```

**Reading the diagram:**
- **Green** = Domain Profile — the learning mechanism. Knowledge flows in (from failures) and out (to future projects). The entry flow checks for an existing profile before loading BUILDER.md.
- **Blue** = Verification Log — the proof mechanism. Real command output, not assumptions.
- **Orange** = Checkpoints — points where the LLM must pause and think before acting. The pre-code checkpoint (step 4) and self-review are explicit pause points.
- **Red** = Failure path — failures are captured, root-caused, and fed back into the profile.
- **Light purple** = Skills — optional guidance loaded before design (aesthetic, conventions, workflow).
- **Purple** = Resume — interrupted sessions recover from the verification log's Progress section.
- Steps 1-9 in the Standard/Full subgraph match the step numbers in BUILDER.md. Verification runs in dual-agent mode (separate GateKeeper) or single-agent mode (self-verify with same discipline).
- The dashed line from Domain Profile back to "Load domain profile" is the **learning cycle**: every project starts with the accumulated knowledge of all previous projects on that stack.

## The four lenses

The LLM doesn't follow a linear sequence. It applies four thinking modes whenever they're relevant:

**User Lens** — What does the software need to do? Extracts goals, implicit needs, and explicit rejections from the prompt. This produces the Intent document — the anchor everything traces back to.

**Architecture Lens** — How should it be built? Component structure, data flow, dependency tree, initialization chain. Every choice needs a reason and rejected alternatives. The LLM reads the public API of every dependency before implementing — if a library already provides the functionality, it uses it.

**Adversary Lens** — What could go wrong? Applied *during* design, not only after. "What input breaks this?", "What happens when X is unavailable?", "What would a careless developer get wrong?" If the domain profile has adversary questions, they must be answered against the specific design before any code is written.

**Domain Lens** — What does the domain profile say? Incorporates integration rules and semantic terminology. The accumulated knowledge of previous projects on this stack.

## Domain profiles: the learning mechanism

Domain profiles are the most valuable artifact in the framework. They are its memory.

A domain profile captures everything an LLM needs to know about a specific technology stack: what commands to run, what mistakes to avoid, what questions to ask before coding, what to check during review. Each section exists because a real project needed it.

### The flywheel effect

The first project on a new stack creates a minimal profile — terminology, verification commands, a couple of pitfalls. Then something breaks. The LLM root-causes the failure, fixes it, and adds the lesson to the profile. By the end of the first project, the profile has grown.

The second project on the same stack loads that profile. The failures from project 1 are now prevented. New discoveries from project 2 further enrich it. And so on.

From real usage: a domain profile started with 3 pitfalls. After two projects, it had 11 pitfalls, 7 adversary questions, and 8 decision history entries — all learned from real bugs. The second project had significantly fewer gate failures because the profile already knew what to watch for.

### What makes them different from documentation

Documentation describes how things work. Domain profiles describe how things *fail* — and what to do about it. They are written by the LLM during implementation, not by a human before it. They grow from experience, not from planning.

They also travel. Copy `framework/` along with the relevant base profiles from `catalog/` to a new repository and every project in that repo inherits the knowledge. Different LLMs can use the same profile. The learning persists regardless of which model or session created it.

## Verification: proof over claims

LLMs are confident. They will tell you "everything works" when it doesn't. The framework addresses this with an adversarial loop — mandatory checkpoints where the GateKeeper runs a real command against the Builder's code, pastes the real output, and records it in the verification log.

"Assumed to pass" is never valid evidence. If the output isn't in the log, it didn't happen.

When a gate fails, the failure is classified as Product (the code is wrong), Environment (the tool or runner blocked execution), or Process (the gate definition itself was wrong). This distinction makes the verification log more honest and actionable — an environment failure doesn't mean the code is broken, and a process failure means the framework needs adjustment, not the implementation.

Gates also create a recovery mechanism. Each verification log has a Progress section at the top. When a session is interrupted — context limit, network issue, next morning — the new session reads Progress and continues from the last completed step. No rework, no guessing.

## Why artifacts matter

The framework produces three documents per project:

- **Intent** — captures *what* and *why*. The anchor for scope. Always produced.
- **Design** — captures *how*. Architecture, decisions, risks, and how domain pitfalls are addressed. Required for Full; optional for Standard (where key decisions live in the Intent).
- **Verification Log** — captures *proof*. Real gate output, failure history, and progress state. Always produced.

These aren't bureaucracy. They exist because LLMs lose context. The Intent prevents scope creep. The Design prevents the LLM from re-deciding things it already decided. The Verification Log prevents it from re-running gates that already passed. Together, they make the process resilient to the reality of working with LLMs: limited context windows, interrupted sessions, and confident hallucinations.

## Execution models

The framework supports **single-agent** (one LLM, both roles) and **dual-agent** (separate Builder and GateKeeper). In orchestrated environments with sub-agents, the orchestrator owns all artifacts and process state; Builder and GateKeeper become delegated roles. GateKeeper maps cleanly to a dedicated sub-agent (defined inputs, structured output, no code edits). Builder works better as a role the orchestrator executes or delegates per phase, not as a permanent sub-agent.

## LLM compatibility

The framework is LLM-agnostic. Any model that can read markdown and follow instructions can use it.

## Documentation

Each document serves a distinct purpose ([Diátaxis](https://diataxis.fr/)):

| Document | Type | Audience | Purpose |
|----------|------|----------|---------|
| [**README.md**](README.md) (this file) | Explanation | Human | Why the framework exists and how it works conceptually |
| [**GUIDE.md**](GUIDE.md) | Tutorial | Human | Step-by-step first project walkthrough |
| [**framework/README.md**](framework/README.md) | Reference | Human | File descriptions, gate definitions, artifact specs, contracts |
| [**framework/BUILDER.md**](framework/BUILDER.md) | Agent contract | LLM | Process instructions the LLM follows during implementation |
| [**framework/GATEKEEPER.md**](framework/GATEKEEPER.md) | Agent contract | LLM | Verification instructions the LLM follows during gate execution |
| [**catalog/README.md**](catalog/README.md) | How-to guide | Human | How to use and contribute domain profiles |

**Start here:** If you're new, read [GUIDE.md](GUIDE.md). If you need specifics, read [framework/README.md](framework/README.md). If you want to understand the design decisions, read this file.

**Domain profiles:** The [`catalog/`](catalog/) directory contains community-contributed domain profiles built from real projects:

- [`apps-sdk-mcp-lit-vite.md`](catalog/apps-sdk-mcp-lit-vite.md) — MCP Apps + Lit + Vite. 11 pitfalls, 7 adversary questions. Shows what a mature profile looks like after the flywheel has turned.

If a relevant catalog profile exists for your stack, create a profile link in `framework/domains/` that extends it (see `framework/domains/_template.md`). If not, the Builder will create a standalone profile during the first project using `framework/templates/DOMAIN_PROFILE-template.md`.

## License

MIT
