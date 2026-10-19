# Phase 4: Walkthrough

## Current state

Tasks 1–2 are done: `skills/phase-discuss/SKILL.md` is complete (Load, Discuss, Propose, Check and apply).
Next: Task 3, integration with phase-run, phase-archive, and the block.
Open (needs Donald's call, not edited by phase-run): removed tasks and dropped requirements now stay as heading stubs (see Deviations). spec.md R3/R5 and design.md still say "gaps"; phase-archive should skip `(dropped)` requirement stubs when promoting.

## How to verify

- `claude plugin validate .` passes
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

## Deviations from spec

- R3 / R5: removed and deferred tasks keep their heading as a stub (`(removed: <reason>)` / `(deferred)`, no boxes, out of the sketch) instead of leaving gaps; dropped requirements keep `### R<N>. <name> (dropped <date>)`; `Deferred:` lines carry the task number. Needed so the next free number can be read from the files. Coordinator's choice during the fix round; not yet confirmed by Donald.
