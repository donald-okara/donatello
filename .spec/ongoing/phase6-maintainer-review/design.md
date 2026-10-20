# Phase 6: Design

### Review the phase, not each task
**Decision:** the maintainer reviews a finished phase once, before archive; per-task checks stay with the `donatello:reviewer` agent.
**Why:** a phase is the unit the maintainer cares about, and per-task sign-off would stall `phase-run --all`.
**Tradeoff:** problems in early tasks surface only at the end of the phase.

### Record only
**Decision:** review comments are recorded in `review.md` and nothing more; no tasks, deferrals, or plan edits are created from them.
**Why:** the maintainer decides what a comment means; `/donatello:phase-discuss` already handles plan changes.
**Tradeoff:** acting on comments is a separate, manual step.

### Separate `review.md`, appended by round
**Decision:** reviews live in `<phase>/review.md`, one `## Review <n>` section per round, never rewritten.
**Why:** the walkthrough is Claude's record and gets rewritten in places; the maintainer's judgment stays separate and keeps its history.
**Tradeoff:** one more file per phase.

### Guided review in chat
**Decision:** `/donatello:phase-review` briefs the maintainer and takes the verdict and comments in chat; Claude writes the file.
**Why:** matches the other commands, and the brief saves the maintainer from assembling context across five files.
**Tradeoff:** a maintainer who prefers to write by hand edits `review.md` directly, outside the command.

### Soft archive gate
**Decision:** `/donatello:phase-archive` asks "archive anyway / stop" when there is no approval; it never blocks.
**Why:** keeps the maintainer in control, consistent with archive's handling of unticked tasks.
**Tradeoff:** an unreviewed phase can still be archived after one answer.
