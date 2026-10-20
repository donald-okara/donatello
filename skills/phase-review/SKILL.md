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

## 3. Verdict and comments

Ask for the verdict: **approved** or **changes requested**. Then ask for comments in one exchange: any number, including none.

Record each comment as one bullet in the maintainer's words. Tidy grammar and length, but don't change the meaning. Claude adds no comments of its own, and nothing said here becomes a task or a plan change.

## 4. Record

1. **Create the file if missing:** if `<phase>/review.md` doesn't exist, build it from `../phase-new/templates/review.md` (relative to this skill's directory, HTML comments removed). Replace `<N>` in the heading with the phase number from the `phase<N>-` directory name.
2. **Round number:** one above the highest `## Review <n>` heading already in the file, or 1 if there is none.
3. **Date:** from `date +%F`.
4. **Reviewer:** from `git config user.name`. If it's unset, or the phase isn't in a git repo, ask the maintainer for a name.
5. **Build the round:**
   ```
   ## Review <n>: <verdict> (<YYYY-MM-DD>, <reviewer>)
   - <comment>
   ```
   Add ` (partial)` after the verdict if the review was partial (section 1). With no comments, write `- No comments.`
6. **Show and ask:** show the round in chat and ask whether to write it. Write only after the maintainer approves. Append the round to `review.md` and leave every earlier round as it is.

## 5. Wrap up

1. **Commit offer:** offer to commit the file with `docs: review phase <N> (<verdict>)`, where `<N>` is the same phase number used in the heading (from the `phase<N>-` directory name). Use `git add <phase dir>` and `git commit -m "docs: review phase <N> (<verdict>)" -- <phase dir>`. The pathspec makes the commit take only the phase directory, even if other changes are already staged; never use `git add -A`. Uncommitted changes elsewhere stay as they are. Outside git, skip this and say so.
2. **Next step:**
   - **approved:** `/donatello:phase-archive`
   - **changes requested:** `/donatello:phase-discuss` to revise the plan, or `/donatello:phase-run` to work the open tasks
3. **Say what didn't change:** no task, spec, design, or other plan file was touched. Only `review.md` changed.
