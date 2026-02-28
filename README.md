# agent-onboarding

Turn any repo into a self-improving agent environment.

---

## Quick Install

**1. Install the plugin**

```bash
claude plugin marketplace add namelesstherebel/agent-onboarding && claude plugin install agent-onboarding
```

**2. Confirm it is installed**

```bash
claude plugin list
```

You should see `agent-onboarding` with status `installed`. If it is not listed, re-run the install command above.

**3. Confirm the commands are registered**

Open Claude Code and type `/` in the chat input. You should see `/onboard`, `/review`, `/reflect`, and `/status` in the list. If they do not appear, restart your Claude Code session and check again.

**4. Run onboarding**

Open Claude Code in the repo you want to set up and run `/onboard`. The agent handles the rest.

---

## What It Does

**Onboarding workflow** — a 7-phase guided process that interviews you about your project and produces `CLAUDE.md`, `INTENT.md`, `PROJECT_BRIEF.md`, `SPEC_INVENTORY.md`, and a full `SPECS/` directory. Works on greenfield projects and existing codebases — it reads your repo before asking any questions.

**Self-improving runtime** — installed into your repo during onboarding. It tracks friction during tasks (gaps in specs, missing context, unclear intent, backtracking), logs errors, and generates improvement proposals into `IMPROVEMENT_QUEUE.md`. You review proposals with `/review`. Approved changes get merged back into the affected artifacts, incrementing their version.

---

## Commands

| Command | What it does |
|---|---|
| `/onboard` | Start or resume the 7-phase onboarding workflow |
| `/review` | Surface pending improvement proposals for approval/rejection |
| `/reflect` | Manually trigger a friction review and proposal generation |
| `/status` | Report environment health: spec coverage, open proposals, last activity |

---

## Setup Guide

### Install the plugin

```bash
claude plugin marketplace add namelesstherebel/agent-onboarding && claude plugin install agent-onboarding
```

To reinstall after an update:

```bash
claude plugin marketplace remove agent-onboarding
claude plugin marketplace add namelesstherebel/agent-onboarding
claude plugin install agent-onboarding
```

### Verify installation

Run `claude plugin list` in your terminal and confirm `agent-onboarding` appears with status `installed`.

### Verify commands in Claude Code

Open Claude Code, type `/`, and confirm `/onboard`, `/review`, `/reflect`, and `/status` are listed. If they are missing, restart your Claude Code session and check again. If still missing after restart, reinstall the plugin.

### Run onboarding

Open Claude Code in your target repo and run `/onboard`. The agent reads your codebase first, then walks you through 7 phases.

Use these signals at any point to control the flow:

| Signal | Action |
|---|---|
| `ready` or `next` | Advance to the next phase |
| `skip` | Accept placeholders and advance |
| `pause` | Save progress and stop cleanly |
| `back` | Return to the previous phase |

After onboarding, your repo will contain:

```
your-repo/
├── CLAUDE.md             <- Agent context
├── INTENT.md             <- Agent intent and trade-off rules
├── PROJECT_BRIEF.md      <- Project overview
├── SPEC_INVENTORY.md     <- Task inventory and spec queue
├── RUNTIME.md            <- Self-improving runtime
├── IMPROVEMENT_QUEUE.md  <- Proposal queue
├── ONBOARDING_STATE.md   <- Onboarding progress tracker
├── CONTEXT/              <- Reference docs the agent always loads
└── SPECS/                <- Agent-executable specifications
```

### Verify the runtime is working

After onboarding, run any task in the repo. At the end of every task the agent surfaces a completion notice. If proposals were generated:

```
Task complete.

Review queue has N pending proposal(s). Run /review to approve, reject, or modify.
```

If the queue is clean:

```
Task complete.

Improvement queue is clean. No proposals pending.
```

If you are not seeing this notice after tasks, check that `RUNTIME.md` exists in your repo root. If it is missing, re-run `/onboard` to reinstall it.

### Review proposals

Run `/review` after your first task. For each proposal the agent shows what triggered it, which file it proposes to change, the exact change written out, and a confidence level. Respond with `APPROVE`, `REJECT`, or `MODIFY`. Nothing changes in your repo until you do.

---

## Onboarding Phases

| Phase | Name | Produces |
|---|---|---|
| 1 | Project Discovery | `PROJECT_BRIEF.md` |
| 2 | Context Engineering | `CLAUDE.md`, `CONTEXT/` |
| 3 | Intent Engineering | `INTENT.md` |
| 4 | Specification Readiness | `SPEC_INVENTORY.md` |
| 5 | Environment Build | File structure, dependencies, runtime |
| 6 | Specification Writing | `SPECS/*.md` |
| 7 | Verify and Launch | Validation, handoff to runtime |

For existing repos, the workflow reads your codebase first and leads with what it found — it only asks about gaps.

---

## License

MIT — Copyright (c) 2026 Stefan Kuczynski
