---
name: phase-discuss
description: Revise an ongoing phase's plan (requirements, scope, tasks) through a conversation, then apply one approved diff across the phase files. Ticked tasks are never changed.
argument-hint: "[topic]"
disable-model-invocation: true
---

# Discuss a phase plan

Rethink the ongoing phase's plan with the user, then apply the result as one diff they approve. Topic: `$ARGUMENTS`

Ticked tasks are frozen: their text never changes, and rework becomes a new task. Nothing is written until the user approves the diff.

## 1. Load

- **Phase:** the single directory in `.spec/ongoing/`. If there are several, ask which one. If there are none, suggest `/donatello:phase-new` and stop. Archived phases are never revised.
- **Ramble branch:** if the current branch is `ramble/*`, stop with "this is a ramble branch; end it with `/donatello:ramble end` or switch to main".
- **Read:** `overview.md`, `spec.md`, `design.md`, `tasks.md`, and the Current state section of `walkthrough.md` if it exists. Don't read source code. If a fact about the code matters to the discussion, dispatch the `donatello:scout` agent and work from its report.
- **Note:** which tasks are ticked, and which requirements each task serves (its `**Serves:**` line). Keep this in mind for the rest of the session.

## 2. Discuss

- Start from the topic if one was given; otherwise ask what the user wants to change. Ask one question at a time.
- If a requested change contradicts the overview's goal, say so and point to the goal.
- If the request is really new work rather than a revision (it's unrelated to the phase's goal), say that it belongs in a new phase, and offer to record it under `overview.md` "Out of Scope" as a `Deferred:` line rather than adding it to `tasks.md`.
- Keep the discussion in chat. Nothing goes into the phase files until section 3.

## 3. Propose

Build one proposal from the discussion. Ticked tasks are never edited, split, or deleted.

- **Unticked tasks:** edit, split, merge, delete, reorder, or defer. New and changed tasks follow "Writing tasks" in `../phase-new/SKILL.md` (relative to this skill's directory); don't restate those rules here.
- **Rework:** a change to a ticked task is a new task titled `Task <N>: <title> (revises Task <M>)`.
- **Numbers:** a new task takes the next free number, one above the highest heading number in `tasks.md`. Numbers are never reused or renumbered. Update the dependency sketch at the top of `tasks.md` to the new order, listing only live tasks.
- **Removed tasks:** a deleted task keeps its heading as a stub, `### Task <N>: <title> (removed: <reason>)`; a deferred one becomes `### Task <N>: <title> (deferred)`. Stubs have no checkboxes or body and are left out of the sketch, so their numbers stay in the file.
- **Requirements:** an added requirement takes the next free R-number, one above the highest heading number in `spec.md`. A dropped one keeps its heading as `### R<N>. <name> (dropped <YYYY-MM-DD>)` with no bullets, so its number is never reused. List every task whose `**Serves:**` cites a changed or dropped requirement. Revise the unticked ones in the diff so they stop citing a dropped one. A ticked task keeps its citation (its text is frozen), so for each one propose a revisit task, or record "no change needed" in the `design.md` entry if the user agrees.
- **Record:** a requirement or scope change adds a dated `design.md` entry, `### <title> (<YYYY-MM-DD>)` with Decision / Why / Tradeoff, citing R-numbers. A task-only reshape adds nothing beyond the change itself.
- **Deferrals:** turn the task into a `(deferred)` stub in `tasks.md` and add `Deferred: Task <N> <title>: <one-line reason>` under `overview.md` "Out of Scope". Any write offered during Discuss, including a `Deferred:` line, is part of this proposal.
- **Walkthrough:** if `walkthrough.md` exists, rewrite its Current state in 10 lines or fewer to match the new plan. Never touch "Deviations from spec".

## 4. Check and apply

1. **Check** the proposal before showing it, and fix any failure first:
   - No ticked task's text has changed.
   - Every heading number in `tasks.md` is unique, and every `**Serves:**` ID has a heading in `spec.md`. A ticked task may still cite a dropped requirement; its stub heading satisfies this, and the citation is the "no change needed" case.
   - Every live requirement (not a dropped stub) is served by at least one live task (ticked or not). Stubs have no boxes and don't count as serving anything.
   - The dependency sketch lists exactly the live tasks: no stubs, and none missing.
2. **Show** the change for a person to read first, then the detail:
   - **Summary table**, one row per change, in plain words a maintainer can approve without reading the diff:

     | Change | What | Why | Affects |
     |---|---|---|---|
     | Added | Task 9: Retry failed sync | R4 now covers offline | Serves R4; after Task 6 |
     | Removed | Task 7: Settings screen | Moved to a later phase | Deferred line in overview |
     | Changed | R3. Login | Adds biometric option | Tasks 3, 8 |

     Use `Added`, `Changed`, `Removed`, `Deferred`, or `Revises Task <M>` in the first column. Name tasks and requirements by number and title, never by line. Below the table, add one line each for anything the maintainer must decide or watch: ticked tasks that still cite a changed requirement, or tasks whose order or dependencies changed.
   - **Detailed diff:** below that, the whole change as one diff across the phase files, under a `Detailed diff` heading, for anyone who wants to check the exact wording.
3. **Ask** for approval. Write nothing until the user answers.
   - **Approved:** apply the diff, then run `git add <phase dir>` and `git commit -m "docs: revise phase <N> plan (<summary>)" -- <phase dir>`. The pathspec makes the commit take only the phase directory, even if other changes are already staged; never use `git add -A`. Uncommitted changes elsewhere stay as they are.
   - **Declined:** write nothing. Summarise the discussion in chat.
