# agent-onboarding

Turn any repo into a self-improving agent environment.

---

## Quick Install

**1. Install the plugin** — run this in your terminal:

```bash
claude plugin marketplace add namelesstherebel/agent-onboarding && claude plugin install agent-onboarding
```

**2. Confirm it's installed** — run:

```bash
claude plugin list
```

You should see `agent-onboarding` with status `installed`. If it's not listed, re-run the install command above.

**3. Confirm the commands are registered** — open Claude Code and type `/` in the chat input. You should see `/onboard`, `/review`, `/reflect`, and `/status` in the list. If they don't appear, restart your Claude Code session (close and reopen the window), then check again.

**4. Run onboarding** — navigate to the repo you want to set up, open Claude Code in that directory, and type:

```
/onboard
```

That's it. The agent handles the rest.

---

## What It Does

This Claude Code plugin has two layers that work together:

**Onboarding workflow** — a 7-phase guided process that interviews you about your project and produces `CLAUDE.md`, `INTENT.md`, `PROJECT_BRIEF.md`, `SPEC_INVENTORY.md`, and a full `SPECS/` directory of agent-executable specifications. Works on greenfield projects and existing codebases — it reads your repo before asking any questions.

**Self-improving runtime** — installed into your repo during onboarding. After setup is complete, it tracks friction during agent tasks (gaps in specs, missing context, unclear intent, backtracking), logs errors, and generates structured improvement proposals into `IMPROVEMENT_QUEUE.md`. You review proposals with `/review`. Approved changes get merged back into the affected artifacts, incrementing their version. The environment gets better every time an agent runs.

---

## Commands

| Command | What it does |
|---|---|
| `/onboard` | Start or resume the 7-phase onboarding workflow |
| `/review` | Surface pending improvement proposals for approval/rejection |
| `/reflect` | Manually trigger a friction review and proposal generation |
| `/status` | Report environment health: spec coverage, open proposals, last activity |

---

## Full Setup and Verification Guide

### Install the plugin

```bash
claude plugin marketplace add namelesstherebel/agent-onboarding && claude plugin install agent-onboarding
```

If you've installed it before and need to reinstall (e.g. after an update):

```bash
claude plugin marketplace remove agent-onboarding
claude plugin marketplace add namelesstherebel/agent-onboarding
claude plugin install agent-onboarding
```

### Confirm the plugin is installed

```bash
claude plugin list
# Expected output includes:
# agent-onboarding   1.0.0   installed
```

If `agent-onboarding` doesn't appear, re-run the install command.

### Confirm the commands are registered in Claude Code

Open Claude Code and type `/` in the chat input. You should see:

- `/onboard`
- - `/review`
  - - `/reflect`
    - - `/status`
     
      - If the commands don't appear after installation:
     
      - 1. Restart your Claude Code session (close and reopen the window)
        2. 2. Type `/` again and check the list
           3. 3. If still missing, run `claude plugin list` in your terminal to confirm the plugin is installed, then reinstall
             
              4. ### Run onboarding in a repo
             
              5. Open Claude Code in the repo you want to set up and run:
             
              6. ```
                 /onboard
                 ```

                 The agent reads your codebase first, then walks you through 7 phases. Control the flow at any time:

                 - Type `pause` to save progress and stop cleanly
                 - - Type `skip` to accept placeholder values for the current phase and move on
                   - - Type `back` to return to the previous phase
                    
                     - What a successful onboarding produces:
                    
                     - ```
                       your-repo/
                       ├── CLAUDE.md             <- Agent context (stack, conventions, tools)
                       ├── INTENT.md             <- Agent intent (goals, trade-offs, escalation rules)
                       ├── PROJECT_BRIEF.md      <- Project overview
                       ├── SPEC_INVENTORY.md     <- Task inventory and spec queue
                       ├── RUNTIME.md            <- Self-improving runtime (installed by Phase 5)
                       ├── IMPROVEMENT_QUEUE.md  <- Proposal queue
                       ├── ONBOARDING_STATE.md   <- Onboarding progress tracker
                       ├── CONTEXT/              <- Reference docs the agent always loads
                       ├── SPECS/                <- Agent-executable specifications
                       │   └── [task-name].md
                       └── LOGS/
                           ├── sessions/         <- Agent session logs
                           └── errors/           <- Error logs
                       ```

                       ### Confirm the runtime is working

                       After onboarding, run any task in the repo. At the end of the task, the agent must surface a completion notice. If there are pending proposals:

                       ```
                       Task complete.
                       [Summary of what was done.]

                       📋 Review queue has 2 pending proposal(s). 2 new proposal(s) were added this session.
                       Run /review to approve, reject, or modify — nothing changes until you do.
                       ```

                       If the queue is clean:

                       ```
                       Task complete.
                       [Summary of what was done.]

                       ✅ Improvement queue is clean. No proposals pending.
                       ```

                       If you are not seeing this notice after tasks, check that `RUNTIME.md` exists in your repo root. If it is missing, re-run `/onboard` — Phase 5 (Environment Build) installs it.

                       ### Review your first proposals

                       Run `/review` after your first task. Each proposal shows:

                       - What triggered it (friction type or error)
                       - - Which artifact it proposes to change
                         - - The exact change, written out
                           - - Confidence level and whether it needs a judgment call from you
                            
                             - Respond with `APPROVE`, `REJECT`, or `MODIFY` for each one. Nothing changes in your repo until you approve.
                            
                             - ---

                             ## How the Runtime Works

                             1. **Friction detection** — the agent tracks every moment it had to guess, backtrack, or make an assumption. When friction exceeds the threshold in `INTENT.md`, it generates a structured proposal targeting the root cause.
                            
                             2. 2. **Error logging** — unexpected errors get logged immediately and, if they reveal a structural gap, trigger a proposal.
                               
                                3. 3. **Human review** — proposals accumulate in `IMPROVEMENT_QUEUE.md`. Nothing changes until you run `/review` and approve. `INTENT.md` changes require explicit confirmation.
                                  
                                   4. 4. **Versioned specs** — every approved change increments the spec version with a traceable history of what changed and why.
                                     
                                      5. ---
                                     
                                      6. ## Onboarding Phases
                                     
                                      7. | Phase | Name | Produces |
                                      8. |---|---|---|
                                      9. | 1 | Project Discovery | `PROJECT_BRIEF.md` |
                                      10. | 2 | Context Engineering | `CLAUDE.md`, `CONTEXT/` |
                                      11. | 3 | Intent Engineering | `INTENT.md` |
                                      12. | 4 | Specification Readiness | `SPEC_INVENTORY.md` |
                                      13. | 5 | Environment Build | File structure, dependencies, runtime |
                                      14. | 6 | Specification Writing | `SPECS/*.md` |
                                      15. | 7 | Verify & Launch | Validation, handoff to runtime |
                                     
                                      16. For existing repos, the workflow reads your codebase first and leads with what it found — it only asks about gaps.
                                     
                                      17. ---
                                     
                                      18. ## Signal System
                                     
                                      19. During onboarding, control the flow:
                                     
                                      20. | Signal | Action |
                                      21. |---|---|
                                      22. | `ready` / `next` | Advance to next phase |
                                      23. | `skip` | Accept placeholders, advance |
                                      24. | `pause` | Save progress, stop cleanly |
                                      25. | `back` | Return to previous phase |
                                     
                                      26. ---
                                     
                                      27. ## License
                                     
                                      28. MIT — Copyright (c) 2026 Stefan Kuczynski
