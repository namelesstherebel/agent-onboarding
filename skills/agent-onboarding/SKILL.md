--- name: agent-onboarding
description: >
  Use this skill whenever a user wants to set up a new repo, project, or environment for autonomous agent operation. Triggers include: "set up a new project", "onboard this repo", "make this agent-ready", "start a new workflow environment", "drop this into a repo", "implement this on my existing project", or any request to systematically build out context, intent, specs, or agent infrastructure from scratch or on top of existing work. This skill guides the user through 7 progressive phases — Project Discovery → Context Engineering → Intent Engineering → Specification Readiness → Environment Build → Specification Writing → Verify & Launch — and installs a self-improving runtime loop that continuously evolves the environment through friction-triggered improvement proposals. Works on greenfield projects AND existing repos. Do NOT use for one-off tasks or single prompts.
---

# Unified Agent Onboarding + Self-Improving Runtime Skill

This skill does two things that compound each other:

1. **Onboards any repo** — new or existing — into a fully specified, agent-ready environment
2. **Installs a self-improving runtime** that gets better every time an agent runs

These are two layers, not two tools. Onboarding runs once. The runtime runs forever.

---

## Mode Detection

At the start of every session, determine which mode applies:

**Onboarding mode** — triggered when:
- User explicitly asks to onboard, set up, or make a repo agent-ready
- `ONBOARDING_STATE.md` exists and shows an incomplete phase
- No `RUNTIME.md` exists in the repo root

**Runtime mode** — triggered when:
- `RUNTIME.md` exists in the repo root
- Onboarding is complete (Phase 7 checked off)
- User is running an agent task (not setting up)

If both conditions exist (runtime installed but onboarding state file shows incomplete): **finish onboarding first**, then hand off to runtime.

---

## Signal System (Active in All Modes)

| Signal | Action |
|--------|--------|
| `"ready"` or `"next"` | Advance to next phase |
| `"skip"` | Accept placeholders, advance |
| `"pause"` | Write `ONBOARDING_STATE.md`, stop cleanly |
| `"back"` | Return to previous phase |
| `*review` | Surface `IMPROVEMENT_QUEUE.md` for human review |
| `*reflect` | Manually trigger a reflection and proposal generation |
| `*status` | Report environment health — spec coverage, open proposals, last audit |

---

# PART ONE: ONBOARDING WORKFLOW

## Before Phase 1 — Existing Repo Analysis

**Always do this before asking a single question.**

If the user is onboarding an existing repo, read the environment before interviewing them. You already have information — use it.

This analysis runs in three stages. Each stage builds on the previous. Don't skip ahead.

### Stage 1 — Structural Analysis (deterministic)

Gather facts. No LLM interpretation yet.

1. List all files in the repo root and top-level directories
2. Read: `README.md`, `package.json` / `composer.json` / `requirements.txt` (whichever exists)
3. Read: any existing `docs/` or documentation files
4. Read: any existing `.md` files at root level
5. Check for: existing `CLAUDE.md`, `INTENT.md`, `specs/`, `CONTEXT/` — note what's already done
6. Check for: `.env.example`, `docker-compose.yml`, `Makefile` — infer the stack
7. Check for: memory or RAG infrastructure — directories like `vector-store/`, `embeddings/`, `memory/`; config files for Pinecone, Chroma, Weaviate, FAISS, LlamaIndex, LangChain, or similar
8. Check for: special dependency manifests beyond standard package files — `Modelfile`, `*.llamafile`, custom model configs, ML framework configs, or unusual lock files
9. Check for: existing tests, CI config — infer quality standards
10. Map entry points — where does execution start? What are the public interfaces?
11. Note LOC distribution by directory — where does the complexity live?

### Stage 2 — Convention Extraction (LLM-assisted)

Sample 3-5 representative files from the most active directories. Extract:

- **Naming patterns** — files, functions, variables, classes, database columns
- **Error handling style** — exceptions vs error returns, logging patterns, error messages
- **Testing conventions** — test location, naming, setup/teardown patterns, assertion style
- **Import/dependency patterns** — how modules reference each other, barrel files, path aliases
- **Comment and documentation style** — when comments appear, what they cover

Don't infer conventions from a single file. Cross-reference across files — only record patterns that appear consistently.

### Stage 3 — Pattern Mining (implicit knowledge)

Look for things that aren't documented but are clearly intentional:

- Code that looks "wrong" but exists in multiple places — likely a deliberate workaround
- Inconsistencies between documentation and actual code — the code is the truth
- Performance patterns, caching strategies, or retry logic that isn't documented
- Dead code or TODOs that signal incomplete migrations or abandoned approaches

Present a brief **"Here's what I found"** summary organized by stage. Fill in what you already know. Only ask about gaps.

This respects their time and demonstrates you're working *with* them, not interrogating them.

If it's a **greenfield repo** (empty or near-empty): skip the analysis, go straight to Phase 1 questions.

---

## Phase 1 — Project Discovery

**Goal:** Understand what this is, who uses it, and what success looks like.

If existing repo analysis was done, lead with what you know:

> "Based on what I read, this looks like [X] — a [Y] built with [Z]. A few things I couldn't infer..."

Then fill gaps with targeted questions. Don't re-ask what the code already answers.

**Questions to fill gaps (use only what's needed):**

1. What is this project in plain language — what does it do?
2. Who or what is the primary user — human, another agent, an external system?
3. What's the problem it's solving? What's painful without it?
4. What does "working" look like? What's the definition of done?
5. What is this explicitly *not* doing? Where is the scope boundary?
6. Are there hard constraints — tech stack, timeline, compliance, budget?
7. Are there special dependencies, memory systems, or RAG infrastructure that are vital to the workflow? (e.g., a vector database, embedding pipeline, external memory store, fine-tuned model, or retrieval layer — anything the agent cannot function without that wouldn't be obvious from standard dependency files)
8. (Existing repos only) What's broken, incomplete, or undocumented that we should fix during onboarding?

**Produce `PROJECT_BRIEF.md`:**

```markdown
# Project Brief

**Name:** [project name]
**Owner:** [name]
**Created:** [date]
**Type:** [Greenfield / Existing repo — onboarded on date]

## What It Is
[One paragraph, plain language]

## Primary Actors
- [Who/what uses this and how]

## Problem Statement
[What breaks or is painful without this]

## Definition of Done
[What does "working" look like?]

## Explicit Non-Scope
[What this is not doing]

## Hard Constraints
[Tech, compliance, timeline, budget]

## Known Gaps (Existing Repos)
[What was incomplete or undocumented at time of onboarding]
```

Confirm with user before advancing.

**Advance:** `"ready"` → Phase 2

---

## Phase 2 — Context Engineering

**Goal:** Build the information environment the agent operates in.

For existing repos: much of this may already be inferable. Read the codebase before asking. Note conventions you observe — naming patterns, file structure, comment style. Ask only what you can't infer.

### Context Best Practices

Apply these principles when generating CLAUDE.md and all context files:

- **Keep it lean.** CLAUDE.md under 200 lines. Only instructions that apply to every session. When in doubt, cut it.
- **Budget the context window.** Rough targets: system prompt + CLAUDE.md ~20%, task spec ~20%, active code ~40%, interfaces + CONTEXT/ ~10%, reserve ~10%. When any category exceeds its budget, summarize (never truncate). These are starting points — adjust based on the model and task complexity.
- **Universal vs. conditional.** If an instruction only matters sometimes, it doesn't belong in CLAUDE.md. Use `.claude/rules/` with glob patterns for scoped context.
- **Never use context files as a linter.** Formatting and style rules belong in tooling (prettier, eslint, black, etc.). An LLM following style rules is slower, more expensive, and less reliable than a formatter.
- **Corrections over instructions.** The highest-value entries are things Claude gets wrong in this specific codebase. Generic instructions Claude would follow anyway are noise.
- **Pointers over content.** Tell Claude where to find information rather than embedding it. A file path or search directive is cheaper than reproducing content inline.
- **Understanding over code.** Context files should hold the agent's understanding *about* the code — architecture relationships, conventions, decision rationale — not reproduced source code. The filesystem is already infinite memory. CONTEXT/ files that are just copied code are wasting the window.
- **One directive per line.** If it takes a paragraph, it's not specific enough.
- **Progressive disclosure.** Don't front-load everything Claude could possibly need. Surface information on demand.
- **Every line pays rent.** If it's not actively correcting behavior or providing critical path information Claude can't infer, it's a cost with no return.

**Questions (use what's needed):**

1. What files or docs should the agent always have access to?
2. What conventions must the agent follow that aren't enforced by existing tooling? (Only conventions that linters/formatters don't already handle)
3. What tools, APIs, or external systems will the agent interact with?
4. What's the full tech stack?
5. What are recurring input types the agent receives?
6. Is there domain-specific vocabulary the agent needs?
7. What does agent memory look like — session-only, persistent, file-based, or RAG? If RAG: what's the data source, embedding model, vector store, and retrieval strategy?
8. Are there special dependencies vital to the workflow that aren't captured in standard package files?
9. What should the agent re-check at the start of each session?
10. (Existing repos) What does Claude get wrong or miss in this codebase? What corrections would save the most time?

**Produce `CLAUDE.md`:**

Keep this lean. One directive per line where possible. Point to files rather than embedding content. Only include what applies to every session.

```markdown
# [Project Name]

## Stack
[Languages, frameworks, databases, hosting — one line each]

## Corrections
[Things Claude gets wrong in this specific codebase — highest value section]
[e.g., "Use pgx not database/sql — the project migrated in Q3"]
[e.g., "The /api/v2 routes use middleware auth, not per-handler — see middleware/auth.go"]

## Conventions Not Enforced by Tooling
[Only conventions that linters/formatters don't handle]
[Skip anything already covered by .eslintrc, .prettierrc, pyproject.toml, etc.]

## Key Paths
[Pointers to important files/directories the agent needs frequently]
[e.g., "API routes: src/routes/ | DB migrations: db/migrations/ | Tests: __tests__/"]

## Tools & Integrations
[APIs, MCPs, external systems — connection notes and auth patterns]

## Never Do Without Asking
[Actions requiring explicit human approval — one per line]

## Known Complexity
[Parts of the codebase that are tricky — with file paths, not explanations]
```

Also produce:
- `.claude/rules/` directory — scoped context files with glob patterns for file-type or workflow-specific instructions that don't apply every session
- `CONTEXT/` directory — the agent's understanding of the codebase, not copies of source code. Architecture relationships, module boundaries, data flow, integration points. If it can be read from a file, point to the file instead.
- `CONTEXT/decisions.md` — append-only log of architectural and convention decisions with rationale. Never summarized, never pruned. The "why" behind choices is the context most vulnerable to loss over time. Format: `[date] [decision] — [why]`
- `ENVIRONMENT.md` — only if hosting, auth, or integrations have notable complexity

When deciding what goes where:
- CLAUDE.md → universal, every-session rules (under 200 lines)
- `.claude/rules/*.md` → conditional rules activated by glob patterns (zero cost when not relevant)
- `CONTEXT/` → agent's understanding of architecture and relationships (not reproduced source code)
- `CONTEXT/decisions.md` → append-only decision log preserving "why" (never compressed)

**Advance:** `"ready"` → Phase 3

---

## Phase 3 — Intent Engineering

**Goal:** Document what the agent optimizes for.

This is the most important document in the system. Intent governs everything. A well-written `INTENT.md` means agents make good judgment calls autonomously. A weak one means they optimize for the wrong thing at scale. Take time here.

For existing repos: look for implicit intent in existing docs, commit messages, README goals sections. Surface it explicitly and confirm.

**Questions:**

1. What is the agent's primary goal — what outcome matters most?
2. When speed conflicts with quality, which wins?
3. When brevity conflicts with completeness, which wins?
4. When the agent is uncertain, does it stop and ask, or log an assumption and continue?
5. What must always be flagged to a human regardless of confidence?
6. What's the agent's tone and approach?
7. What would a bad outcome look like — and how should the agent detect and avoid it?
8. Are there ethical, legal, or brand constraints on behavior?
9. What does the agent do if it finishes a task early?
10. What does the agent do if it runs out of context mid-task?
11. What friction threshold generates an improvement proposal? (see Runtime section)

**Produce `INTENT.md`:**

```markdown
# Agent Intent — [Project Name]

*Last updated: [date]*
*⚠️ Changes to this file require explicit human review and approval via *review*

## Primary Goal
[What the agent is fundamentally optimizing for — one sentence]

## Trade-Off Hierarchy
When priorities conflict, default to:
1. [Highest] over [lower] — [brief rationale]
2. [e.g., Correctness over speed — never ship broken output to save time]
3. [e.g., Explicit over assumed — surface uncertainty rather than guess]

## Uncertainty Protocol
When the agent doesn't have enough information to proceed correctly:
- [Stop and ask / log assumption and continue / generate improvement proposal]
- Threshold: [describe what "not enough information" looks like specifically]

## Friction Threshold
Generate an improvement proposal when:
- A task required more than [N] clarifying questions or assumptions
- The agent had to backtrack or redo work mid-task
- A spec step was ambiguous enough to have multiple valid interpretations
- The agent encountered an error not covered by existing specs or context
- The agent completed a task but is not confident the output is correct

## Always Escalate to Human
- [Condition 1 — e.g., any outbound communication before sending]
- [Condition 2 — e.g., any destructive or irreversible action]
- [Condition 3 — e.g., any proposed change to INTENT.md itself]

## Agent Tone & Approach
[How the agent communicates — direct, collaborative, formal, etc.]

## Failure Modes to Avoid
- [Failure 1] → [how to detect] → [how to avoid]
- [Failure 2] → [how to detect] → [how to avoid]

## Edge Case Default
[What the agent does when a situation isn't covered above]

## What Good Looks Like
[Positive description of ideal agent behavior in this environment]
```

**Advance:** `"ready"` → Phase 4

---

## Phase 4 — Specification Readiness

**Goal:** Inventory all recurring tasks. Identify what needs specs — and what needs harness gates.

For existing repos: look for existing runbooks, READMEs with process steps, Makefile targets, CI/CD configs. These are proto-specs — they just need to be formalized.

**Questions:**

1. What are the recurring tasks — things that happen more than once?
2. For each: does a written process exist? Clear enough for an agent without questions?
3. Are there deliverables with quality bars that need to be defined?
4. Are there multi-session tasks needing progress logs?
5. Are there approval gates — human sign-off before continuing?
6. (Existing repos) What processes exist only in someone's head?

### Harness Assessment

After inventorying tasks, evaluate each for harness-worthiness. Not every task needs code-enforced gates — most are fine as specs. But some tasks have failure modes that prompts and specs can't reliably prevent.

**A task qualifies for harness recommendation when any of these are true:**

1. **5+ sequential steps** where failure in one corrupts downstream steps
2. **High failure cost** — production data, client-facing output, financial impact, compliance/legal exposure
3. **Repeated agent drift** — the agent has historically skipped steps, marked tasks done prematurely, or violated the process despite clear spec instructions
4. **Verification dependency** — correctness requires programmatic checking (test suites, schema validation, visual diff) that the agent can't reliably self-assess
5. **Knowledge distribution required** — the system doesn't learn unless specific post-task steps happen (closing gates, knowledge base updates, log distribution)

For each qualifying task, assess using the **March of Nines** lens:
- How many sequential steps does this workflow have?
- What's the approximate per-step reliability with a spec alone?
- What's the compounding failure rate? (p^n where p = per-step rate, n = step count)
- What specific gates would move it from spec-level to harness-level reliability?

**Harness gate types to recommend:**

| Gate Type | What It Does | When to Recommend |
|---|---|---|
| **Pre-flight** | Script that auto-loads required context/files before task starts | Agent frequently starts tasks without full context |
| **Validation gate** | Script that checks output meets criteria before allowing next phase | Outputs have structured requirements (schema, format, completeness) |
| **Visual verification** | Automated screenshot/diff with human approval | CSS, layout, or design tasks where the agent can't judge correctness |
| **Closure validator** | Script that checks all post-task steps were completed | Knowledge distribution, logging, or cleanup steps get skipped |
| **Test gate** | Script that runs relevant tests and blocks on failure | Code changes that must pass tests before merging |
| **Approval gate** | Programmatic pause requiring explicit human sign-off | Destructive actions, external communications, production deployments |

**Produce `SPEC_INVENTORY.md`:**

```markdown
# Specification Inventory — [Project Name]

*Last updated: [date]*

## Recurring Tasks

| Task | Spec Exists? | Quality Bar Defined? | Multi-Session? | Harness? | Priority |
|------|-------------|---------------------|----------------|----------|----------|
| [Task 1] | No | No | No | No | High |
| [Task 2] | Partial | Yes | Yes | **Yes** | Medium |

## Phase 6 Queue (write in this order)
1. [Spec name] — [what it covers]
2. [Spec name]
3. [Spec name]

## Harness Assessment

### Tasks Requiring Harness Gates

| Task | Steps | Est. Per-Step Rate | Compounding Rate | Failure Cost | Recommended Gates |
|------|-------|--------------------|------------------|--------------|-------------------|
| [Task] | [N] | [%] | [p^n] | [High/Critical] | [Gate types] |

### Tasks Where Specs Are Sufficient
[List tasks assessed but not needing harness gates, with brief rationale]

### Recommended Harness Artifacts
[For each task needing gates: what scripts/validators to build, where they integrate into the workflow]

## Existing Runbooks to Formalize (Existing Repos)
- [Doc/file name] → [spec it should become]

## Notes
[Anything notable about the spec landscape]
```

If the harness assessment identifies tasks needing gates, build those gate scripts during Phase 5. Add sub-step **5G — Harness Gates** after the smoke test:

### 5G — Harness Gates *(if harness assessment identified qualifying tasks)*

For each task with recommended harness gates:
1. Build the gate script (shell preferred, Python if needed)
2. Place in `scripts/` or `harness/` directory
3. Document in `CLAUDE.md` under a Harness Gates section
4. Reference in the relevant spec's process steps — the spec should call the gate script at the appropriate point
5. Smoke test each gate script independently

**Advance:** `"ready"` → Phase 5

---

## Phase 5 — Environment Build

**Stop asking. Start building.**

Work through in order. Confirm each section before moving to the next.

### 5A — File Structure

```
/[repo-root]/
├── CLAUDE.md                 ← Agent context (lean, under 200 lines)
├── INTENT.md                 ← Agent intent (⚠️ protected)
├── PROJECT_BRIEF.md          ← Project brief
├── SPEC_INVENTORY.md         ← Spec queue
├── RUNTIME.md                ← Self-improving runtime (installed here)
├── IMPROVEMENT_QUEUE.md      ← Proposal queue (starts empty)
├── ONBOARDING_STATE.md       ← Onboarding progress tracker
├── WORKTREES.md              ← Worktree profiles and usage (if worktrees enabled)
├── .claude/
│   └── rules/                ← Scoped context (glob-activated, zero cost when inactive)
│       └── [pattern-name].md
├── CONTEXT/                  ← Agent's understanding of the codebase (not reproduced source)
│   └── decisions.md          ← Append-only decision log (never summarized)
├── SPECS/                    ← Agent-executable specifications
│   └── [task-name].md
├── profiles/                 ← Behavior profiles (if worktrees enabled)
│   └── [name]/
│       ├── CLAUDE.md         ← Behavior-mode context
│       └── INTENT.md         ← Behavior-mode trade-offs
├── worktrees/                ← Worktree entry points (if worktrees enabled)
│   └── [name]/
│       └── CLAUDE.md         ← Thin file, references root + profile
├── LOGS/                     ← Agent session logs + error logs
│   ├── sessions/
│   └── errors/
└── [project-specific dirs]
```

### 5B — Dependencies

Install and configure everything from the stack definition in `CLAUDE.md`:
- Language runtimes and package managers
- Project packages
- Dev tooling (linters, formatters, test runners)
- CLI tools referenced in `CLAUDE.md`

### 5C — MCP Connections

For each external system in `CLAUDE.md`:
- Confirm MCP server availability
- Configure connection
- Run a minimal smoke test call
- Document confirmed connection in `CLAUDE.md`

### 5D — Rules Files

Based on `INTENT.md` and the context best practices, create:
- `.claude/rules/` scoped context files — conditional instructions with glob patterns that only activate when relevant (e.g., `*.test.ts` rules, `migrations/*.sql` rules, `docs/*.md` rules). Move any Phase 2 instructions that aren't universal here.
- `AGENTS.md` — for Claude Code environments
- `.cursorrules` / `.windsurfrules` — for AI IDE environments
- Custom rule files per project conventions

When creating scoped rules, each file should specify its glob pattern and contain only the instructions relevant to that file pattern. This keeps CLAUDE.md lean and avoids loading irrelevant context.

**Task-phase rules:** Different work modes need different context. Consider creating rules scoped to task phase, not just file type:
- **Exploring/understanding** → architecture map, dependency graph, module boundaries. Useful when the agent is reading code it hasn't seen before.
- **Implementing** → conventions, patterns, interface contracts, active file neighbors. Useful when the agent is writing new code.
- **Debugging** → error patterns from `LOGS/errors/`, known failure modes from specs, relevant test fixtures. Useful when the agent is diagnosing failures.
- **Reviewing** → quality bars from specs, convention checklist, common mistakes from `CONTEXT/decisions.md`.

These can be activated by naming convention (e.g., specs named `debug-*.md` or `review-*.md`) or by explicit instruction in the relevant spec's process steps.

### 5E — Install Runtime

Copy `RUNTIME.md` into the repo root. This is the self-improving operating system. It activates automatically after onboarding completes.

### 5F — Smoke Test

Run one minimal agent task end-to-end. Verify the environment works before writing specs.

### 5F — Worktree Setup *(optional)*

After the smoke test, ask:

> "Do you plan on using git worktrees with this repo?"

**No** → skip to Phase 6.

**Yes** → continue with the steps below.

**Step 1 — Identify behavior profiles**

Ask which behavior profiles they want. Suggest these as defaults:
- `scaffold` — building new features, files, and structure from scratch
- `refactor` — restructuring existing code without changing behavior
- `debug` — diagnosing and resolving failures
- `review` — auditing code for correctness, clarity, and spec alignment

Accept custom profile names. After presenting suggestions, ask: *"Any additional profiles?"* — repeat until the user says no.

**Step 2 — Build profile files**

For each confirmed profile, generate:

```
/profiles/[name]/
├── CLAUDE.md     ← Behavior-mode context: allowed actions, restrictions, focus areas
└── INTENT.md     ← Behavior-mode trade-offs: goal, priority order, uncertainty protocol
```

**Profile file constraints — enforce strictly:**
- Profile files must be **project-agnostic** — no stack names, filenames, domain terms, or technology references
- Those specifics live in the root `CLAUDE.md`
- Profiles are **behavioral contracts only** — they define how the agent operates, not what it operates on
- This makes profiles reusable across repos

`CLAUDE.md` template for a profile:
```markdown
# [Profile Name] Mode

## Allowed Actions
[What this mode permits — behavioral, not technical]

## Restrictions
[What this mode prohibits or requires extra caution around]

## Focus
[What the agent prioritizes its attention on in this mode]

## Entry Checklist
[What to verify before starting work in this mode]
```

`INTENT.md` template for a profile:
```markdown
# [Profile Name] Intent

## Goal
[What this mode is optimizing for — one sentence]

## Priority Order
1. [Highest priority]
2. [Second priority]
3. [Third priority]

## Uncertainty Protocol
[How to handle uncertainty specific to this mode]

## Exit Criteria
[When this mode's work is considered done]
```

**Step 3 — Build worktree entry points**

For each profile, generate a thin `CLAUDE.md` in the corresponding worktree directory:

```
/worktrees/[name]/
└── CLAUDE.md
```

Worktree `CLAUDE.md` template:
```markdown
# [Name] Worktree

## Context Inheritance
This worktree inherits from:
- Root `CLAUDE.md` — project context and universal rules
- Root `INTENT.md` — project-level goals and trade-offs
- `profiles/[name]/CLAUDE.md` — behavior-mode context
- `profiles/[name]/INTENT.md` — behavior-mode trade-offs

## Branch Naming
[branch-prefix]/[descriptor] — e.g., `[name]/add-login-flow`

## Notes
[Any worktree-specific operational notes]
```

**Step 4 — Generate `WORKTREES.md` at repo root**

```markdown
# Worktrees

*Last updated: [date]*

## Active Profiles

| Profile | Purpose | Branch Pattern |
|---------|---------|---------------|
| [name] | [one-line purpose] | `[name]/[descriptor]` |

## Inheritance Chain

Each worktree loads context in this order:
1. Root `CLAUDE.md` — universal project context
2. Root `INTENT.md` — project goals and trade-offs
3. `profiles/[name]/CLAUDE.md` — behavior-mode context
4. `profiles/[name]/INTENT.md` — behavior-mode trade-offs

## Usage

```bash
# Create a new worktree
git worktree add ../[repo]-[name] -b [name]/[descriptor]

# List active worktrees
git worktree list

# Remove a worktree
git worktree remove ../[repo]-[name]
```

## Profile Notes
[Any cross-profile guidance or constraints]
```

**Advance:** `"ready"` → Phase 6

---

## Phase 6 — Specification Writing

Write agent-executable specs for every item in the Phase 6 queue. One spec at a time, in priority order. Confirm each before moving to the next. Save each as `SPECS/[task-name].md`.

**Spec template:**

```markdown
# [Task Name] Specification

*Version: 1.0 | Last updated: [date]*

## Purpose
What is this task? Why does it exist?

## Trigger
What causes this task to start?

## Inputs Required
- [Input 1 — name, type, source]
- [Input 2]

## Output Definition
- Format: [file type, structure]
- Quality bar: [how to evaluate correctness]
- Destination: [where it goes when done]

## Step-by-Step Process
1. [Step 1 — specific enough that a new agent follows it without asking]
2. [Step 2]
3. [Step 3]

## Quality Checklist
- [ ] [Check 1]
- [ ] [Check 2]

## Approval Gates
- [Gate — what triggers it, what the human decides]

## Error Handling
- [Error type] → [what to do] → [what to log]

## Progress Log Pattern
*(Multi-session tasks only — append to LOGS/sessions/[task-name]-[date].md)*

### Session [N] — [date]
- Completed: [what was finished]
- Stopped at: [exactly where the agent paused]
- Next session starts with: [first action]
- Blockers: [anything needing human input]
- Friction encountered: [any assumptions made or ambiguities hit]

## Common Failure Modes
- [Failure] → [how to detect] → [how to recover] → [what to log]

## Version History
| Version | Date | Change | Triggered by |
|---------|------|--------|-------------|
| 1.0 | [date] | Initial spec | Onboarding |
```

**Advance:** `"all specs done"` or `"ready"` → Phase 7

---

## Phase 7 — Verify and Launch

### Structural Verification

- [ ] `PROJECT_BRIEF.md` is accurate and complete
- [ ] `CLAUDE.md` is under 200 lines, contains only universal rules
- [ ] `CLAUDE.md` has no formatting/style rules that belong in tooling
- [ ] `CLAUDE.md` uses pointers (file paths) not embedded content
- [ ] Conditional instructions are in `.claude/rules/` with glob patterns, not in CLAUDE.md
- [ ] `INTENT.md` covers known edge cases and trade-offs
- [ ] Friction threshold is clearly defined in `INTENT.md`
- [ ] `SPEC_INVENTORY.md` is complete and prioritized
- [ ] `CONTEXT/decisions.md` exists (may be empty for greenfield)
- [ ] All Phase 5 dependencies installed and tested
- [ ] All MCP connections live and smoke-tested
- [ ] `RUNTIME.md` is present in repo root
- [ ] `IMPROVEMENT_QUEUE.md` exists (empty is fine)
- [ ] `LOGS/sessions/` and `LOGS/errors/` directories exist
- [ ] *(If worktrees)* All profiles have `CLAUDE.md` and `INTENT.md` in `profiles/[name]/`
- [ ] *(If worktrees)* `WORKTREES.md` exists at repo root and accurately reflects all profiles
- [ ] *(If worktrees)* `SPEC_INVENTORY.md` notes which specs belong to which profile

### Behavioral Verification

An agent that asks itself "do I understand this codebase?" says yes regardless. Verify through execution instead.

- [ ] Run a small real task (a bug fix, a test, or a minor feature) end-to-end using only the onboarded context
- [ ] Agent completed the task without asking clarifying questions covered by existing artifacts
- [ ] Agent followed extracted conventions without being reminded (check the output against Stage 2 conventions)
- [ ] Agent used the correct paths, tools, and patterns from CLAUDE.md without hallucinating alternatives
- [ ] If the agent violated a convention or used a wrong path, update the relevant artifact and re-test
- [ ] *(If worktrees)* Each profile smoke-tested with a real agent run in its worktree

### Reconciliation Check

- [ ] Run one reconciliation pass: compare CLAUDE.md Key Paths against actual file structure
- [ ] Verify CONTEXT/ files describe the current state of the code, not a stale snapshot

Any unchecked box: return to the relevant phase and fix.

When all boxes are checked: **onboarding is complete.** `RUNTIME.md` takes over.

Update `ONBOARDING_STATE.md` to `Status: Complete`.

---

## State Management

Write or update `ONBOARDING_STATE.md` at the end of any incomplete session:

```markdown
# Onboarding State

**Status:** In Progress / Complete
**Last updated:** [date]
**Current phase:** [number and name]
**Completed phases:** [list]
**Next action:** [exactly what happens when we resume]
**Open questions:** [anything unresolved]
```

On session start: read this file before asking anything. Resume from recorded state.

---

# PART TWO: RUNTIME BEHAVIOR

*(This section governs how you behave after onboarding is complete, when running agent tasks in an onboarded repo.)*

## Session Start Protocol

Every session in an onboarded repo:
1. Read `RUNTIME.md` — confirms runtime is active
2. Read `CLAUDE.md` — loads context
3. Read `INTENT.md` — loads goals and trade-off hierarchy
4. Check `IMPROVEMENT_QUEUE.md` — note any open proposals (don't interrupt task for them)
5. Check `LOGS/errors/` — note any recent errors relevant to current task
6. Confirm the relevant spec exists before starting task work

## Friction Detection

During task execution, track friction in real time. Friction is defined as:
- Making an assumption because the spec or context didn't cover a case
- Asking a clarifying question that a better spec would have answered
- Backtracking or redoing work because initial understanding was wrong
- Encountering an error not covered by existing error handling
- Completing a task but lacking confidence the output meets the quality bar
- Encountering ambiguity with multiple valid interpretations

Each friction event gets logged internally. At task completion, evaluate against the threshold in `INTENT.md`. If threshold is met or exceeded: generate a proposal.

## Error Logging

When an unexpected error occurs:
1. Log immediately to `LOGS/errors/[date]-[task]-[N].md`
2. Evaluate whether to stop and escalate or log and continue based on `INTENT.md` uncertainty protocol
3. If the error indicates a structural gap: generate an improvement proposal

## *reflect Command

When triggered manually or at natural task completion checkpoints:
1. Review friction events from current session
2. Review any new error logs
3. Evaluate against friction threshold in `INTENT.md`
4. Generate proposals for anything that meets the threshold
5. Run context reconciliation — compare CLAUDE.md paths, CONTEXT/ descriptions, and `.claude/rules/` globs against actual codebase. Generate proposals for any drift. Never patch a stale summary from another summary — regenerate from the code.
6. Report to user: *"Reflected on [N] tasks. Generated [N] proposals. Use *review to see them."*

## *review Command

When triggered:
1. Open `IMPROVEMENT_QUEUE.md`
2. Surface all proposals with status `PENDING`
3. For each proposal, present with a recommended action
4. Accept user decision: approve / reject / modify
5. On approval: apply the change, update status, increment spec version
6. On approval of convention/architecture changes: append to `CONTEXT/decisions.md` with date, decision, and rationale
7. On rejection: update status with reason
8. On modify: user edits inline, agent applies modified version

**⚠️ INTENT.md proposals require explicit `APPROVE INTENT CHANGE` confirmation.**

## *status Command

Report current environment health: spec coverage, open proposals, recent activity, health flags.
