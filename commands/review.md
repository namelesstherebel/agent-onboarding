# /review

Surface pending improvement proposals for human review and approval.

## Behavior

1. Read `IMPROVEMENT_QUEUE.md`
2. Filter for all proposals with status `PENDING`
3. If no pending proposals: report "No pending proposals. Environment is clean." and suggest `/reflect` if the user wants to generate proposals from recent work.
4. For each pending proposal, present:

```
PROPOSAL [ID] — [affected artifact]
Triggered by: [friction type or error]
Confidence: [HIGH/MEDIUM/LOW] | Human decision required: [YES/NO]

[Full proposal content]

Recommended action: [APPROVE / REJECT / MODIFY]
Your decision: _____
```

5. Process the user's decision:
   - **APPROVE** — Apply the proposed change to the affected artifact. Update proposal status to `APPROVED`. If a spec was changed, increment its version number and add a version history entry.
   - **REJECT** — Update status to `REJECTED`. Ask for a brief reason. Record the reason so the agent does not re-propose the same change.
   - **MODIFY** — Let the user edit the proposal inline. Apply the modified version. Status becomes `APPROVED (MODIFIED)`.

6. Move processed proposals to the "Processed Proposals" section of `IMPROVEMENT_QUEUE.md`.
7. Update the Queue Summary counts.

## INTENT.md Proposals

When any proposal targets `INTENT.md`, display this warning **before** showing the proposal:

```
⚠️  INTENT.MD CHANGE PROPOSED ⚠️

This proposal modifies the agent's core intent rules. Changes to INTENT.md
affect how ALL agents behave in this environment going forward.

Please read this carefully before approving.
This cannot be auto-applied. You must explicitly type APPROVE INTENT CHANGE to confirm.
```

Do not apply INTENT.md changes without the explicit `APPROVE INTENT CHANGE` confirmation.

## After Review

Report summary: "Reviewed [N] proposals. [N] approved, [N] rejected, [N] modified."
