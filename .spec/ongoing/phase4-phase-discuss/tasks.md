# Phase 4: Tasks

Each task ends with a commit. As in phase 3, checks in Tasks 1–3 are reviewer walk-throughs of the skill text; live runs are in Task 5, run interactively. Run this phase after phase 3's Task 8.

```
1. phase-discuss: load and discuss
   └─> 2. phase-discuss: changes, checks, apply
          └─> 3. Integration: phase-run, phase-archive, block
                 ├─> 4. Docs and README
                 └─> 5. Fixture verification (needs Donald)
```

---

### Task 1: phase-discuss — load and discuss
**Serves:** R1, R2
**Files:** `skills/phase-discuss/SKILL.md`

- [x] Frontmatter: `name: phase-discuss`, a one-line `description`, `argument-hint: "[topic]"`, `disable-model-invocation: true`; a short intro stating that ticked tasks are frozen and nothing is written without approval
- [x] Section "1. Load": phase selection (one / several / none), `ramble/*` refusal, the files read (overview, spec, design, tasks, walkthrough Current state only), no source reads, scout on demand; build the ticked list and the task → requirement map
- [x] Section "2. Discuss": start from the topic or ask; one question at a time; flag changes that contradict the overview's goal; for work that is really a new phase, say so and offer to defer it
- [x] Check: the reviewer walks the one, several, none, and `ramble/*` cases and the new-phase response against R1 and R2
- [x] Commit: `feat: add /donatello:phase-discuss load and discuss`

### Task 2: phase-discuss — changes, checks, apply
**Serves:** R3, R4, R5, R6, NF2
**Files:** `skills/phase-discuss/SKILL.md`

- [ ] Section "3. Propose": unticked task operations; frozen ticked tasks with `(revises Task <M>)` rework tasks; numbering rule (next free number, gaps kept, sketch updated); follow "Writing tasks" in `../phase-new/SKILL.md` without repeating it
- [ ] Requirement changes: next free R-number for additions, numbers of dropped ones not reused; list every task citing a changed requirement and handle ticked vs unticked per R4
- [ ] Recording: dated `design.md` entry for requirement or scope changes; `Deferred:` lines in `overview.md`; rewrite walkthrough Current state if it exists; never touch Deviations from spec
- [ ] Section "4. Check and apply": the four consistency checks from R6, one diff, apply on approval, summary only on decline; commit with `git add <phase dir>` and `docs: revise phase <N> plan (<summary>)`
- [ ] Check: the reviewer walks a task split, a requirement change across a ticked and an unticked task, a deferral, and a declined diff against R3–R6
- [ ] Commit: `feat: add plan revision to /donatello:phase-discuss`

### Task 3: Integration
**Serves:** R7
**Files:** `skills/phase-run/SKILL.md`, `skills/phase-archive/SKILL.md`, `templates/claude-block.md`

- [ ] phase-run step 4 "Stop": add that if the task or spec looks wrong, suggest `/donatello:phase-discuss`
- [ ] phase-run: one line stating it never edits `tasks.md` or `spec.md` beyond ticking boxes, and that reviewer notes implying a plan change go in its report
- [ ] phase-archive step 3: `Carried over:` also lists `overview.md` items marked `Deferred:`
- [ ] `templates/claude-block.md`: add `/donatello:phase-discuss` to Commands
- [ ] Check: the block measures under 300 tokens (`sed -n '/donatello:start/,/donatello:end/p' templates/claude-block.md | wc -c` / 4); `claude plugin validate .` passes
- [ ] Commit: `feat: connect phase-discuss to phase-run and phase-archive`

### Task 4: Docs and README
**Serves:** R8
**Files:** `docs/SPECIFICATION.md`, `docs/ARCHITECTURE.md`, `README.md`

- [ ] SPECIFICATION: workflow 7 `/donatello:phase-discuss` after workflow 6, without renumbering others
- [ ] ARCHITECTURE: a `phase-discuss` component (frontmatter, reads, writes, guardrails); phase-run and phase-archive entries updated; this phase's lasting decisions; the edge cases from R8; block copy synced with `templates/claude-block.md`
- [ ] README: the command in Commands, a short "Revising a phase" section, `skills/phase-discuss/` in the layout, block copy synced
- [ ] Check: `grep -n "phase-discuss" README.md docs/*.md` covers the command, frozen ticked tasks, and no renumbering; block copies match the template
- [ ] Commit: `docs: document /donatello:phase-discuss`

### Task 5: Fixture verification
**Serves:** R1–R7

- [ ] Fixture phase with 5 tasks, Tasks 1–2 ticked: split Task 3 → Task 6 added, numbers kept, sketch updated, ticked tasks unchanged
- [ ] Change a requirement served by Task 1 (ticked) and Task 4 (unticked): Task 4 revised, revisit task proposed for Task 1, dated `design.md` entry
- [ ] Defer Task 5: removed from `tasks.md`, `Deferred:` line in `overview.md`; then `phase-archive` lists it under `Carried over:`
- [ ] With an uncommitted source change: the revision commit holds only phase files; decline a diff and confirm nothing was written
- [ ] Two ongoing phases → asks which; on a `ramble/*` branch → refuses
- [ ] Log anything unexpected in this phase's `walkthrough.md` and fix the skill text
- [ ] Commit: `fix: adjust phase-discuss from fixture trial` (only if fixes were needed)
