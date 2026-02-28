# Agent Onboarding Plugin

This plugin is installed via `claude plugin install agent-onboarding`. It provides a 7-phase onboarding workflow and a self-improving runtime for any repository.

## Star Commands

The following star commands are available once this plugin is installed. Type them in Claude Code to trigger the workflows.

### *onboard

Start or resume the 7-phase onboarding workflow for this repository.

Read the full workflow definition from this plugin's `commands/onboard.md`. Execute it exactly as described.

Behavior:
- If no `ONBOARDING_STATE.md` exists: start fresh from Phase 1 (or Pre-Phase analysis if the repo has existing content).
- If `ONBOARDING_STATE.md` exists with `Status: In Progress`: resume from the recorded phase without re-asking answered questions.
- If `ONBOARDING_STATE.md` exists with `Status: Complete`: report complete, offer `*status` or a targeted re-run.

### *review

Surface pending improvement proposals from `IMPROVEMENT_QUEUE.md` for human review and approval.

Read the full behavior definition from this plugin's `commands/review.md`. Execute it exactly as described.

Behavior:
- Read `IMPROVEMENT_QUEUE.md` and filter for `PENDING` proposals.
- Present each proposal with its trigger, affected artifact, confidence level, and recommended action.
- Process APPROVE / REJECT / MODIFY decisions. Nothing changes until the user decides.
- After review, report a summary of what was approved, rejected, and modified.

### *reflect

Manually trigger a friction review and generate improvement proposals from recent work.

Read the full behavior definition from this plugin's `commands/reflect.md`. Execute it exactly as described.

Behavior:
- Review recent task history for friction signals: gaps in specs, missing context, unclear intent, backtracking, repeated errors.
- Generate proposals and write them to `IMPROVEMENT_QUEUE.md`.
- Report how many proposals were generated and suggest `*review` to process them.

### *status

Report environment health: spec coverage, open proposals, and last activity.

Read the full behavior definition from this plugin's `commands/status.md`. Execute it exactly as described.

Behavior:
- Check for presence and completeness of: `CLAUDE.md`, `INTENT.md`, `PROJECT_BRIEF.md`, `SPEC_INVENTORY.md`, `RUNTIME.md`, `IMPROVEMENT_QUEUE.md`, `SPECS/`.
- Report spec coverage, count of pending proposals, and last recorded activity.
- Flag any missing or incomplete artifacts.

## Notes

- These commands work in any repository where this plugin is installed.
- The onboarding workflow reads your existing codebase before asking questions. It only asks about gaps.
- Nothing is written to your repo without your review. All proposals go through `*review` before any changes apply.
- The self-improving runtime (`RUNTIME.md`) is installed into your repo during onboarding. It runs automatically during tasks and generates proposals into `IMPROVEMENT_QUEUE.md`.
