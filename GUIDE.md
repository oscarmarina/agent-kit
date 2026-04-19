# Tutorial: Your first project with Agent Kit

> **Non-normative view.** This tutorial walks humans through a first project. Operational truth — the rules agents must follow — lives in [`framework/BUILDER.md`](framework/BUILDER.md) and [`framework/GATEKEEPER.md`](framework/GATEKEEPER.md). If this page disagrees with either, those files win.

In this tutorial, we will set up Agent Kit in a new repository and use it to build a project with an LLM. By the end, you will have seen the full cycle: prompt, intent, design, gated build, and a domain profile that captures what the LLM learned.

## What we need

- A repository (new or existing) where you want to build something
- Access to an LLM that can read files (Claude Code, ChatGPT with file access, Cursor, Windsurf, or similar)
- A project idea and a technology stack in mind

## Step 1: Copy the framework into your repo

Copy the framework into your repository's root:

```bash
cp -R framework/ your-repo/framework/
cp AGENTS.md your-repo/
mkdir -p your-repo/docs

# Optional: if a catalog profile exists for your stack, copy it too
mkdir -p your-repo/catalog
cp catalog/[your-profile].md your-repo/catalog/
```

Your repo should now look like this:

```
your-repo/
├── AGENTS.md
├── framework/
│   ├── BUILDER.md
│   ├── GATEKEEPER.md
│   ├── ARTIFACT_SCHEMA_VERSION
│   ├── README.md
│   ├── VERSION
│   ├── domains/
│   │   └── _template.md
│   └── templates/
│       ├── INTENT.md
│       ├── DESIGN.md
│       ├── VERIFICATION_LOG-template.md
│       └── DOMAIN_PROFILE-template.md
├── catalog/                         ← only if you copied a profile
│   └── [your-profile].md
└── docs/
```

Open `AGENTS.md` and verify it contains reading order instructions that point to `framework/BUILDER.md` first, then `framework/GATEKEEPER.md`, then `framework/domains/` (only when BUILDER step 2 routes you there).

`framework/VERSION` tells you which framework release you copied. `framework/ARTIFACT_SCHEMA_VERSION` tells you which artifact contract the templates expect. When the schema changes, migrate your existing docs before treating old artifacts as current.

The key principle: BUILDER.md owns the profile-selection algorithm (step 2). It deterministically decides whether to load an existing profile, link to a catalog base, or create a minimal skeleton. Agents do not pre-select profiles by filename guess — they follow the algorithm.

In single-agent environments (most CLI tools and IDE assistants), the same LLM fulfills both Builder and GateKeeper roles — `BUILDER.md` defines how that works. The process logic lives in their respective files.

## Step 2: Write your prompt

Open your LLM and write a prompt. Start with the magic line, then describe what you want:

```
Read AGENTS.md. Build a task management widget
that runs inside ChatGPT as an MCP App.
Stack: Lit 3, Vite, TypeScript, MCP SDK.
```

The more context you provide, the better the result. Add constraints if you have them:

```
Read AGENTS.md. Build a task management widget
that runs inside ChatGPT as an MCP App.
Stack: Lit 3, Vite, TypeScript, MCP SDK.

Use signals for state management.
No inline scripts in the widget HTML — ChatGPT blocks them.
Tests must run in a real browser, not jsdom.
```

Send the prompt.

## Step 3: Watch the LLM determine the size

The LLM reads `AGENTS.md`, follows the link to `BUILDER.md`, and determines the project size:

- **Quick** — less than 3 files, clear scope. Skips most documentation.
- **Standard** — feature-sized. Produces Intent + Verification Log. Design is optional — if the architecture is straightforward, key decisions go directly in the Intent.
- **Full** — new project or major refactor. Everything from Standard plus a mandatory Design document with ADR Summary and Devil's Advocate review.

For a new project like ours, it will choose **Standard** or **Full**. You will see it say something like: *"This is a Full-sized project. I'll start with the Intent document."*

## Step 4: Review the Intent

The LLM creates `docs/[project]-intent.md`. Open it. You will see:

- **Goal** — one or two sentences about what we're building
- **Behavior** — given/when/then descriptions of how the software should work
- **Decisions** — choices the LLM made, with rejected alternatives
- **Constraints** — MUST, MUST NOT, and SHOULD rules
- **Scope** — what's IN and what's explicitly OUT

This is your chance to course-correct. If a decision is wrong, say so now. If a constraint is missing, add it. The Intent is the anchor — everything else traces back to it.

> **Tip:** When you tell the LLM "never use X", that becomes a permanent constraint. If it applies to all projects with this stack (not just this one), tell the LLM: *"Add this to the domain profile's Decision History."*

## Step 5: Watch the domain profile load (or get created)

If a matching base profile exists in `catalog/`, the LLM creates a **profile link** in `framework/domains/` with `extends: [profile-id]` and reads every pitfall and adversary question from the base before continuing. The link template is `framework/domains/_template.md`.

If no base profile exists, the LLM creates a **standalone full profile** directly in `framework/domains/` using `framework/templates/DOMAIN_PROFILE-template.md`. The first version will be minimal — terminology mapping, verification commands, a couple of pitfalls. It will grow as the project discovers new pitfalls. When you want to reuse it across projects, move it to `catalog/` and replace it with a link.

## Step 6: Skills get loaded (if they exist)

If your repo has skills installed in `.github/skills/`, `.agents/skills/`, or `.claude/skills/`, the LLM scans their descriptions and loads any that match the task. For example, a `frontend-design` skill would be loaded for a UI task but ignored for a backend API.

Skills provide design guidance — aesthetic direction, API conventions, documentation style — but they don't replace the domain profile or the verification gates. You might not have any skills yet, and that's fine. The framework works without them.

> **Tip:** You can install community skills from [skills.sh](https://skills.sh) (`npx skills add owner/skill-name`) or create your own in `.github/skills/your-skill/SKILL.md`, `.agents/skills/your-skill/SKILL.md`, or `.claude/skills/your-skill/SKILL.md`.

## Step 6.5: If your environment supports sub-agents

Some agent runtimes support sub-agents or delegated workers. In Agent Kit, use them as an execution aid, not as a replacement for artifacts:

- The **orchestrator** still owns `Intent`, `Design`, `Verification Log`, and phase state.
- A **sub-agent** can explore the codebase, read dependency APIs, implement a bounded change, or execute GateKeeper-style verification in isolated context.
- A **skill** remains a reusable instruction file; it is not the same thing as a sub-agent.

Quick rule:

- use a **domain profile** for reusable stack knowledge
- use a **skill** for reusable procedure
- use a **sub-agent** for isolated execution
- use **sub-agent + skill** when the worker should follow a reusable procedure

### Invoking sub-agents in Claude Code

Claude Code exposes the `Agent` tool. The orchestrator (your main Claude session) uses it to spawn workers. Two things matter: the **role** and the **briefing**.

**The briefing must be self-contained.** The sub-agent starts with no knowledge of your session. Do not tell it to "read AGENTS.md" — that makes it behave as a full orchestrator, not a scoped worker. Instead, paste the relevant excerpt directly into the prompt.

**Common patterns:**

```
// GateKeeper sub-agent — run a gate and return real output
You are a GateKeeper sub-agent. Run one verification gate and report the result.

Gate: 2 (Feature phase)
Command: npm run build && npm test  (run from project root: my-project/)
Project root: /path/to/my-project/

Return:
- exit code
- raw stdout/stderr (paste, do not paraphrase)
- failure classification: Product / Environment / Process
Do not edit any files.
```

```
// Research sub-agent — survey the codebase before design
You are a research sub-agent. Survey the existing codebase and return findings.

Files to read: src/server/, src/widget/
Question: What patterns does the current MCP transport use? What cannot change without breaking existing clients?

Return a concise summary: modules found, patterns in use, constraints imposed by existing code.
Do not make recommendations. Do not edit files.
```

```
// Dependency API reader — read an SDK before implementation
You are a dependency API reader. Read the public API of a library and return what is relevant.

Library: @modelcontextprotocol/ext-apps (installed at node_modules/)
Question: What does the App class export? What methods handle tool calls and notifications?

Return: relevant exported types, method signatures, and any version notes.
Do not implement anything.
```

After the sub-agent returns, **you** (the orchestrator) decide what is durable:

- A gate result → write it to `docs/[project]-verification.md`
- A codebase finding → summarize it in the Design's Research Summary section
- A new pitfall discovered during gate failure → add it to the domain profile

If you want to keep repeat delegations for a role consistent, author the prompt body once in `framework/templates/SUBAGENT_PROMPT_TEMPLATE.md` — it is a **prompt template**, not a runtime contract, since current tooling spawns sub-agents per-call without persistent role registration.

## Step 7: Review the Design (Full) or Decisions (Standard)

Before writing any code, the LLM runs a **pre-code checkpoint** — four questions that force it to pause: *Do my dependencies already solve this? What environment assumption could be wrong? Have I checked every pitfall? Is this still the right size?* This checkpoint is inlined into the process (step 4) so it cannot be skipped.

Then comes the design phase, which varies by size:

**Full projects** — the LLM creates `docs/[project]-design.md`, one document that replaces a separate PRD, tech spec, and implementation plan. It includes:

- **Domain Profile Selection** — which profile was chosen and why (with scores)
- **Skills Loaded** — which skills were loaded (or "none")
- **Stack** — technologies with verified versions
- **Architecture** — structure, data flow, initialization chain
- **Decisions** — every architectural choice with rationale
- **ADR Summary** — one durable row per major architecture decision in Full projects
- **Risks** — what could go wrong, identified before coding
- **Adversary Questions Applied** — answers to domain-specific traps from the profile
- **Domain Pitfalls Applied** — how each known pitfall is addressed

**Standard projects** — the Design is optional. If the architecture is straightforward, key decisions and the Adversary Questions / Domain Pitfalls tables are recorded directly in the Intent document's Decisions section. The invariant is that pitfalls and adversary questions are answered *somewhere* before code — not that a specific document exists.

The Adversary Questions and Domain Pitfalls sections are where the domain profile earns its value. They force the LLM to confront known failure modes *before* writing a single line of code.

## Step 8: Watch the gated build

Now the LLM starts building. It proceeds through gates:

1. **Gate 0** — installs dependencies, verifies they resolve
2. **Gate 1** — creates the scaffold, runs the build, verifies output exists
3. **Gate 2** — implements features, builds, runs existing tests, verifies no regressions
4. **Gate 3** — writes tests, runs the full suite
5. **Gate 4** — clean build from scratch (deletes everything, reinstalls, rebuilds, retests)

After each gate, the LLM records the real command result in `docs/[project]-verification.md`. Passing gates may use the compact format; failures, blocked gates, and command substitutions use the expanded format with raw output. Open it periodically — you will see real evidence, not claims.

If a gate fails, the LLM:
1. Records the failure in the verification log
2. Root-causes it (not just fixes the symptom)
3. Fixes the code
4. Re-runs the gate from scratch
5. Updates the domain profile with what it learned

This last step is the learning cycle in action. A failure today becomes a prevention for tomorrow.

## Step 9: Check the verification log

When the build is complete, open `docs/[project]-verification.md`. At the top you will see the Progress table:

```markdown
| Step | Status |
|------|--------|
| Intent | PASS |
| Skills loaded | PASS |
| Design | PASS |
| Gate 0: Dependencies | PASS |
| Gate 1: Scaffold | PASS |
| Gate 2: Feature | PASS |
| Gate 3: Tests | PASS |
| Gate 4: Clean build | PASS |
| Self-Review | PASS |
| Domain update | PASS |
```

Below that: real output for every gate, a failure history (if anything failed along the way), and a record of what was updated in the domain profile.

If Gate 4 passed, the project builds and tests from a clean state. That's the proof.

## Step 10: Check what the LLM learned

After the project, check what the LLM added to the domain profile.

**If you created a standalone profile** (`framework/domains/[your-profile].md`) — all discoveries are in that single file: new pitfalls, adversary questions, decision history, and automated checks.

**If you created a profile link** (`framework/domains/[your-profile].md` with `extends`) — check two places:

- **Your profile link** — project-specific discoveries: new Local Pitfalls (using the same pitfall metadata fields as base profiles), Local Decision History
- **The base profile** (`catalog/[profile-id].md`) — stack-wide discoveries: new Common Pitfalls, Adversary Questions, Decision History, Automated Checks

This is the flywheel. The next project on this stack inherits all accumulated knowledge. For profile links, local pitfalls that prove useful across projects should be contributed back to the catalog profile.

## Step 11: Resume interrupted work (when it happens)

Sessions get interrupted — context limits, network issues, or just closing the chat. When you come back, start a new session and prompt:

```
Read AGENTS.md. Continue where the last session left off.
```

The LLM reads the verification log's Progress section, finds the last completed step, and continues from there. No rework.

## What to do next

**Start another project on the same stack.** Notice how the domain profile prevents mistakes that happened in the first project. The second project will be smoother. The third, smoother still.

**Add constraints as you discover preferences.** Every time you say "always do X" or "never do Y", the framework captures it — in the Intent for this project, in the domain profile for all future projects on this stack.

**Bring profiles to new repos.** When you start a new repository with the same stack, copy `framework/` and include the relevant base profiles from `catalog/`. Create a new profile link in `framework/domains/` that extends the base. All accumulated stack knowledge travels with it; project-specific knowledge stays behind.

For the full technical reference — file descriptions, gate definitions, domain profile contract, and artifact specs — see [`framework/README.md`](framework/README.md).

For the concepts behind the framework — why it works, how the learning cycle operates, what makes domain profiles different — see the [project README](README.md).
