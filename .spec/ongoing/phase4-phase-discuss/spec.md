# Phase 4: Specification

## Requirements

### R1. Command and loading
- Explicit-only skill at `skills/phase-discuss/SKILL.md`; argument hint `[topic]`
- Phase: the single directory in `.spec/ongoing/`; with several, ask; with none, suggest `/donatello:phase-new` and stop. Archived phases are never revised
- Refuses on a `ramble/*` branch (same message style as phase-run)
- Reads `overview.md`, `spec.md`, `design.md`, `tasks.md`, and the walkthrough's "Current state" if it exists; no source code. Facts about the code come from `donatello:scout` when needed
- Notes which tasks are ticked and which requirements each task serves
- **Check:** with two ongoing phases it asks which; with none it suggests phase-new; on `ramble/*` it refuses; the session reads no files outside `.spec/` unless the scout is dispatched

### R2. Discussion
- Starts from the topic argument, or asks what to change; one question at a time
- Says so when a change contradicts the overview's goal, or is really a new phase rather than a revision (and offers to defer it instead)
- **Check:** asking to add work unrelated to the phase's goal gets the "new phase" response and a defer offer

### R3. Task changes
- Unticked tasks can be edited, split, merged, deleted, reordered, or deferred; new and changed tasks follow "Writing tasks" in `../phase-new/SKILL.md`
- Ticked tasks are never changed; rework is a new task titled `Task <N>: <title> (revises Task <M>)`
- Task numbers are never reused or renumbered: new tasks take the next free number; deleted or deferred tasks keep their heading as a stub with no checkboxes (`### Task <N>: <title> (removed: <reason>)` or `(deferred)`), outside the dependency sketch; the next free number is one above the highest task heading; the dependency sketch at the top of `tasks.md` is updated to the new order
- **Check:** splitting Task 3 of a 5-task phase gives Task 6 for the new part, Tasks 1, 2, 4, 5 unchanged in number, and a sketch that matches

### R4. Requirement changes
- Requirements in `spec.md` can be edited, added (next free R-number), or dropped (the heading stays as `### R<N>. <name> (dropped <YYYY-MM-DD>)` with no bullets, so the number is never reused); a ticked task citing a dropped requirement is exempt and recorded as "no change needed"
- Every task whose `Serves:` cites a changed or dropped requirement is listed: unticked ones are revised in the same diff; ticked ones get a revisit task or an explicit "no change needed" agreed with the user
- **Check:** changing a requirement cited by one ticked and one unticked task revises the unticked task and proposes a revisit task (or records "no change needed") for the ticked one

### R5. Recording why
- Requirement or scope changes add a dated `design.md` entry: `### <title> (<YYYY-MM-DD>)` with Decision / Why / Tradeoff, citing R-numbers
- Task-only reshaping adds nothing to the phase files beyond the change itself
- Deferred tasks become stubs in `tasks.md` (R3) and are added to `overview.md` "Out of Scope" as `Deferred: Task <N> <title>: <one-line reason>`
- If `walkthrough.md` exists, its "Current state" is rewritten to reflect the new plan; "Deviations from spec" is not touched
- **Check:** a requirement change produces a dated design entry; a deferral produces a `Deferred:` line; a task split produces neither

### R6. Consistency checks, apply, commit
- Before showing the diff, check: no ticked task's text changed; every `Serves:` ID exists in `spec.md`; every requirement is served by at least one task (ticked or not) or was dropped; the dependency sketch lists exactly the tasks in the file
- Show the whole change as one diff; apply only on approval; on decline, write nothing and summarise the discussion in chat
- Commit only the phase directory: `git add <phase dir>` then `git commit -m "docs: revise phase <N> plan (<summary>)"`; uncommitted changes elsewhere are left as they are
- **Check:** with an uncommitted source change present, the revision commit contains only files under the phase directory and the source change is still uncommitted

### R7. Integration
- `phase-run` Stop step: also suggests `/donatello:phase-discuss` when the task or spec looks wrong
- `phase-run`: when a reviewer NOTE implies a plan change, it goes in phase-run's report; phase-run never edits `tasks.md` or `spec.md` beyond ticking boxes
- `phase-archive` step 3: `Carried over:` also lists `overview.md` items marked `Deferred:`
- `phase-archive` step 4: `(dropped <date>)` requirement stubs are never promoted
- `templates/claude-block.md`: `/donatello:phase-discuss` in Commands; the block stays under 300 tokens
- **Check:** each of the four files contains the change; the block measures under 300 tokens (chars / 4)

### R8. Documentation
- `docs/SPECIFICATION.md`: workflow 7 for `/donatello:phase-discuss`
- `docs/ARCHITECTURE.md`: a `phase-discuss` component, this phase's lasting decisions, edge cases (uncommitted changes, no or several ongoing phases, change that is really a new phase, all tasks ticked, diff declined, `ramble/*` branch)
- `README.md`: the command in Commands and a short section
- **Check:** each doc mentions `/donatello:phase-discuss`, frozen ticked tasks, and that numbers are never renumbered

## Non-Functional Requirements

- **NF1. Token cost:** explicit-only, so always-on cost rises by ~50 tokens plus ~5 in the block; the skill points to phase-new's task rules instead of repeating them
- **NF2. Safety:** nothing is written without approval of the diff; nothing outside the phase directory is staged or committed
