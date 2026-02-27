# /onboard

Start or resume the agent onboarding workflow for this repository.

## Behavior

When invoked, determine the current state:

1. **No `ONBOARDING_STATE.md` exists** — Start fresh from Pre-Phase (Existing Repo Analysis) or Phase 1 (Project Discovery) depending on whether the repo has existing content.

2. **`ONBOARDING_STATE.md` exists with `Status: In Progress`** — Resume from the recorded phase and next action. Do not re-ask questions that have already been answered.

3. **`ONBOARDING_STATE.md` exists with `Status: Complete`** — Report that onboarding is already complete. Suggest `/status` to check environment health or offer to re-run specific phases if the user wants to update artifacts.

## Workflow

Execute the full onboarding skill defined in `skills/agent-onboarding/SKILL.md`:

- **Phase 1** — Project Discovery → produces `PROJECT_BRIEF.md`
- **Phase 2** — Context Engineering → produces `CLAUDE.md`, `CONTEXT/`, optionally `ENVIRONMENT.md`
- **Phase 3** — Intent Engineering → produces `INTENT.md`
- **Phase 4** — Specification Readiness → produces `SPEC_INVENTORY.md`
- **Phase 5** — Environment Build → installs file structure, dependencies, runtime
- **Phase 6** — Specification Writing → produces `SPECS/*.md`
- **Phase 7** — Verify & Launch → validates everything, marks onboarding complete

## Signals

The user can control flow with these signals at any point:

| Signal | Action |
|--------|--------|
| `ready` / `next` | Advance to next phase |
| `skip` | Accept placeholders for current phase, advance |
| `pause` | Save state to `ONBOARDING_STATE.md`, stop cleanly |
| `back` | Return to previous phase |

## Important

- For existing repos: **read the codebase before asking questions**. Lead with what you found, ask only about gaps.
- For greenfield repos: go straight to Phase 1 questions.
- Confirm each phase's output with the user before advancing.
- Write `ONBOARDING_STATE.md` at the end of every session, whether complete or paused.
