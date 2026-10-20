# Phase 6: Walkthrough

## Current state

Tasks 1–2 done: `/donatello:phase-review` is complete (load, brief, verdict, record, commit) with `skills/phase-new/templates/review.md`.
Next: Task 3 (archive soft gate, phase-run suggestion, block).
Open: none.

## How to verify

- Task 1: walk no phase, two phases, a ramble branch, an unticked task, and an earlier round through sections 1–2 of `skills/phase-review/SKILL.md` against R1 and R2
- Task 2: walk an approved round with no comments, a partial changes-requested round with three comments, and a second round through sections 3–5 against R3–R5

## Changes by task

### Task 1: phase-review — load and brief
- Added `skills/phase-review/SKILL.md` with frontmatter, intro, "1. Load" (worded like phase-archive step 1, ramble stop matching phase-run), and "2. Brief" (four parts per R2, git commands, outside-git skip, no source/diffs/agents rule)
- Sections 3–5 left for Task 2
- NOTE: phase-start command (copied from phase-archive) doesn't say what to do if `tasks.md` isn't committed or yields several hashes; same as archive, not blocking

### Task 2: phase-review — verdict, record, commit
- `skills/phase-review/SKILL.md`: added "3. Verdict and comments", "4. Record", "5. Wrap up" per R3–R5
- Added `skills/phase-new/templates/review.md` (heading plus a comment on the round format)
- Fix round: section 4 now replaces `<N>` with the phase number from the directory name; the commit message uses the same number

## Deviations from spec

None.
