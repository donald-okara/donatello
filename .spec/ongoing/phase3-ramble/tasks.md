# Phase 3: Tasks

Each task ends with a commit. As in phase 2, checks in Tasks 1–6 are reviewer walk-throughs of the skill text; live runs are collected in Task 8, run interactively.

```
1. Notes template and starting a ramble
   └─> 2. Picker and resume
          └─> 3. Ramble mode
                 └─> 4. Wrap-up: Learnings, outcomes, close-out
                        └─> 5. Skills to the catalog
                               └─> 6. Phase integration and CLAUDE.md block
                                      ├─> 7. Docs and README
                                      └─> 8. Fixture verification (needs Donald)
```

---

### Task 1: Notes template and starting a ramble
**Serves:** R2, R3
**Files:** `skills/ramble/SKILL.md`, `skills/ramble/templates/notes.md`

- [x] Create `skills/ramble/templates/notes.md`: frontmatter `status: open`, `mode`, `branch`, `worktree`, `started`, `closed` (empty); headings `## Question`, `## Log`, `## Learnings`, `## Outcome`, with HTML-comment guidance (Log: terse timestamped one-liners, no code; Learnings: ≤ 10 lines, written at wrap-up)
- [x] Create `skills/ramble/SKILL.md` with frontmatter `name: ramble`, a one-line `description`, `argument-hint: "[topic | end]"`, `disable-model-invocation: true`, and a short intro listing the three entry points (no argument, topic, `end`)
- [x] Section "Start": slug rule and collision check from R2; ask worktree / in place; the exact git commands for each mode; clean-tree check with stash / worktree / stop; notes file created in the main checkout's `.spec/rambles/` from the template (comments removed); commit `docs: start ramble <slug>` on main in worktree mode only; notes-only fallback outside git
- [x] Check: the reviewer walks both modes and the non-git case through the text against R2 and R3
- [x] Commit: `feat: start rambles with /donatello:ramble`

### Task 2: Picker and resume
**Serves:** R1, R5
**Files:** `skills/ramble/SKILL.md`

- [x] Section "No argument": detect the current ramble (branch `ramble/*`, or a worktree path matching a notes file's `worktree`) and offer "continue" first; otherwise list open, then parked, then orphan branches (`git branch --list 'ramble/*'` with no notes file), then "new ramble"; offer "mark abandoned" for an open notes file whose branch is gone
- [x] Section "Resume": read only the frontmatter, Question, and the last 5 Log bullets; move into the worktree if there is one (or `git switch` in place, with the clean-tree check); restate the question and continue in ramble mode
- [x] Topic argument skips the picker and goes to "Start"
- [x] Check: the reviewer walks the picker for open, parked, orphan, and abandoned cases and the resume read limits against R1 and R5
- [x] Commit: `feat: add ramble picker and resume`

### Task 3: Ramble mode
**Serves:** R4
**Files:** `skills/ramble/SKILL.md`

- [x] Section "Ramble mode" with the rules from R4 as a short list: main session only (scout on request), teaching on, preferences relaxed and said once, main untouched except the notes, `wip:` checkpoints, `git add <paths>` never `-A`, notes never staged on the branch
- [x] Log rule: when to append, the one-line format (`HH:MM — <what was learned or tried>`), no code or output
- [x] Skill-building rule: suggest `.claude/skills/<id>/SKILL.md` so it loads and can be tested in the same ramble
- [x] Check: the reviewer confirms every R4 bullet is covered and the section stays under ~30 lines
- [x] Commit: `feat: define ramble mode`

### Task 4: Wrap-up — Learnings, outcomes, close-out
**Serves:** R6, R8, R9, NF2
**Files:** `skills/ramble/SKILL.md`

- [x] Section "End", step 1 Tidy: uncommitted changes → checkpoint or discard, asked
- [x] Step 2 Learnings: draft ≤ 10 lines from the log and `git log main..ramble/<slug> --oneline` (know now / didn't work and why / still open); show for edit
- [x] Step 3 is a placeholder line pointing to "Skills to the catalog" (filled in Task 5)
- [x] Step 4 Outcome: the five choices and their effects and statuses as in R8's table; throwaway shows `git log main..ramble/<slug> --oneline | wc -l` and asks before `git worktree remove` and `git branch -D`; merge uses `git merge --squash` and asks for the commit message; PR uses `gh pr create`; promote runs phase-new with the notes path
- [x] Step 5 Close-out in R9's order, ending with the branch action; on failure of the branch action, report it and leave the notes as committed
- [x] Check: the reviewer walks each outcome against R8 and the order against R9, including a failing merge
- [x] Commit: `feat: add ramble wrap-up and outcomes`

### Task 5: Skills to the catalog
**Serves:** R7
**Files:** `skills/ramble/SKILL.md`

- [x] Replace the Task 4 placeholder with "Skills to the catalog": detect `SKILL.md` folders new on the branch (`git diff --name-only main...HEAD` plus `git ls-files --others --exclude-standard`), show id / description / file count, ask add or skip for each
- [x] Add "draft a skill from these Learnings": write a `SKILL.md` with `name` and `description` frontmatter and a body from Learnings and log; show for edit; treat as found
- [x] Copy chosen folders to `$TMPDIR/donatello-ramble-<slug>/<id>/` before any branch action
- [x] In close-out step 2 (after returning to main), follow `add` in `../skills/SKILL.md` with each temp folder, then remove `$TMPDIR/donatello-ramble-<slug>/`; record `skill added: <id>` in Outcome
- [x] Check: the reviewer walks a throwaway ramble inside a project and one inside the Donatello repo, confirming the catalog change lands on main in both
- [x] Commit: `feat: send ramble skills to the catalog`

### Task 6: Phase integration and CLAUDE.md block
**Serves:** R5, R10, NF1
**Files:** `skills/phase-new/SKILL.md`, `skills/phase-run/SKILL.md`, `templates/claude-block.md`

- [x] phase-new step 1: read the Learnings of closed rambles per R10 (closed after the newest archive date, at most 5); when given a ramble notes path, read that file in full
- [x] phase-run: before picking a task, stop with "this is a ramble branch; end it with `/donatello:ramble end` or switch to main" when the branch is `ramble/*`
- [x] phase-run: the clean-tree check ignores changes under `.spec/rambles/` (worktree rambles keep appending to their notes in the main checkout), and the task commit stages everything except `.spec/rambles/` (`git add -A -- . ':!.spec/rambles'`)
- [x] `templates/claude-block.md`: add the ramble line and `/donatello:ramble` to Commands
- [x] Check: the block measures under 300 tokens (chars / 4); the reviewer confirms phase-new's read limits and phase-run's guard
- [x] Commit: `feat: connect rambles to phases`

### Task 7: Docs and README
**Serves:** R11
**Files:** `docs/SPECIFICATION.md`, `docs/ARCHITECTURE.md`, `README.md`

- [x] SPECIFICATION: workflow 6 `/donatello:ramble`; `.spec/rambles/` in the project tree; ramble mode under Key Behaviors
- [x] ARCHITECTURE: a `ramble` component (frontmatter, reads, writes, guardrails); this phase's lasting decisions; the edge cases from R11
- [x] README: the command in Commands and a short "Ramble" section (start, ramble mode, end and outcomes, skills to catalog); `skills/ramble/` in the layout
- [x] Check: `grep -n "ramble" README.md docs/*.md` covers the command, `.spec/rambles/`, and all five outcomes
- [x] Commit: `docs: document /donatello:ramble`

### Task 8: Fixture verification
**Serves:** R1–R10

- [ ] Worktree ramble in a scratch repo: start, two checkpoints, `/clear`, `/donatello:ramble` offers continue and resumes with the question restated
- [ ] In-place ramble: notes stay untracked on the branch and cross to main on `end`; a dirty tree at start offers stash / worktree / stop
- [ ] Picker with one open, one parked, one orphan branch, and one notes file whose branch was deleted
- [ ] Each outcome: throwaway (warns with commit count), keep (parked), merge (one squash commit), PR (skip if no remote), promote (phase-new cites the notes)
- [ ] A skill built at `.claude/skills/<id>/` on a throwaway ramble reaches the catalog in the clone; repeat inside the Donatello repo; "draft from Learnings" produces a reviewed skill
- [ ] `phase-run` on a ramble branch refuses; `phase-new` after a closed ramble mentions its Learnings
- [ ] Log anything unexpected in this phase's `walkthrough.md` and fix the skill text
- [ ] Commit: `fix: adjust ramble from fixture trial` (only if fixes were needed)
