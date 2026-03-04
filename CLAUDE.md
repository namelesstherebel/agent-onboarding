# Agent Onboarding Plugin

## Commands

`*onboard` — Start/resume the 7-phase onboarding workflow. Read `commands/onboard.md`, then execute.
`*review` — Surface pending improvement proposals. Read `commands/review.md`, then execute.
`*reflect` — Trigger friction review and generate proposals. Read `commands/reflect.md`, then execute.
`*status` — Report environment health. Read `commands/status.md`, then execute.

Do not ask for clarification on any of these — just execute.

## Onboarding State

- No `ONBOARDING_STATE.md` → start fresh (Pre-Phase for existing repos, Phase 1 for greenfield).
- `Status: In Progress` → resume from recorded phase. Don't re-ask answered questions.
- `Status: Complete` → report complete, suggest `*status`.

## Runtime Behavior

After onboarding installs `RUNTIME.md`, surface a completion notice after every task:

- Proposals exist → `Task complete. Review queue has N pending proposal(s). Run *review.`
- Queue clean → `Task complete. Improvement queue is clean.`

## Rules

- Never change the user's repo without approval.
- All proposals go through `*review` before applying.
- Confirm each phase output before advancing.
- Existing repos: read codebase first, lead with findings, ask only about gaps.

## Context Best Practices

These principles govern how this plugin generates and maintains context files:

- Keep CLAUDE.md under 200 lines. Universal rules only — everything else scoped.
- Use `.claude/rules/` with glob patterns for file-type or workflow-specific instructions.
- Never use context files as a linter — formatting belongs in tooling.
- Corrections over instructions: prioritize things Claude gets wrong in your codebase.
- Pointers over content: file paths and search directives, not reproduced content.
- One directive per line. If it takes a paragraph, it's not specific enough.
- Audit periodically: if Claude already does it correctly, delete the instruction.
- Progressive disclosure: surface information on demand, don't preload everything.
- Every line pays rent from the context window. No rent, no line.
