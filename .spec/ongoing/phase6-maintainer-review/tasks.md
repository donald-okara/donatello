# Phase 6: Tasks

Each task ends with a commit. As in phases 3–5, checks in Tasks 1–4 are reviewer walk-throughs of the skill text; live runs are in Task 5, run interactively.

```
1. phase-review: load, brief
   └─> 2. phase-review: verdict, record, commit
          └─> 3. Integration: archive gate, phase-run, block
                 ├─> 4. Docs and README
                 └─> 5. Fixture verification (needs Donald)
```

---

### Task 1: phase-review — load and brief
**Serves:** R1, R2
**Files:** `skills/phase-review/SKILL.md`

- [x] Frontmatter: `name: phase-review`, a one-line `description` ("Record a maintainer's review of a finished phase: brief, verdict, comments, appended to review.md"), `argument-hint: "[phase]"`, `disable-model-invocation: true`; a short intro saying the review is a record and nothing is acted on
- [x] Section "1. Load": phase selection and ramble-branch stop per R1, worded like `skills/phase-archive/SKILL.md` step 1; unticked tasks → list and ask "review anyway / stop", remember partial
- [x] Section "2. Brief": the parts from R2, in that order, about 15 lines; the git commands for the commit list; skip the commit list outside git; show the latest earlier round if `review.md` exists; one rule line: no source, no diffs, no agents
- [x] Check: the reviewer walks no phase, two phases, a ramble branch, an unticked task, and a phase with an earlier round against R1 and R2
- [x] Commit: `feat: add /donatello:phase-review`

### Task 2: phase-review — verdict, record, commit
**Serves:** R3, R4, R5, NF2
**Files:** `skills/phase-review/SKILL.md`, `skills/phase-new/templates/review.md`

- [x] Template `skills/phase-new/templates/review.md`: `# Phase <N>: Review`, an HTML comment describing the round format from R4 and that rounds are appended, never edited
- [x] Section "3. Verdict and comments" per R3: the two verdicts, comments in one exchange, maintainer's words only
- [x] Section "4. Record" per R4: create from template if missing (path relative to this skill: `../phase-new/templates/review.md`), round number, `date +%F`, `git config user.name` (ask if unset), `(partial)`, `- No comments.`; show the round and write only after approval
- [x] Section "5. Wrap up" per R5: commit offer with pathspec limited to the phase dir; next-step suggestion by verdict; state that no tasks or plan files change
- [x] Check: the reviewer walks an approved round with no comments, a partial changes-requested round with three comments, and a second round, against R3–R5
- [x] Commit: `feat: record verdict and comments in /donatello:phase-review`

### Task 3: Integration
**Serves:** R6, R7
**Files:** `skills/phase-archive/SKILL.md`, `skills/phase-run/SKILL.md`, `templates/claude-block.md`

- [ ] `phase-archive`: new step after "2. Check the tasks" (renumber the steps after it) reading the last `## Review` heading in `<phase>/review.md`, with the three cases from R6; don't continue until the user chooses
- [ ] `phase-archive` wrap-up: add the review line from R6
- [ ] `phase-run` step 6: when every task is ticked, suggest `/donatello:phase-review`, then `/donatello:phase-archive`
- [ ] `templates/claude-block.md`: add `/donatello:phase-review` to Commands, after `phase-run`
- [ ] Check: the reviewer walks archive with no `review.md`, changes requested, and approved against R6; the block measures under 300 tokens (`sed -n '/donatello:start/,/donatello:end/p' templates/claude-block.md | wc -c` / 4); `claude plugin validate .` passes
- [ ] Commit: `feat: connect phase-review to phase-run and phase-archive`

### Task 4: Docs and README
**Serves:** R8
**Files:** `docs/SPECIFICATION.md`, `docs/ARCHITECTURE.md`, `README.md`

- [ ] SPECIFICATION: a workflow for `/donatello:phase-review` after the last workflow, without renumbering others; the archive workflow notes the soft gate; `skills/phase-review/` and `templates/review.md` in the directory tree
- [ ] ARCHITECTURE: a `phase-review` component (frontmatter, reads, writes, guardrails); the archive entry updated with the gate; this phase's lasting decisions; the edge cases from R8; block copy synced with `templates/claude-block.md`
- [ ] README: the command in Commands; `/donatello:phase-review` in the flow diagram between `phase-run` and `phase-archive`; the review step in "How a phase works"; `skills/phase-review/` in the layout; block copy synced
- [ ] Check: `grep -n "phase-review" README.md docs/*.md` covers the command, the archive gate, and `review.md`; block copies match the template
- [ ] Commit: `docs: document /donatello:phase-review`

### Task 5: Fixture verification
**Serves:** R1–R7

- [ ] Fixture with all tasks ticked: brief shows without source reads; approved round 1 written and committed with only the phase dir
- [ ] Second run: brief shows round 1; changes-requested round 2 with comments appended; round 1 unchanged
- [ ] Fixture with an unticked task: "review anyway" gives a `(partial)` round
- [ ] Archive: no `review.md` → asks; changes requested → asks; approved → continues; archived folder contains `review.md`
- [ ] `phase-run` on the last task suggests the review before the archive
- [ ] Log anything unexpected in this phase's `walkthrough.md` and fix the skill text
- [ ] Commit: `fix: adjust phase-review from fixture trial` (only if fixes were needed)
