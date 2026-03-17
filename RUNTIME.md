# Agent Runtime

**Installed by:** Agent Onboarding Workflow
**Version:** 1.2

This file activates the self-improving runtime for this repository. Its presence signals that onboarding is complete and autonomous agent operation is authorized. Read this file at the start of every session.

---

## What This Runtime Does

Two mechanisms that compound over time:

**Friction detection** — the agent tracks every moment it had to guess, backtrack, ask, or assume during a task. When friction accumulates past the threshold in `INTENT.md`, it generates a structured improvement proposal targeting the root cause.

**Error logging** — unexpected errors get logged immediately and generate proposals to prevent recurrence.

**Context hygiene** — the runtime monitors context file health. Proposals can target bloated CLAUDE.md, instructions that belong in scoped rules, or stale directives Claude already follows without being told.

Proposals accumulate in `IMPROVEMENT_QUEUE.md`. A human reviews them via `*review`. Approved proposals get merged. The environment improves. The next run is cleaner.

---

## Session Start Protocol

Execute before any task work:

1. Confirm `RUNTIME.md` is present (you're reading it — done)
2. Read `CLAUDE.md` — load conventions, stack, tools, domain knowledge
3. Read `INTENT.md` — load goals, trade-off hierarchy, friction threshold
4. Scan `IMPROVEMENT_QUEUE.md` — note pending proposals (don't interrupt task work)
5. Scan `LOGS/errors/` for recent errors relevant to current task
6. Confirm the relevant SPEC exists — if not, flag it and generate a spec stub proposal

If any file is missing, stop and report which ones before proceeding.

---

## Friction Tracking

Track friction silently during task execution. Surface at the end, never during.

| Friction Type | Example | What to Log |
|---|---|---|
| Spec gap | A spec step said "format the output" without specifying how | Step number, what was ambiguous, what assumption was made |
| Context gap | Agent didn't know the project's naming convention for a new file | What was missing, what was guessed |
| Intent gap | Two valid approaches existed, INTENT.md didn't define which to prefer | The decision point, what was chosen and why |
| Missing error handling | An error occurred that no spec step or CLAUDE.md entry covered | Error type, what the agent did, whether it was the right call |
| Backtrack | Agent started down a path, realized it was wrong, restarted | What caused the wrong turn |
| Low confidence completion | Task finished but agent isn't sure output is correct | Which quality bar was uncertain and why |
| Context bloat | A CLAUDE.md instruction was irrelevant to this task or redundant with tooling | Which line, why it didn't need to be there |

Friction log (internal, per session): Maintain a running list. At task completion, evaluate against the threshold in `INTENT.md`.

---

## Task Completion Protocol

Execute at the end of every task:

1. Evaluate accumulated friction against the threshold in `INTENT.md`
2. Generate improvement proposals for anything meeting the threshold — append to `IMPROVEMENT_QUEUE.md`
3. Log any errors encountered to `LOGS/errors/`
4. Count PENDING proposals in `IMPROVEMENT_QUEUE.md`
5. **Always surface a completion notice:**

---

**Task complete.**

> [One sentence summary of what was done.]
>
> [If pending proposals exist:]
> Review queue has [N] pending proposal(s).
> [N new this session. / No new — N from prior sessions still pending.]
> Run `*review` to approve, reject, or modify — nothing changes until you do.
>
> [If no pending proposals:]
> Improvement queue is clean. No proposals pending.

---

Do not skip this notice even if no new proposals were generated.

---

## Proposal Generation

When friction threshold is met, generate one or more proposals and append to `IMPROVEMENT_QUEUE.md`. One proposal per root cause — if three friction events trace to one missing spec step, that's one proposal.

### Context Hygiene Proposals

In addition to standard friction proposals, generate proposals when context files violate best practices:

- **CLAUDE.md over 200 lines** → propose pruning or moving content to `.claude/rules/`
- **Conditional instructions in CLAUDE.md** → propose moving to `.claude/rules/` with appropriate glob pattern
- **Style/formatting rules in context files** → propose moving to tooling config (eslint, prettier, etc.)
- **Instructions Claude follows without being told** → propose deletion (corrections over instructions)
- **Embedded content that should be a pointer** → propose replacing with file path reference
- **Stale instructions** → propose removal if behavior is already correct without them

These proposals use the same format and go through the same `*review` approval process.

### Proposal Format

```
---
## Proposal [YYYY-MM-DD-NNN]
**Status:** PENDING
**Generated:** [timestamp]
**Triggered by:** [FRICTION / ERROR / CONTEXT_HYGIENE]
**Task:** [spec or task being executed when this was triggered]
**Friction type:** [Spec gap / Context gap / Intent gap / Missing error handling / Backtrack / Low confidence / Context bloat]

### Root Cause
[What was missing, wrong, or unclear in the existing infrastructure]

### Affected Artifact
[SPECS/task-name.md / CLAUDE.md / .claude/rules/X.md / INTENT.md / CONTEXT/file.md / new file needed]

### Proposed Change
[Specific, concrete change — a diff, an addition, a new section. Not vague suggestions.]

### Why This Fixes It
[One or two sentences connecting the change to the root cause]

### Confidence
[HIGH / MEDIUM / LOW]

### Requires Human Decision
[YES / NO]

### Do Not Re-Propose If Rejected
[YES / NO]
---
```

---

## Error Logging

When an unexpected error occurs:

**Step 1** — Log immediately to `LOGS/errors/[YYYY-MM-DD]-[task-slug]-[N].md`:

```
# Error Log — [timestamp]
**Task:** [spec being executed]
**Spec version:** [version number]
**Step:** [which step in the spec process]

## What Happened
[Exact error — include error messages verbatim]

## What the Agent Was Trying to Do
[Context — what action triggered this]

## How the Agent Responded
[Stop / log and continue / escalate / assumption?]
[Was this response correct per INTENT.md?]

## Impact
[Task fail / degraded output / continued with uncertainty?]

## Structural Gap Identified
[Yes / No — missing spec step, wrong CLAUDE.md assumption, unclear intent rule?]
```

**Step 2** — Evaluate stop/escalate vs. log/continue per `INTENT.md` uncertainty protocol.

**Step 3** — If structural gap identified: generate an improvement proposal.

---

## Spec Version Control

Every spec update through an approved proposal increments its version:

```
## Version History
| Version | Date | Change | Triggered by |
|---------|------|--------|-------------|
| 1.0 | [date] | Initial spec | Onboarding |
| 1.1 | [date] | [what changed] | Proposal [ID] |
```

Every change is traceable to a root cause.

---

## Commands

**`*review`** — Surface all PENDING proposals. For each, present with recommended action. Process decisions:
- **APPROVE** → apply change, update status, increment spec version if applicable
- **REJECT** → update status with reason, agent won't re-propose
- **MODIFY** → user edits inline, agent applies modified version

For INTENT.md proposals, require explicit `APPROVE INTENT CHANGE` confirmation.

**`*reflect`** — Manually trigger reflection: review friction, review errors, evaluate threshold, generate proposals. Also audit context hygiene — check CLAUDE.md line count, look for conditional instructions that should be scoped, identify stale directives. Also run **context reconciliation** (see below).

**`*status`** — Report environment health: spec coverage, open proposals, recent activity, health flags, context hygiene metrics, last reconciliation date.

---

## Context Reconciliation

Context artifacts drift as the codebase evolves. Never summarize summaries — each layer regenerates from the source below it, and the codebase is always the lossless source of truth.

**When to reconcile:**
- During `*reflect` (every manual reflection includes a reconciliation pass)
- When friction events suggest the agent is working from stale context
- At natural project milestones (major feature complete, dependency upgrade, architecture change)

**Reconciliation steps:**
1. Compare `CLAUDE.md` Key Paths against actual file structure — flag paths that no longer exist or have moved
2. Compare `CLAUDE.md` Corrections against current code — flag corrections for patterns that have been fully migrated
3. Compare `CONTEXT/` files against the modules they describe — flag descriptions that no longer match
4. Check `.claude/rules/` glob patterns against current file structure — flag rules targeting files that don't exist
5. Generate proposals for any drift found — same format, same `*review` process

**Rule: never patch a summary from another summary.** If a CONTEXT/ file is stale, regenerate it by reading the actual code, not by editing the old description.

---

## Decision Log

`CONTEXT/decisions.md` is an append-only log preserving *why* architectural and convention decisions were made. The "why" is the context most vulnerable to loss when artifacts are updated or compressed.

**When to append:**
- When `*review` approves a proposal that changes conventions, architecture, or intent
- When a task reveals an undocumented decision that was clearly intentional
- When the user explains why something is the way it is

**Format:** `[YYYY-MM-DD] [Decision] — [Why]`

**Rules:**
- Never edit or delete entries — append only
- Never summarize the decision log — it stays verbatim
- Reconciliation may add entries (documenting discovered decisions) but never removes them

---

## What the Runtime Does NOT Do

- Auto-apply proposals. Human approval always required.
- Modify `INTENT.md` without explicit human confirmation.
- Delete or archive specs — only propose additions or changes.
- Generate proposals for one-off errors. Pattern or structural gap required.
- Interrupt task work to report friction. Surface after completion.
- Add instructions to CLAUDE.md that Claude would follow anyway.

---

## Runtime Health Indicators

**Healthy:**
- Spec versions incrementing — specs getting refined, not staying at 1.0
- Friction decreasing for mature specs
- Error logs less frequent in established task types
- Proposal confidence trending HIGH
- CLAUDE.md staying lean (under 200 lines)
- `.claude/rules/` growing instead of CLAUDE.md when new context is needed

**Unhealthy:**
- Same friction types repeating without proposals being approved
- Proposal queue growing without `*review`
- `INTENT.md` not updated despite environment evolving
- Error logs with no corresponding proposals
- CLAUDE.md growing past 200 lines
- Conditional instructions accumulating in CLAUDE.md instead of `.claude/rules/`
- Style/formatting rules in context files instead of tooling

---

`RUNTIME.md` is a system file. Modify only through the `*review` approval process.
