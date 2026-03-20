# Tutorial: Your first project with Agent Kit

In this tutorial, we will set up Agent Kit in a new repository and use it to build a project with an LLM. By the end, you will have seen the full cycle: prompt, intent, design, gated build, and a domain profile that captures what the LLM learned.

## What we need

- A repository (new or existing) where you want to build something
- Access to an LLM that can read files (Claude Code, ChatGPT with file access, Cursor, Windsurf, or similar)
- A project idea and a technology stack in mind

## Step 1: Copy the framework into your repo

Copy three things into your repository's root:

```bash
cp -R framework/ your-repo/framework/
cp AGENTS.md your-repo/
mkdir -p your-repo/docs
```

Your repo should now look like this:

```
your-repo/
├── AGENTS.md
├── framework/
│   ├── BUILDER.md
│   ├── domains/
│   │   └── _template.md
│   └── templates/
│       ├── INTENT.md
│       ├── DESIGN.md
│       └── VERIFICATION_LOG-template.md
└── docs/
```

Open `AGENTS.md` and verify it contains:

```markdown
# Agent Instructions

Read and follow `framework/BUILDER.md` for all tasks.

Project artifacts (intent, design, verification) go in `docs/`.
```

That's all `AGENTS.md` needs. The process logic lives in `BUILDER.md`.

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
- **Standard** — feature-sized. Produces Intent + Design + Verification Log.
- **Full** — new project or major refactor. Everything from Standard plus ADRs and Devil's Advocate review.

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

If a domain profile exists for your stack in `framework/domains/`, the LLM loads it and reads every pitfall and adversary question before continuing.

If no profile exists, the LLM creates one from `framework/domains/_template.md`. The first version will be minimal — terminology mapping, verification commands, a couple of pitfalls. That's fine. It will grow.

## Step 6: Review the Design

The LLM creates `docs/[project]-design.md`. This is one document that replaces a separate PRD, tech spec, and implementation plan. You will see:

- **Domain Profile Selection** — which profile was chosen and why (with scores)
- **Stack** — technologies with verified versions
- **Architecture** — structure, data flow, initialization chain
- **Decisions** — every architectural choice with rationale
- **Risks** — what could go wrong, identified before coding
- **Adversary Questions Applied** — answers to domain-specific traps from the profile
- **Domain Pitfalls Applied** — how each known pitfall is addressed

The Adversary Questions and Domain Pitfalls sections are where the domain profile earns its value. They force the LLM to confront known failure modes *before* writing a single line of code.

## Step 7: Watch the gated build

Now the LLM starts building. It proceeds through gates:

1. **Gate 0** — installs dependencies, verifies they resolve
2. **Gate 1** — creates the scaffold, runs the build, verifies output exists
3. **Gate 2** — implements features, builds, runs existing tests, verifies no regressions
4. **Gate 3** — writes tests, runs the full suite
5. **Gate 4** — clean build from scratch (deletes everything, reinstalls, rebuilds, retests)

After each gate, the LLM pastes the real command output into `docs/[project]-verification.md`. Open it periodically — you will see actual terminal output, not claims.

If a gate fails, the LLM:
1. Records the failure in the verification log
2. Root-causes it (not just fixes the symptom)
3. Fixes the code
4. Re-runs the gate from scratch
5. Updates the domain profile with what it learned

This last step is the learning cycle in action. A failure today becomes a prevention for tomorrow.

## Step 8: Check the verification log

When the build is complete, open `docs/[project]-verification.md`. At the top you will see the Progress table:

```markdown
| Step | Status |
|------|--------|
| Intent | PASS |
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

## Step 9: Check the domain profile

Open `framework/domains/[your-profile].md`. Compare it to how it looked before the project. You may see:

- New entries in **Common Pitfalls** — things the LLM discovered during implementation
- New **Adversary Questions** — traps specific to this stack
- New **Decision History** entries — constraints learned the hard way
- Updated **Automated Checks** — new detection patterns

This is the flywheel. The next project on this stack will load this profile and avoid the problems this project discovered.

## Step 10: Resume interrupted work (when it happens)

Sessions get interrupted — context limits, network issues, or just closing the chat. When you come back, start a new session and prompt:

```
Read AGENTS.md. Continue where the last session left off.
```

The LLM reads the verification log's Progress section, finds the last completed step, and continues from there. No rework.

## What to do next

**Start another project on the same stack.** Notice how the domain profile prevents mistakes that happened in the first project. The second project will be smoother. The third, smoother still.

**Add constraints as you discover preferences.** Every time you say "always do X" or "never do Y", the framework captures it — in the Intent for this project, in the domain profile for all future projects on this stack.

**Bring profiles to new repos.** When you start a new repository with the same stack, copy the domain profile along with `framework/`. All accumulated knowledge travels with it.

For the full technical reference — file descriptions, gate definitions, domain profile contract, and artifact specs — see [`framework/README.md`](framework/README.md).

For the concepts behind the framework — why it works, how the learning cycle operates, what makes domain profiles different — see the [project README](README.md).
