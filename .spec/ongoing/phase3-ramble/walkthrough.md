# Phase 3: Walkthrough

## Current state

Tasks 1–7 are done: `/donatello:ramble` is written (start, picker, resume, ramble mode, end with five outcomes, skills to the catalog), wired into phase-new, phase-run, and the CLAUDE.md block (~274 tokens), and documented in SPECIFICATION, ARCHITECTURE, and README.
Everything so far was reviewed as text; nothing has run live.
Next: Task 8 with Donald (live fixture checks). Riskiest paths to try first: throwaway in worktree mode (worktree remove, then branch -D), in-place notes crossing to main, and a skill surviving a throwaway inside the Donatello repo. Task 6 gained a step: phase-run ignores `.spec/rambles/` in its clean check and task commits.

## How to verify

- `claude plugin validate .` passes
- Task 1: walk worktree, in-place, and non-git starts through the "Start" section of `skills/ramble/SKILL.md` against R2 and R3
- Task 7: `grep -n "ramble" README.md docs/*.md` shows the command, `.spec/rambles/`, and the outcomes; the block copies in README and ARCHITECTURE match `templates/claude-block.md`
- Task 6: block under 300 tokens (`sed -n '/donatello:start/,/donatello:end/p' templates/claude-block.md | wc -c` / 4); phase-new step 1 read limits and step 4 use of Learnings; phase-run's ramble guard and `.spec/rambles` pathspecs
- Task 5: walk a throwaway worktree ramble, an in-place one, one inside the Donatello repo, and a notes-only one; the skill reaches the clone's catalog on main each time
- Task 4: walk each outcome in "End" against R8 and the close-out order against R9 (worktree, in-place, notes-only, failing merge)
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

### Task 4: Wrap-up — Learnings, outcomes, close-out
- `skills/ramble/SKILL.md`: added "End": Tidy, Learnings, step 3 placeholder, Outcome (five choices, asked before any git change), Close-out in R9 order
- Main-side commands use `git -C <main checkout>`; close-out `cd`s to the main checkout before removing anything; worktree remove before `branch -D`; no `--force` unless asked
- PR uses `gh pr create --head ramble/<slug>` and records the URL in a follow-up `docs: record PR for ramble <slug>` commit; promote records `promoted:` after phase-new returns, and with throwaway deletes only afterwards
- Failed squash: reports `git status`; no `MERGE_HEAD`, so no `merge --abort`
- Clean checks exclude `.spec/rambles/` (in-place notes are untracked by design); notes-only skips Tidy and Outcome and runs no git
- Fix round: 8 review issues (untracked notes tripping the dirty check, branch deletion before worktree removal, PR head and URL, `-C` on merge, NF2 on promote + throwaway, keep's `closed` date, notes-only)

### Task 5: Skills to the catalog
- `skills/ramble/SKILL.md`: new "Skills to the catalog" (Find, Stage, Ask, Draft, Add); End step 3 asks and drafts; close-out step 2 runs `add` from `../skills/SKILL.md` per staged folder, then removes `$TMPDIR/donatello-ramble-<slug>/`
- Detection: `git diff --name-only --diff-filter=A main...HEAD` plus `git ls-files --others --exclude-standard`, `SKILL.md` paths only, `.spec/rambles/` excluded; notes-only rambles offer drafting only
- Find and Stage run first in Tidy, so a discard can't delete a skill before it's copied
- Inside the Donatello repo, a declined `skills add` commit leaves the catalog change uncommitted on main and is reported; merge asks to commit it first since the squash can collide on `catalog/` paths
- Fix round: 2 review issues (discard before detection; declined commit vs squash)
- Reviewer NOTE: abandoning `end` after staging leaves `$TMPDIR/donatello-ramble-<slug>/`; a later `end` overwrites it

### Task 6: Phase integration and CLAUDE.md block
- `skills/phase-new/SKILL.md`: step 1 reads Learnings of rambles closed after the newest archive date (max 5), or a promoted ramble's full notes; step 4 folds them into overview (goal / out of scope) and design (decisions), naming the ramble
- `skills/phase-run/SKILL.md`: stops on `ramble/*`; clean check `git status --porcelain -- . ':!.spec/rambles'`; commit `git add -A -- . ':!.spec/rambles'`
- `templates/claude-block.md`: ramble line with an explicit read bound and the notes location; `/donatello:ramble` in Commands
- Fix round (phase-new didn't use the Learnings; line over 30 tokens), then escalated to Sonnet: shortening the line kept dropping the read bound or the notes location. Stopped and asked Donald, who approved the reviewer's wording and relaxing NF1's per-line cap

### Task 7: Docs and README
- SPECIFICATION: workflow 6 (Ramble), `.spec/rambles/` and `skills/ramble/` in the trees, ramble mode under Key Behaviors, block bullets
- ARCHITECTURE: `ramble` component; phase-new and phase-run entries updated; block shape synced with the template; this phase's decisions; R11 edge cases plus squash conflict, missing worktree path, refused worktree removal
- README: command, a "Ramble" section, `.spec/rambles/` and `skills/ramble/` in the trees, block synced
- Reviewer NOTE: ARCHITECTURE doesn't list the five outcomes in one place (they're spread across guardrails, decisions, and edge cases)

## Deviations from spec

- NF1: the block's ramble line is ~50 tokens, not ≤ 30; the line has to state the notes location, the R5 read bound, and resume. NF1 now caps the whole block (< 300 tokens, ~273 measured) instead. Approved by Donald.
