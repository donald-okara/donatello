# Phase 4: Design

### Talk first, then one diff
**Decision:** phase-discuss discusses the change, then proposes a single diff across the phase files, applied only on approval.
**Why:** it matches onboard and phase-archive (propose, then write); one diff lets you see spec, design, and task changes together.
**Tradeoff:** more turns than editing by hand. A hand-edit made beforehand is still caught by the consistency checks.

### Ticked tasks are frozen
**Decision:** ticked tasks are never edited; rework becomes a new task "(revises Task M)".
**Why:** each commit keeps matching the task text it was made for, and the walkthrough's "Changes by task" stays true.
**Tradeoff:** a phase can grow revisit tasks instead of tidy edits.

### Task numbers are never reused or renumbered
**Decision:** new tasks take the next free number; removed and deferred tasks and dropped requirements stay as heading stubs; order is expressed by the dependency sketch.
**Why:** commits, the walkthrough, and reviewer notes cite task numbers.
**Tradeoff:** stub headings accumulate in `tasks.md` and `spec.md`.

### Where the reason goes
**Decision:** requirement and scope changes get a dated `design.md` entry (Decision / Why / Tradeoff, citing R-numbers); task reshaping is recorded only in the commit message; "Deviations from spec" stays for results that differ from the spec.
**Why:** `design.md` already collects decisions made during the phase, and phase-archive promotes the lasting ones. After a revision the spec matches the plan, so it isn't a deviation.
**Tradeoff:** task-only changes leave no in-file trace beyond git history.

### Deferred work lives in the overview
**Decision:** deferred tasks become `(deferred)` stubs in `tasks.md` and are listed in `overview.md` Out of Scope as "Deferred: Task <N> …"; phase-archive adds them to `Carried over:`.
**Why:** a walkthrough may not exist yet when you revise, but the overview always does, and phase-new already reads `Carried over:`.

### Commit only the phase files
**Decision:** phase-discuss stages and commits only the phase directory, and works with uncommitted code changes present.
**Why:** its main use is right after phase-run stops mid-task, when uncommitted changes are expected and belong to you.

### phase-run suggests, never edits the plan
**Decision:** phase-run's Stop step suggests `/donatello:phase-discuss`; plan-change hints from reviews go into its report, not into `tasks.md`.
**Why:** plan changes need your agreement and a recorded reason; this phase-3 run edited `tasks.md` and `spec.md` by hand mid-run, which is what this command replaces.

### Heading stubs instead of gaps (2026-10-19)
**Decision:** removed or deferred tasks keep `### Task <N>: <title> (removed: <reason>)` / `(deferred)` headings with no boxes; dropped requirements keep `### R<N>. <name> (dropped <date>)`; phase-archive skips dropped stubs when promoting (R3, R4, R5, R7).
**Why:** with plain gaps, removing the highest-numbered task or requirement erased any trace of its number, so the next one would reuse it. Found in Task 2's review.
**Tradeoff:** stub headings stay in the files; they have no boxes, so phase-run and phase-archive skip them.
