---
name: agent-onboarding
description: >
  Use this skill whenever a user wants to set up a new repo, project, or
  environment for autonomous agent operation. Triggers include: "set up a new
  project", "onboard this repo", "make this agent-ready", "start a new workflow
  environment", "drop this into a repo", "implement this on my existing
  project", or any request to systematically build out context, intent, specs,
  or agent infrastructure from scratch or on top of existing work. This skill
  guides the user through 7 progressive phases — Project Discovery → Context
  Engineering → Intent Engineering → Specification Readiness → Environment
  Build → Specification Writing → Verify & Launch — and installs a
  self-improving runtime loop that continuously evolves the environment through
  friction-triggered improvement proposals. Works on greenfield projects AND
  existing repos. Do NOT use for one-off tasks or single prompts.
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
| `/review` | Surface `IMPROVEMENT_QUEUE.md` for human review |
| `/reflect` | Manually trigger a reflection and proposal generation |
| `/status` | Report environment health — spec coverage, open proposals, last audit |

---

# PART ONE: ONBOARDING WORKFLOW

## Before Phase 1 — Existing Repo Analysis

**Always do this before asking a single question.**

If the user is onboarding an existing repo, read the environment before interviewing them. You already have information — use it.

1. List all files in the repo root
2. Read: `README.md`, `package.json` / `composer.json` / `requirements.txt` (whichever exists)
3. Read: any existing `docs/` or documentation files
4. Read: any existing `.md` files at root level
5. Check for: existing `CLAUDE.md`, `INTENT.md`, `specs/`, `CONTEXT/` — note what's already done
6. Check for: `.env.example`, `docker-compose.yml`, `Makefile` — infer the stack
7. Check for: existing tests, CI config — infer quality standards
8. Summarize what you learned before asking anything

Present a brief **"Here's what I found"** summary to the user. Fill in what you already know. Only ask about gaps. This respects their time and demonstrates you're working *with* them, not interrogating them.

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
7. (Existing repos only) What's broken, incomplete, or undocumented that we should fix during onboarding?

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

**Questions (use what's needed):**

1. What files or docs should the agent always have access to?
2. What conventions must the agent follow? (naming, formatting, code style, communication)
3. What tools, APIs, or external systems will the agent interact with?
4. What's the full tech stack?
5. What are recurring input types the agent receives?
6. Is there domain-specific vocabulary the agent needs?
7. What does agent memory look like — session-only, persistent, file-based, RAG?
8. What should the agent re-check at the start of each session?
9. (Existing repos) Are there undocumented conventions the agent should know about?

**Produce `CLAUDE.md`:**

```markdown
# Agent Context — [Project Name]
*Last updated: [date]*

## Stack
[Languages, frameworks, databases, hosting — be specific]

## Conventions
[Naming, formatting, code style, communication patterns]
[For existing repos: note observed patterns + confirmed by owner]

## Tools & Integrations
[APIs, MCPs, external systems — with connection notes and auth patterns]

## Domain Knowledge
[Key terms, concepts, domain-specific knowledge]

## Check Each Session
[Things that may have changed — open issues, recent deploys, updated docs]

## Never Do Without Asking
[Actions requiring explicit human approval before executing]

## Known Complexity
[Parts of the codebase or environment that are tricky or require extra care]
```

Also produce:
- `CONTEXT/` directory — reference files the agent should always load
- `ENVIRONMENT.md` — if hosting, auth, or integrations have notable complexity

**Advance:** `"ready"` → Phase 3

---

## Phase 3 — Intent Engineering

**Goal:** Document what the agent optimizes for. This is the most important document in the system.

Intent governs everything. A well-written `INTENT.md` means agents make good judgment calls autonomously. A weak one means they optimize for the wrong thing at scale. Take time here.

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
*⚠️ Changes to this file require explicit human review and approval via /review*

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

**Goal:** Inventory all recurring tasks. Identify what needs to be written as agent-executable specs.

For existing repos: look for existing runbooks, READMEs with process steps, Makefile targets, CI/CD configs. These are proto-specs — they just need to be formalized.

**Questions:**

1. What are the recurring tasks — things that happen more than once?
2. For each: does a written process exist? Clear enough for an agent without questions?
3. Are there deliverables with quality bars that need to be defined?
4. Are there multi-session tasks needing progress logs?
5. Are there approval gates — human sign-off before continuing?
6. (Existing repos) What processes exist only in someone's head?

**Produce `SPEC_INVENTORY.md`:**

```markdown
# Specification Inventory — [Project Name]
*Last updated: [date]*

## Recurring Tasks
| Task | Spec Exists? | Quality Bar Defined? | Multi-Session? | Priority |
|------|-------------|---------------------|----------------|----------|
| [Task 1] | No | No | No | High |
| [Task 2] | Partial | Yes | Yes | Medium |

## Phase 6 Queue (write in this order)
1. [Spec name] — [what it covers]
2. [Spec name]
3. [Spec name]

## Existing Runbooks to Formalize (Existing Repos)
- [Doc/file name] → [spec it should become]

## Notes
[Anything notable about the spec landscape]
```

**Advance:** `"ready"` → Phase 5

---

## Phase 5 — Environment Build

**Stop asking. Start building.**

Work through in order. Confirm each section before moving to the next.

### 5A — File Structure

```
/[repo-root]/
├── CLAUDE.md                  ← Agent context
├── INTENT.md                  ← Agent intent (⚠️ protected)
├── PROJECT_BRIEF.md           ← Project brief
├── SPEC_INVENTORY.md          ← Spec queue
├── RUNTIME.md                 ← Self-improving runtime (installed here)
├── IMPROVEMENT_QUEUE.md       ← Proposal queue (starts empty)
├── ONBOARDING_STATE.md        ← Onboarding progress tracker
├── CONTEXT/                   ← Reference docs
├── SPECS/                     ← Agent-executable specifications
│   └── [task-name].md
├── LOGS/                      ← Agent session logs + error logs
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

Based on `INTENT.md`, create:
- `AGENTS.md` — for Claude Code environments
- `.cursorrules` / `.windsurfrules` — for AI IDE environments
- Custom rule files per project conventions

### 5E — Install Runtime

Copy `RUNTIME.md` into the repo root. This is the self-improving operating system. It activates automatically after onboarding completes.

### 5F — Smoke Test

Run one minimal agent task end-to-end. Verify the environment works before writing specs.

**Advance:** `"ready"` → Phase 6

---

## Phase 6 — Specification Writing

Write agent-executable specs for every item in the Phase 6 queue.

One spec at a time, in priority order. Confirm each before moving to the next.

Save each as `SPECS/[task-name].md`.

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

- [ ] `PROJECT_BRIEF.md` is accurate and complete
- [ ] `CLAUDE.md` loads correctly — agent can reference conventions
- [ ] `INTENT.md` covers known edge cases and trade-offs
- [ ] Friction threshold is clearly defined in `INTENT.md`
- [ ] `SPEC_INVENTORY.md` is complete and prioritized
- [ ] All Phase 5 dependencies installed and tested
- [ ] All MCP connections live and smoke-tested
- [ ] `RUNTIME.md` is present in repo root
- [ ] `IMPROVEMENT_QUEUE.md` exists (empty is fine)
- [ ] `LOGS/sessions/` and `LOGS/errors/` directories exist
- [ ] At least one spec tested end-to-end with a real agent run
- [ ] Agent completed a full task without asking clarifying questions

Any unchecked box: return to the relevant phase and fix.

When all boxes are checked: **onboarding is complete.** `RUNTIME.md` takes over. Update `ONBOARDING_STATE.md` to `Status: Complete`.

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

## /reflect Command

When triggered manually or at natural task completion checkpoints:

1. Review friction events from current session
2. Review any new error logs
3. Evaluate against friction threshold in `INTENT.md`
4. Generate proposals for anything that meets the threshold
5. Report to user: *"Reflected on [N] tasks. Generated [N] proposals. Use /review to see them."*

## /review Command

When triggered:

1. Open `IMPROVEMENT_QUEUE.md`
2. Surface all proposals with status `PENDING`
3. For each proposal, present with a recommended action
4. Accept user decision: approve / reject / modify
5. On approval: apply the change, update status, increment spec version
6. On rejection: update status with reason
7. On modify: user edits inline, agent applies modified version

**⚠️ INTENT.md proposals require explicit `APPROVE INTENT CHANGE` confirmation.**

## /status Command

Report current environment health: spec coverage, open proposals, recent activity, health flags.
