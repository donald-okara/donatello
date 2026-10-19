# Phase 3: Walkthrough

## Current state

Task 1 is done: `skills/ramble/SKILL.md` (frontmatter, entry points, Start section) and `skills/ramble/templates/notes.md`.
Next: Task 2, the picker and resume.

## How to verify

- `claude plugin validate .` passes
- Task 1: walk worktree, in-place, and non-git starts through the "Start" section of `skills/ramble/SKILL.md` against R2 and R3

## Changes by task

### Task 1: Notes template and starting a ramble
- Added `skills/ramble/templates/notes.md` (frontmatter, Question / Log / Learnings / Outcome) and `skills/ramble/SKILL.md` (frontmatter, three entry points, Start with worktree / in-place / notes-only)
- Worktree is created before the start commit, so the branch doesn't carry the notes file; the start commit lands on main
- Added: in worktree mode, checks the main checkout is on main before the start commit (asks if not)
- Template frontmatter has no HTML comments (they'd break YAML); value guidance sits in a body comment
- Reviewer NOTE: the collision check looks only at today's `<date>-<slug>.md`; a glob on `*-<slug>.md` would catch older notes in the non-git case

## Deviations from spec

None
