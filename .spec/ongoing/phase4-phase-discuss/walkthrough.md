# Phase 4: Walkthrough

## Current state

Tasks 1–4 and 6 are done: `/donatello:phase-discuss` is written, wired into phase-run, phase-archive (including skipping dropped stubs), and the block, and documented. Plan revised for heading stubs (Donald approved). Released as 0.2.0.
Next: Task 5 with Donald (live fixture checks).
Open: the "no ticked task text changed" check has no baseline, so a hand-edit to a ticked task made before running phase-discuss isn't caught; comparing against `git show HEAD:<phase>/tasks.md` would fix it. design.md and ARCHITECTURE claim hand-edits are caught. Candidate for `/donatello:phase-discuss` once it's installed.

## How to verify

- `claude plugin validate .` passes
- Task 6: phase-archive step 4 skips `(dropped <date>)` requirement headings
- Task 4: `grep -n "phase-discuss" README.md docs/*.md` covers the command, frozen ticked tasks, and no renumbering; block copies match the template
- Task 3: block under 300 tokens (`sed -n '/donatello:start/,/donatello:end/p' templates/claude-block.md | wc -c` / 4); phase-run Stop suggests phase-discuss; phase-archive step 3 includes `Deferred:` items
- Task 2: walk a task split, a requirement change across ticked and unticked tasks, a deferral, and a declined diff through "Propose" and "Check and apply" against R3–R6 and NF2
- Task 1: walk one / several / no ongoing phases, the `ramble/*` refusal, the read limits, and the new-phase response through "Load" and "Discuss" against R1 and R2

## Changes by task

### Task 1: phase-discuss — load and discuss
- Added `skills/phase-discuss/SKILL.md`: frontmatter (`[topic]`, explicit-only), intro (ticked tasks frozen, nothing written without approval), "1. Load", "2. Discuss"
- The `ramble/*` refusal reuses phase-run's wording
- Reviewer NOTE: Load doesn't handle a topic that names a phase (optional)

### Task 2: phase-discuss — changes, checks, apply
- `skills/phase-discuss/SKILL.md`: added "3. Propose" (unticked operations, frozen ticked tasks with `(revises Task <M>)`, numbering, requirement changes, design entries, `Deferred:` lines, walkthrough Current state) and "4. Check and apply" (four checks, one diff, apply on approval, nothing on decline)
- Commit: `git add <phase dir>` then `git commit -m "..." -- <phase dir>`, so changes you staged earlier are never swept in
- A ticked task citing a dropped requirement is exempt, recorded as "no change needed"
- The first implementer run committed its own change, misreading a coordinator brief about the skill's commit behavior; it undid the unpushed commit before the fix round
- Fix round: 3 review issues (task and R-numbers unrecoverable after removing the highest one; commit without pathspec; check 2 impossible after dropping a requirement cited by a ticked task)

### Task 3: Integration
- `skills/phase-run/SKILL.md`: Stop suggests `/donatello:phase-discuss`; new "Plan files" rule (never edits `tasks.md` or `spec.md` beyond ticking; plan-change notes go in the report)
- `skills/phase-archive/SKILL.md`: `Carried over:` also lists `overview.md` `Deferred:` items
- `templates/claude-block.md`: `/donatello:phase-discuss` in Commands
- Reviewer NOTE: heading stubs have no boxes, so phase-run and phase-archive skip them without any change

### Task 4: Docs and README
- SPECIFICATION: workflow 7, tree entry, phase-run and phase-archive notes in workflows 3 and 4
- ARCHITECTURE: `phase-discuss` component, phase-run and phase-archive updates, this phase's decisions, edge cases; block copy synced
- README: command, "Revising a phase" section, layout, FAQ; block copy synced
- Reviewer NOTE: "a hand-edit made beforehand is still caught" overstates the ticked-task check (see Current state)

### Task 6: phase-archive skips dropped requirement stubs
- `skills/phase-archive/SKILL.md` step 4: `(dropped <date>)` requirement headings are never promoted
- Added by the plan revision for heading stubs

## Deviations from spec

None. (Heading stubs started as a deviation in Task 2; Donald approved them, and the spec and design now describe them.)
