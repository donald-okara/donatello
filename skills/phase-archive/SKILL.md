---
name: phase-archive
description: Close out a Donatello phase: finalize the walkthrough, promote decisions and requirements to the project docs, and move it to .spec/archive/.
argument-hint: "[phase]"
disable-model-invocation: true
---

# Archive a phase

Close out a phase so that what it shipped lives on in the project docs and the phase folder moves to the archive. Arguments: `$ARGUMENTS`

Nothing is deleted. Doc paths are the ones the Donatello block in `CLAUDE.md` names (defaults: `docs/architecture.md`, `docs/requirements.md`). Changes to them are shown as a diff and applied only after approval.

## 1. Pick the phase

The one named in the arguments, or else the single directory in `.spec/ongoing/`. If there are several, ask which one. If there are none, say so and stop.

## 2. Check the tasks

Read `<phase>/tasks.md`. If any boxes are unticked, list those tasks by number and title and ask:
- **archive anyway**: they're recorded as dropped
- **carry over**: they're recorded for the next phase to pick up
- **stop**: leave the phase open

Don't continue until the user chooses.

## 3. Finalize the walkthrough

**If `<phase>/walkthrough.md` exists:**
- Rewrite **Current state** as a closing summary in 10 lines or fewer: what shipped, what was deferred or dropped, open issues worth knowing about. List carried-over tasks by title under `Carried over:`, plus any `Deferred:` items from `overview.md`. `/donatello:phase-new` reads this section when it plans the next phase.
- Make sure **How to verify** covers the phase as a whole, not just the last task.

**If it's missing:** build it from `../phase-new/templates/walkthrough.md` (relative to this skill's directory, HTML comments removed):
- **Changes by task:** from `tasks.md` and the git history since the phase started. Find the start with `git log --format=%h --diff-filter=A -- <phase>/tasks.md`, then `git log --oneline <start>..HEAD`. Use commit messages and `git show --stat`, not full diffs.
- **How to verify:** collect each task's Check line, and ask the user if there's anything to add.
- Then write Current state as above.

## 4. Promote what outlives the phase

Prepare both changes, then show them together as one diff:

- **Decisions → the architecture doc.** From `<phase>/design.md`, take decisions that shape the project beyond this phase (architecture, conventions, libraries), not phase-local ones. Add each to a `## Decisions` section as a single line with its reason; if the doc already records decisions another way (e.g. one file per ADR), follow that format instead. Update an existing line rather than adding a contradicting one.
- **Requirements → the requirements doc.** From `<phase>/spec.md`, take requirements that describe lasting behavior; skip one-off setup or migration steps. Adjust them for the walkthrough's "Deviations from spec", so the file says what was built. Write them as plain statements grouped under `## <Area>` headings, or match an adopted doc's existing structure. When a new requirement changes an existing entry, **replace** the old entry; never keep both. If the file doesn't exist, create it at the block's path with `# Requirements` and a one-line note that it describes current behavior.

Apply the changes only after approval. Promote nothing for a phase that delivered no lasting behavior or decisions, and say so.

## 5. Move to the archive

1. Get today's date with `date +%F`.
2. Move with **literal paths**, never computed in a subshell: `git mv .spec/ongoing/<dir> .spec/archive/<YYYY-MM-DD>-<dir>`. Create `.spec/archive/` first if needed. Outside a git repo, use `mv` and mention that history isn't tracked.

## 6. Wrap up

Report in a few lines: where the phase now lives, what was promoted, and any carried-over tasks. Suggest `/donatello:phase-new` for the next phase. Offer to commit with `docs: archive phase <N> (<slug>)`; commit only if the user agrees.
