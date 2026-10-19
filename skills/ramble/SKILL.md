---
name: ramble
description: Explore a question on a branch or worktree with teaching on and notes in .spec/rambles/, then end it as throwaway, kept, merged, PR, or promoted to a phase.
argument-hint: "[topic | end]"
disable-model-invocation: true
---

# Ramble

Explore a question with the user in the main session: a branch or worktree for experiments, a notes file for what was learned, and a wrap-up that decides what survives. Arguments: `$ARGUMENTS`

Three entry points:
- **No argument:** the picker (section "No argument"): continue the current ramble, resume an open or parked one, or start a new one.
- **A topic:** start a new ramble on it, skipping the picker (section "Start").
- **`end`:** wrap up the current ramble (section "End"). Outside a ramble, the picker lists rambles to choose from.

Notes live in `.spec/rambles/` in the main checkout. The template is `templates/notes.md` in this skill's directory.

## Start

1. **Topic:** from the argument, or ask what to explore. Make a kebab-case slug of at most 4 words.
2. **Collisions:** if `.spec/rambles/<date>-<slug>.md` exists, or branch `ramble/<slug>` exists (`git show-ref --verify --quiet refs/heads/ramble/<slug>`), ask for another topic or slug. Never reuse one.
3. **Git:** run `git rev-parse --is-inside-work-tree`. If it isn't a git repo, go to "Notes-only" below.
4. **Mode:** ask every time, **worktree** or **in place**. Don't pick for them.

### Worktree

1. Create the worktree before anything is committed, so the branch doesn't carry the notes file: `git worktree add ../<repo>-ramble-<slug> -b ramble/<slug>`, where `<repo>` is the basename of `git rev-parse --show-toplevel`.
2. Write the notes file into the main checkout's `.spec/rambles/` (the directory you started in), then commit it on main: `git add .spec/rambles/<date>-<slug>.md` and `git commit -m "docs: start ramble <slug>"`. If the main checkout isn't on `main`, ask first.
3. From now on, work in the worktree by absolute path. Say so in one line.

### In place

1. Require a clean tree: `git status --porcelain` must be empty. If it isn't, offer three choices: **stash** (`git stash push -u -m "ramble <slug>"`, and note in the Log that it was stashed), **worktree instead** (go back to the worktree steps), or **stop**.
2. `git switch -c ramble/<slug>`.
3. Write the notes file into `.spec/rambles/`. Leave it **untracked**; it crosses to main only at `end`.

### Notes-only (not a git repo)

Create `.spec/rambles/` if needed and write the notes file with `mode: notes-only` and empty `branch` and `worktree`. No commits, no branch, and no outcome step at the end.

### Notes file

- Filename `.spec/rambles/<date>-<slug>.md`, with `<date>` from `date +%F`.
- Copy `templates/notes.md`, set `status: open`, `mode`, `branch` (`ramble/<slug>`, or empty), `worktree` (absolute path, or empty), and `started` (today). Set the title to the topic.
- Question: the topic restated as a question, confirmed with the user in one line.
- Remove every HTML comment from the file. The comments are guidance, not content.

### Finish

Tell the user the branch, the worktree path or "in place", and the notes path. Then enter ramble mode (section "Ramble mode").
