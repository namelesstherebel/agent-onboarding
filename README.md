# agent-onboarding

```bash
claude plugin marketplace add namelesstherebel/agent-onboarding && claude plugin install agent-onboarding
```

Turn any repo into a self-improving agent environment.

## What It Does

This Claude Code plugin has two layers that work together:

**Onboarding workflow** — a 7-phase guided process that interviews you about your project and produces `CLAUDE.md`, `INTENT.md`, `PROJECT_BRIEF.md`, `SPEC_INVENTORY.md`, and a full `SPECS/` directory of agent-executable specifications. Works on greenfield projects and existing codebases — it reads your repo before asking any questions.

**Self-improving runtime** — installed into your repo during onboarding. After setup is complete, it tracks friction during agent tasks (gaps in specs, missing context, unclear intent, backtracking), logs errors, and generates structured improvement proposals into `IMPROVEMENT_QUEUE.md`. You review proposals with `/review`. Approved changes get merged back into the affected artifacts, incrementing their version. The environment gets better every time an agent runs.

## Commands

| Command | What it does |
|---------|-------------|
| `/onboard` | Start or resume the 7-phase onboarding workflow |
| `/review` | Surface pending improvement proposals for approval/rejection |
| `/reflect` | Manually trigger a friction review and proposal generation |
| `/status` | Report environment health: spec coverage, open proposals, last activity |

## Onboarding Phases

| Phase | Name | Produces |
|-------|------|----------|
| 1 | Project Discovery | `PROJECT_BRIEF.md` |
| 2 | Context Engineering | `CLAUDE.md`, `CONTEXT/` |
| 3 | Intent Engineering | `INTENT.md` |
| 4 | Specification Readiness | `SPEC_INVENTORY.md` |
| 5 | Environment Build | File structure, dependencies, runtime |
| 6 | Specification Writing | `SPECS/*.md` |
| 7 | Verify & Launch | Validation, handoff to runtime |

For existing repos, the workflow reads your codebase first and leads with what it found — it only asks about gaps.

## What Gets Installed in Your Repo

After onboarding, your repo will contain:

```
your-repo/
├── CLAUDE.md                  ← Agent context (stack, conventions, tools)
├── INTENT.md                  ← Agent intent (goals, trade-offs, escalation rules)
├── PROJECT_BRIEF.md           ← Project overview
├── SPEC_INVENTORY.md          ← Task inventory and spec queue
├── RUNTIME.md                 ← Self-improving runtime
├── IMPROVEMENT_QUEUE.md       ← Proposal queue
├── ONBOARDING_STATE.md        ← Onboarding progress tracker
├── CONTEXT/                   ← Reference docs the agent always loads
├── SPECS/                     ← Agent-executable specifications
│   └── [task-name].md
└── LOGS/
    ├── sessions/              ← Agent session logs
    └── errors/                ← Error logs
```

## How the Runtime Works

1. **Friction detection** — the agent tracks every moment it had to guess, backtrack, or make an assumption. When friction exceeds the threshold in `INTENT.md`, it generates a structured proposal targeting the root cause.

2. **Error logging** — unexpected errors get logged immediately and, if they reveal a structural gap, trigger a proposal.

3. **Human review** — proposals accumulate in `IMPROVEMENT_QUEUE.md`. Nothing changes until you run `/review` and approve. `INTENT.md` changes require explicit confirmation.

4. **Versioned specs** — every approved change increments the spec version with a traceable history of what changed and why.

## Installation

```bash
claude plugin marketplace add namelesstherebel/agent-onboarding && claude plugin install agent-onboarding
```

Then run `/onboard` in any repo to start.

## Signal System

During onboarding, control the flow:

| Signal | Action |
|--------|--------|
| `ready` / `next` | Advance to next phase |
| `skip` | Accept placeholders, advance |
| `pause` | Save progress, stop cleanly |
| `back` | Return to previous phase |

## License

MIT — Copyright (c) 2026 Stefan Kuczynski
