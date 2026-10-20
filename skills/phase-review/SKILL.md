---
name: phase-review
description: Record a maintainer's review of a finished phase: brief, verdict, comments, appended to review.md.
argument-hint: "[phase]"
disable-model-invocation: true
---

# Review a phase

Record a maintainer's review of a phase: a brief, a verdict, and comments, appended to `<phase>/review.md`. The review is a record. Nothing is acted on: no tasks change, no plan changes, no code is read. Arguments: `$ARGUMENTS`

## 1. Load

- **Phase:** the one named in the arguments, else the single directory in `.spec/ongoing/`. If there are several, ask which one. If there are none, say so and stop.
- **Ramble branch:** if the current branch is `ramble/*`, stop with "this is a ramble branch; end it with `/donatello:ramble end` or switch to main".
- **Tasks:** read `<phase>/tasks.md`. If any boxes are unticked, list those tasks by number and title and ask:
  - **review anyway**: the round is recorded as `(partial)`
  - **stop**: leave the phase unreviewed

  Don't continue until the user chooses. Remember whether the review is partial.

## 2. Brief

Show this in chat before asking anything. About 15 lines, in this order:

1. **Goal and success criteria** from `<phase>/overview.md`.
2. **How to verify**, **Deviations from spec**, and **Current state** from `<phase>/walkthrough.md`. Each only if present.
3. **Commits by task.** Find the phase start with `git log --format=%h --diff-filter=A -- <phase>/tasks.md`, then run `git log --oneline <start>..HEAD` and `git show --stat` per commit. Condense to files per task. Outside a git repo, skip this and say so.
4. **Earlier round**, if `<phase>/review.md` exists with a `## Review` heading: the latest round's verdict and comments, so the maintainer can say which are resolved.

Read only the phase files and git log/stat output. Don't read source files or diffs, and don't dispatch agents.
