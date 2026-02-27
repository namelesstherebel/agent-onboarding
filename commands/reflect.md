# /reflect

Manually trigger a friction review and generate improvement proposals.

## Behavior

1. **Review friction events** from the current session. Look for:
   - Assumptions made because specs or context didn't cover a case
   - Clarifying questions that a better spec would have answered
   - Backtracking or redoing work due to initial misunderstanding
   - Errors not covered by existing error handling
   - Tasks completed with low confidence in output quality
   - Ambiguities with multiple valid interpretations

2. **Review error logs** — scan `LOGS/errors/` for new entries since the last reflection.

3. **Evaluate against threshold** — compare accumulated friction against the friction threshold defined in `INTENT.md`.

4. **Generate proposals** — for anything that meets the threshold, create structured improvement proposals and append them to `IMPROVEMENT_QUEUE.md`. Follow the proposal format defined in `RUNTIME.md`. One proposal per root cause — consolidate related friction events.

5. **Report to user:**

```
Reflected on [N] tasks since last review. Generated [N] proposals. Use /review to see them.
```

If no friction met the threshold:

```
Reflected on [N] tasks since last review. No proposals generated — friction below threshold.
```

## When to Use

- After completing a complex or multi-step agent task
- When you suspect the environment has gaps but haven't hit the automatic threshold
- Periodically as a health check on environment quality
- After a session with unusual errors or unexpected behavior

## Important

- Reflection does not auto-apply changes. It only generates proposals.
- All proposals go through `/review` before any artifacts are modified.
- Do not generate proposals for one-off errors — only structural gaps or patterns.
