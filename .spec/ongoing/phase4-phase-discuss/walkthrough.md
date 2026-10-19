# Phase 4: Walkthrough

## Current state

Tasks 1–3 are done: `skills/phase-discuss/SKILL.md` is complete; phase-run suggests it at Stop and never edits the plan; phase-archive carries over `Deferred:` items; the block lists the command (~282 tokens).
Next: Task 4, docs and README (sync the block copies).
Plan revised (Donald approved heading stubs): spec R3–R5 and R7, design, and a new Task 6 (phase-archive skips dropped stubs).

## How to verify

- `claude plugin validate .` passes
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

## Deviations from spec

None. (Heading stubs started as a deviation in Task 2; Donald approved them, and the spec and design now describe them.)
