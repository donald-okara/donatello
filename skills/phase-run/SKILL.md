---
name: phase-run
description: Run the active phase's next task: implementer (Haiku) builds it, reviewer (Sonnet) checks it, then commit, tick, and update the walkthrough.
argument-hint: "[task | --all]"
disable-model-invocation: true
---

# Run phase tasks

You coordinate; the agents do the work. Arguments: `$ARGUMENTS` (a task number, `--all`, or nothing for the next task).

**Stay light.** Don't read source files, diffs, or build logs yourself, and don't run builds or tests. Work from the agents' reports. Read only `tasks.md`, `walkthrough.md`, and `git status --porcelain`.

**Plan files:** never edit `tasks.md` or `spec.md` beyond ticking boxes. A reviewer NOTE that implies a plan change goes in your report instead.

## 1. Set up

- **Phase:** the single directory in `.spec/ongoing/`. If there are several, ask which one. If there are none, suggest `/donatello:phase-new` and stop.
- **Ramble branch:** if the current branch is `ramble/*`, stop with "this is a ramble branch; end it with `/donatello:ramble end` or switch to main".
- **Clean tree:** run `git status --porcelain -- . ':!.spec/rambles'`. If there are uncommitted changes, stop and ask the user to commit or stash them first; otherwise the review and commit would pick them up. Changes under `.spec/rambles/` are ignored, since worktree rambles append to their notes in the main checkout. If this isn't a git repo, skip commits throughout and give the reviewer the list of files the implementer reported instead.
- **Task:** the one named, or else the first task in `tasks.md` with unticked boxes whose dependencies (see the sketch at the top) are done. Note its number and the requirement IDs in its `**Serves:**` line.

## 2. Implement

Dispatch the `donatello:implementer` agent with a short prompt: the phase directory, the task number, and the requirement IDs. Pass paths and IDs only; the agent reads what it needs.

- `STATUS: DONE` → step 3.
- `STATUS: BLOCKED` → if the blocker is a missing decision or something only the user can provide, stop and bring it to the user. Otherwise escalate (step 4).

## 3. Review

Dispatch the `donatello:reviewer` agent with the phase directory, the task number, and the requirement IDs.

- `VERDICT: PASS` → step 5.
- `VERDICT: ISSUES` caused by the environment (a broken tool or missing setup, not the change) → stop and bring it to the user. Don't loop on it.
- `VERDICT: ISSUES` → step 4.

## 4. Fix, then escalate

Follow this ladder, re-running the review (step 3) after each step:
1. **Fix round:** dispatch the implementer again with the same inputs plus the reviewer's issue list, verbatim.
2. **Escalate:** still failing (or blocked in step 2)? Dispatch the implementer again with the Agent tool's `model` set to `sonnet`, passing every issue list so far.
3. **Stop:** still failing? Bring the latest issues to the user, briefly, with your read on whether the task or spec needs changing. If it looks wrong, suggest `/donatello:phase-discuss`. Leave the changes uncommitted so the user can inspect them.

## 5. Record and commit

After a PASS:
1. **Tick** every box of the task in `tasks.md`.
2. **Walkthrough:** if `<phase>/walkthrough.md` doesn't exist, create it from `../phase-new/templates/walkthrough.md` (relative to this skill's directory) and remove the HTML comments. Then:
   - **Changes by task:** append `### Task <n>: <title>` with the implementer's FILES and NOTES condensed to a few bullets, any reviewer `NOTE:` lines, and `Escalated to Sonnet` if that happened.
   - **How to verify:** add or update the task's Check.
   - **Deviations from spec:** add only results that differ from `spec.md`. Mismatches between the task text and the spec (where the code follows the spec) go in that task's Changes entry instead.
   - **Current state:** rewrite it in 10 lines or fewer: what's done, what's next, anything open.
3. **Commit** everything except `.spec/rambles/` (`git add -A -- . ':!.spec/rambles'`; the tree was clean before the task) using the task's `Commit:` line as the message.

## 6. Continue or stop

- Without `--all`: report in 3–4 lines (task done, review result, escalation if any, the next task) and stop.
- With `--all`: go back to step 1 for the next task. Stop when the phase is done, or when any step needs the user.
- When every task is ticked, suggest `/donatello:phase-review` for the maintainer's review, then `/donatello:phase-archive`.
