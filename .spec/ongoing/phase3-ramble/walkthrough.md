# Phase 3: Walkthrough

## Current state

Tasks 1–3 are done: `skills/ramble/SKILL.md` has frontmatter, entry points, Start, No argument (picker), Resume, and Ramble mode; `skills/ramble/templates/notes.md` exists.
Next: Task 4, wrap-up. Task 6 gained a step: phase-run ignores `.spec/rambles/` in its clean check and task commits.

## How to verify

- `claude plugin validate .` passes
- Task 1: walk worktree, in-place, and non-git starts through the "Start" section of `skills/ramble/SKILL.md` against R2 and R3
- Task 3: every R4 bullet appears in "Ramble mode" (under ~30 lines)
- Task 2: walk the picker (current, open, parked, orphan, abandoned, new; `end` outside a ramble) and Resume's read limit against R1 and R5

## Changes by task

### Task 1: Notes template and starting a ramble
- Added `skills/ramble/templates/notes.md` (frontmatter, Question / Log / Learnings / Outcome) and `skills/ramble/SKILL.md` (frontmatter, three entry points, Start with worktree / in-place / notes-only)
- Worktree is created before the start commit, so the branch doesn't carry the notes file; the start commit lands on main
- Added: in worktree mode, checks the main checkout is on main before the start commit (asks if not)
- Template frontmatter has no HTML comments (they'd break YAML); value guidance sits in a body comment
- Reviewer NOTE: the collision check looks only at today's `<date>-<slug>.md`; a glob on `*-<slug>.md` would catch older notes in the non-git case

### Task 2: Picker and resume
- `skills/ramble/SKILL.md`: added "No argument" (current-ramble detection, picker groups, `end` outside a ramble lists only open and parked) and "Resume" (frontmatter, Question, last 5 Log bullets; worktree or in-place switch with clean-tree check)
- Notes commits for resume (parked → open), abandon, and orphan branches run with `git -C <main checkout>` on main and only when the notes file is tracked; in-place notes stay untracked until `end`
- A `ramble/*` branch with no notes file goes through the orphan path; a missing worktree path offers to recreate it
- Fix round: three review issues, all about commits landing on the wrong branch or committing in-place notes early
- Reviewer NOTE: after an in-place `git switch`, a tracked-notes commit takes the "main isn't checked out, ask first" path; safe and rare

### Task 3: Ramble mode
- `skills/ramble/SKILL.md`: added "Ramble mode" (11 lines): main session only, teaching on, preferences relaxed and said once, main untouched except notes, `wip:` checkpoints with `git add <paths>`, log line format, skills at `.claude/skills/<id>/`
- Found: in worktree mode, log appends leave the main checkout's notes file modified, which would trip phase-run's clean-tree check and land in its `git add -A`. Added a step to Task 6 to exclude `.spec/rambles/` there

## Deviations from spec

None
