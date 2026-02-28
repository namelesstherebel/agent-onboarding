# Agent Onboarding Plugin

## Commands

When the user types `*onboard`, immediately start the 7-phase onboarding workflow. Do not ask for clarification. Read `commands/onboard.md` from this plugin for the full workflow, then execute it.

When the user types `*review`, immediately surface pending improvement proposals from `IMPROVEMENT_QUEUE.md`. Do not ask for clarification. Read `commands/review.md` from this plugin for the full behavior, then execute it.

When the user types `*reflect`, immediately trigger a friction review and generate improvement proposals. Do not ask for clarification. Read `commands/reflect.md` from this plugin for the full behavior, then execute it.

When the user types `*status`, immediately report environment health. Do not ask for clarification. Read `commands/status.md` from this plugin for the full behavior, then execute it.

## What This Plugin Does

This plugin sets up a self-improving agent environment in any repository. It provides:

- A 7-phase onboarding workflow that produces `CLAUDE.md`, `INTENT.md`, `PROJECT_BRIEF.md`, `SPEC_INVENTORY.md`, and a full `SPECS/` directory.
- A self-improving runtime (`RUNTIME.md`) that tracks friction during tasks, logs errors, and generates structured improvement proposals into `IMPROVEMENT_QUEUE.md`.
- A review workflow where the user approves, rejects, or modifies proposals before anything changes.

## Onboarding State

When running `*onboard`:
- If no `ONBOARDING_STATE.md` exists: start fresh from Phase 1, or run Pre-Phase repo analysis first if the repo has existing content.
- If `ONBOARDING_STATE.md` exists with `Status: In Progress`: resume from the recorded phase. Do not re-ask answered questions.
- If `ONBOARDING_STATE.md` exists with `Status: Complete`: report that onboarding is complete and suggest `*status`.

## Runtime Behavior

After onboarding installs `RUNTIME.md` into the user's repo, the runtime runs automatically during every task. At the end of each task, surface a completion notice:

If proposals were generated:
```
Task complete. Review queue has N pending proposal(s). Run *review to approve, reject, or modify.
```

If the queue is clean:
```
Task complete. Improvement queue is clean. No proposals pending.
```

## Important Rules

- Never make changes to the user's repo without their approval.
- All proposals go through `*review` before any changes apply.
- During onboarding, confirm each phase output with the user before advancing.
- For existing repos: read the codebase before asking questions. Lead with what you found, ask only about gaps.
