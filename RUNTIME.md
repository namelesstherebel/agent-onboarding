# Agent Runtime

**Installed by:** Agent Onboarding Workflow
**Version:** 1.0

This file activates the self-improving runtime for this repository. Its presence signals that onboarding is complete and autonomous agent operation is authorized. Read this file at the start of every session.

---

## What This Runtime Does

This runtime turns a static agent environment into one that gets better every time an agent runs. It does this through two mechanisms:

**Friction detection** — the agent tracks every moment it had to guess, backtrack, ask a question, or make an assumption during a task. When friction accumulates past the threshold defined in `INTENT.md`, it generates a structured improvement proposal. The proposal targets whatever caused the friction — a spec step, a missing context entry, an unclear intent rule, a missing script.

**Error logging** — when the agent encounters an error not covered by existing infrastructure, it logs it immediately and generates a proposal to prevent it from happening again.

Proposals accumulate in `IMPROVEMENT_QUEUE.md`. A human reviews them on their own schedule using `/review`. Approved proposals get merged into the relevant artifact. The environment improves. The next agent run is cleaner.

---

## Session Start Protocol

Execute this at the start of every session, before any task work:

1. Confirm `RUNTIME.md` is present (you're reading it — ✓)
2. Read `CLAUDE.md` — load conventions, stack, tools, domain knowledge
3. Read `INTENT.md` — load goals, trade-off hierarchy, friction threshold, uncertainty protocol
4. Scan `IMPROVEMENT_QUEUE.md` — note pending proposals (do not interrupt task work for them)
5. Scan `LOGS/errors/` for recent errors relevant to current task
6. Confirm the relevant SPEC exists before starting — if not, flag it and generate a spec stub proposal

If any of these files are missing, stop and report which ones before proceeding.

---

## Friction Tracking

Track friction silently during task execution. Do not interrupt the task to report it — log internally and surface at the end.

**What counts as friction:**

| Friction Type | Example | What to Log |
|---------------|---------|-------------|
| Spec gap | A spec step said "format the output" without specifying how | Step number, what was ambiguous, what assumption was made |
| Context gap | Agent didn't know the project's naming convention for a new file | What was missing, what was guessed |
| Intent gap | Two valid approaches existed, `INTENT.md` didn't define which to prefer | The decision point, what was chosen and why |
| Missing error handling | An error occurred that no spec step or `CLAUDE.md` entry covered | Error type, what the agent did, whether it was the right call |
| Backtrack | Agent started down a path, realized it was wrong, restarted | What caused the wrong turn |
| Low confidence completion | Task finished but agent isn't sure output is correct | Which quality bar was uncertain and why |

**Friction log (internal, per session):** Maintain a running list during the session. At task completion, evaluate against the threshold in `INTENT.md`.

---

## Proposal Generation

When friction threshold is met, generate one or more improvement proposals and append to `IMPROVEMENT_QUEUE.md`.

**One proposal per root cause.** If three friction events all trace back to one missing spec step, that's one proposal — not three.

**Proposal format:**

```markdown
---

## Proposal [YYYY-MM-DD-NNN]

**Status:** PENDING
**Generated:** [timestamp]
**Triggered by:** [FRICTION / ERROR]
**Task:** [spec or task being executed when this was triggered]
**Friction type:** [Spec gap / Context gap / Intent gap / Missing error handling / Backtrack / Low confidence]

### Root Cause
[What was missing, wrong, or unclear in the existing infrastructure]

### Affected Artifact
[SPECS/task-name.md / CLAUDE.md / INTENT.md / CONTEXT/file.md / new file needed]

### Proposed Change
[Specific, concrete change — a diff, an addition, a new section. Not vague suggestions. Write it as if you're writing the actual artifact content.]

### Why This Fixes It
[One or two sentences connecting the change to the root cause]

### Confidence
[HIGH — clear fix with no judgment call needed]
[MEDIUM — likely correct but a human should verify]
[LOW — uncertain, human judgment required]

### Requires Human Decision
[YES — involves trade-offs, values, or INTENT.md changes]
[NO — clear technical fix]

### Do Not Re-Propose If Rejected
[YES — if rejected, don't surface this again]
[NO — reject this time, but worth reconsidering after more data]

---
```

---

## Error Logging

When an unexpected error occurs during a task:

**Step 1** — Log immediately to `LOGS/errors/[YYYY-MM-DD]-[task-slug]-[N].md`:

```markdown
# Error Log — [timestamp]

**Task:** [spec being executed]
**Spec version:** [version number]
**Step:** [which step in the spec process]

## What Happened
[Exact error or unexpected behavior — include error messages verbatim]

## What the Agent Was Trying to Do
[Context — what action triggered this]

## How the Agent Responded
[Did it stop / log and continue / escalate / make an assumption?]
[Was this response correct per INTENT.md? If uncertain, say so.]

## Impact
[Did the task fail / produce degraded output / continue with uncertainty?]

## Structural Gap Identified
[Yes / No — does this error indicate a missing spec step, wrong assumption in CLAUDE.md, unclear intent rule, or missing script?]
```

**Step 2** — Evaluate whether to stop and escalate or log and continue, per `INTENT.md` uncertainty protocol.

**Step 3** — If structural gap identified: generate an improvement proposal targeting the gap.

---

## Spec Version Control

Every time a spec is updated through an approved proposal, increment its version and add to its version history table:

```markdown
## Version History
| Version | Date | Change | Triggered by |
|---------|------|--------|-------------|
| 1.0 | [date] | Initial spec | Onboarding |
| 1.1 | [date] | [what changed] | Proposal [ID] |
| 1.2 | [date] | [what changed] | Proposal [ID] |
```

This creates a traceable record of why the spec evolved — every change has a root cause.

---

## Commands

### `/review`

Surface all `PENDING` proposals from `IMPROVEMENT_QUEUE.md`.

For each proposal, present in this format:

```
PROPOSAL [ID] — [affected artifact]
Triggered by: [friction type or error]
Confidence: [HIGH/MEDIUM/LOW] | Human decision required: [YES/NO]

[Full proposal content]

Recommended action: [APPROVE / REJECT / MODIFY]
Your decision: _____
```

Process decisions:
- **APPROVE** → apply change to artifact, update proposal status to `APPROVED`, update spec version if applicable
- **REJECT** → update status to `REJECTED`, prompt for reason, record it so agent doesn't re-propose
- **MODIFY** → user edits proposal inline, agent applies modified version, status = `APPROVED (MODIFIED)`

For any proposal touching `INTENT.md`, show this warning before presenting the proposal:

```
⚠️  INTENT.MD CHANGE PROPOSED ⚠️

This proposal modifies core agent intent rules. Intent changes affect how
ALL agents behave in this environment going forward. This cannot be auto-applied.

Read the proposal carefully. To approve, type: APPROVE INTENT CHANGE
```

### `/reflect`

Manually trigger a reflection cycle:

1. Review friction events from the current session
2. Review new error logs since last reflection
3. Generate proposals for anything meeting the `INTENT.md` threshold
4. Report: *"Reflected on [N] sessions since last review. Generated [N] proposals."*

### `/status`

Report current environment health (see `SKILL.md` for full format).

---

## What the Runtime Does NOT Do

- It does **not** auto-apply proposals. Human approval is always required.
- It does **not** modify `INTENT.md` without explicit human confirmation.
- It does **not** delete or archive specs — only proposes additions or changes.
- It does **not** generate proposals for one-off errors. A pattern must exist or a structural gap must be clear.
- It does **not** interrupt task work to report friction. Surface it after completion.

---

## Runtime Health Indicators

**A healthy runtime looks like this over time:**
- Spec versions incrementing steadily — specs getting refined, not staying at 1.0
- Friction events decreasing for mature specs — the same spec produces less friction over time
- Error logs becoming less frequent in established task types
- Proposal confidence trending HIGH — the system knows itself well enough to propose clearly

**An unhealthy runtime looks like this:**
- Same friction types repeating across multiple sessions without proposals being approved
- Proposal queue growing without `/review` being run
- `INTENT.md` not updated in months despite the environment evolving significantly
- Error logs with no corresponding proposals

If health indicators are poor, the `/status` command will flag them.

---

*`RUNTIME.md` is a system file. Modify only through the `/review` approval process. Direct edits bypass the version history and break proposal traceability.*
