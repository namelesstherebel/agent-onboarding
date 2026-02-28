# *status

Report the current health of the agent environment.

## Behavior

Read the environment artifacts and generate a health report:

```markdown
# Environment Status — [Project Name]
**Last updated:** [date]

## Spec Coverage
- Total recurring tasks identified: [N]
- Specs written: [N] / [N]
- Specs with version > 1.0: [N] (improved since initial write)

## Improvement Queue
- Open proposals: [N]
- Approved this month: [N]
- Rejected this month: [N]

## Recent Activity
- Last agent task: [date + task name]
- Last proposal generated: [date]
- Last spec updated: [spec name + date]

## Health Flags
- [Any specs that haven't been updated despite repeated friction]
- [Any error patterns that haven't been addressed]
- [Any INTENT.md proposals pending > 7 days]
```

## Data Sources

- `SPEC_INVENTORY.md` — for task count and spec coverage
- `SPECS/*.md` — for version numbers and last-updated dates
- `IMPROVEMENT_QUEUE.md` — for proposal counts and statuses
- `LOGS/sessions/` — for last agent task activity
- `LOGS/errors/` — for error patterns
- `INTENT.md` — for friction threshold reference

## Health Flags

Flag these conditions when detected:

- **Stale specs** — specs at version 1.0 that have had friction logged against them
- **Unaddressed errors** — error patterns in `LOGS/errors/` with no corresponding proposal
- **Queue buildup** — more than 5 pending proposals without a `*review` run
- **Intent drift** — `INTENT.md` proposals pending for more than 7 days
- **Missing specs** — tasks in `SPEC_INVENTORY.md` that still have no spec written

## If Environment Is Not Onboarded

If `RUNTIME.md` does not exist, report:

```
Environment not yet onboarded. Use *onboard to start the setup workflow.
```

If `ONBOARDING_STATE.md` shows in-progress, report current phase and suggest resuming with `*onboard`.
