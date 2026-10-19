# Phase 3: Design

### One command with modes
**Decision:** `/donatello:ramble [topic | end]`: no argument shows a picker (continue the current ramble, pending rambles, or new), a topic starts one, `end` wraps up.
**Why:** one thing to remember, matching `/donatello:skills` subcommands. The picker makes pending rambles visible instead of relying on the branch you happen to be on.
**Tradeoff:** the skill file is longer than two separate ones; the wrap-up sits behind an explicit `end`, so it's not triggered by accident.

### Rambles are not phases
**Decision:** rambles live in `.spec/rambles/`, have no number, spec, tasks, or review loop, and don't go through `phase-archive`.
**Why:** the point is low ceremony. Folding them into phases would muddle numbering and bring back the process a ramble is meant to skip.
**Tradeoff:** a second kind of record in `.spec/`; `phase-new` has to read it explicitly.

### Notes always survive; code survives only by choice, decided at the end
**Decision:** every ramble gets a notes file committed to main; throwaway / keep / merge / PR / promote is chosen at `end`, not at start.
**Why:** you don't know whether an experiment is worth keeping until it's done, but what you learned is worth keeping either way.
**Tradeoff:** every ramble leaves a file on main, even trivial ones. Kept small: the log is terse and Learnings ≤ 10 lines.

### Worktree or in place, asked each time
**Decision:** each new ramble asks. Worktree: `../<repo>-ramble-<slug>`, notes written directly in the main checkout and committed to main at start. In place: `git switch -c`, clean tree required, notes left untracked so they cross back to main on `git switch`.
**Why:** worktrees keep main free for phase work in parallel; in place avoids a second checkout and a cold Gradle build.
**Tradeoff:** two code paths in the skill. In place relies on the notes file never being staged on the branch, so ramble mode uses `git add <paths>`, never `git add -A`.

### Ramble mode runs in the main session, preferences relaxed
**Decision:** no implementer or reviewer; teaching on; tests, previews, and naming rules skipped unless the experiment is about them; checkpoint `wip:` commits on the branch.
**Why:** it's a learning tool, so the reasoning should stay visible, and experiments shouldn't pay for conventions they may throw away.
**Tradeoff:** kept code needs cleanup. Merge squashes the history, and promote-to-phase redoes the work properly.

### Catalog adds happen from main (approach A)
**Decision:** at `end`, chosen skill folders are copied to `$TMPDIR/donatello-ramble-<slug>/<id>/`; after returning to main, wrap-up runs `/donatello:skills add <that folder>`.
**Why:** inside the Donatello repo, `skills` treats the current directory as the clone, so an add on the ramble branch would be deleted with a throwaway branch. Doing it from main fixes that without changing `skills`.
**Tradeoff:** a temp copy step; the temp folder is removed after the add.

### Draft a skill from Learnings
**Decision:** wrap-up can also write a new `SKILL.md` from the Learnings and log, for review, and send it through the same catalog path.
**Why:** a ramble often teaches something skill-worthy without a skill having been written.
**Tradeoff:** a drafted skill is only as good as the notes; it's shown for edit before anything is added.

### Merge squashes
**Decision:** the merge outcome is `git merge --squash` with one written commit message.
**Why:** `wip:` checkpoints are for rolling back during the ramble, not for main's history.
**Tradeoff:** the checkpoint history is lost on main; choose "keep" too if it matters.

### Branch action runs last
**Decision:** close-out order is: back to main → catalog adds → write Learnings and Outcome → commit notes → branch action.
**Why:** if deleting or merging fails, the notes and skills are already safe on main.

### `phase-run` refuses on a ramble branch
**Decision:** `phase-run` stops if the current branch is `ramble/*`.
**Why:** phase work mixed into an experiment branch would be lost with a throwaway.
