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

## No argument

Runs only with no argument, or with `end` outside a ramble (then the picker lists only open and parked rambles, and the choice goes to section "End"). A topic never reaches this section.

Find the notes from any checkout through the main checkout: the first entry of `git worktree list --porcelain`, then its `.spec/rambles/`.

1. **Current ramble:** if the branch is `ramble/<slug>`, or the top-level path matches the `worktree` field of a notes file, offer **continue `<slug>`** first. Choosing it goes to "Resume". If that branch has no notes file, treat it as an orphan branch below instead.
2. **Picker:** otherwise, read the frontmatter of each file in `.spec/rambles/` and list, in order:
   - **open:** slug, `started` date, question, and the time on the last Log line.
   - **parked:** the same fields.
   - **orphan branch:** each `ramble/*` branch from `git branch --list 'ramble/*'` with no notes file. Choosing one creates its notes file first in the main checkout's `.spec/rambles/` (question asked, `mode` from `git worktree list`: `worktree` if it has a worktree, else `in-place`), then resumes it. Commit rule per mode: a **worktree** orphan's notes are committed on main as `docs: start ramble <slug>`; an **in-place** orphan's notes stay untracked until `end`. Commits run with `git -C <main checkout>` on `main`; if main isn't checked out there, ask first.
   - **mark abandoned:** an open notes file whose branch is gone (`git show-ref --verify --quiet refs/heads/ramble/<slug>` fails). Choosing it sets `status: abandoned` and `closed` to today in the main checkout's notes file. If the file is tracked (`git -C <main checkout> ls-files --error-unmatch <path>` succeeds), commit only that file with `git -C <main checkout>` on `main` as `docs: abandon ramble <slug>`; if main isn't checked out there, ask first. If it is untracked (an in-place ramble), set the fields without committing and tell the user the file is untracked and left for them to keep or delete.
   - **new ramble:** asks what to explore, then goes to "Start".
3. Ask which entry the user wants, by number. Don't pick for them.

Nothing to list (no notes, no orphans): go straight to "new ramble".

## Resume

1. Read only the notes frontmatter, the Question, and the last 5 Log bullets. Don't read the rest of the file.
2. **Where to work:**
   - `worktree` set and the path exists: work there by absolute path, and say so in one line.
   - `worktree` set but the path is gone: say so, and offer `git worktree add <path> ramble/<slug>` to recreate it.
   - In place (`mode: in-place`): if the current branch isn't `ramble/<slug>`, require a clean tree (`git status --porcelain` empty; else offer stash or stop), then `git switch ramble/<slug>`.
   - Notes-only: nothing to switch.
3. If `status` was `parked`, set it to `open`. If the notes file is tracked on main, commit only that file with `git -C <main checkout> commit` on `main` as `docs: resume ramble <slug>`, never on the ramble branch or in the worktree; if main isn't checked out there, ask first. If it is untracked (in-place), edit the status in the working copy only; it stays untracked until `end`.
4. Restate the question in one line and say the ramble is resumed. Then enter ramble mode (section "Ramble mode").

## Ramble mode

Applies from Finish (Start) or Resume until the ramble ends.

- **Main session only.** No implementer or reviewer agents. Launch `donatello:scout` only when the user asks for a survey.
- **Teaching on:** explain why things work or break, compare designs with their trade-offs, and follow tangents that bear on the question.
- **Preferences relaxed:** tests, previews, and naming rules are skipped unless the experiment is about them. Say so once, when the ramble starts.
- **Main is never touched** except the notes file. Experiments happen on `ramble/<slug>` or in its worktree.
- **Checkpoints:** commit when something works or before something risky, with a `wip: <what>` message. Stage with `git add <paths>`, never `git add -A`, and never stage the notes file on the branch.
- **Log:** when something is learned, a design is tried, a dead end is hit, or a question is answered, append one line to the notes file's Log: `HH:MM — <what was learned or tried>`. No code or raw output.
- **Building a skill:** suggest `.claude/skills/<id>/SKILL.md` so it loads and can be tested in the same ramble.

## End

Runs on `end` from inside a ramble, or from the picker when `end` was given outside one. If the chosen ramble isn't the current one, first go to its worktree, or in place run `git switch ramble/<slug>` after the clean-tree check.

Notes always go in the main checkout's `.spec/rambles/`. Git commands that touch main run with `git -C <main checkout>` on `main`; if main isn't checked out there, ask first. An in-place ramble's notes file is untracked, so `git status` shows it as `??`. That is expected; the checks below exclude `.spec/rambles/`.

**Notes-only (not a git repo):** skip steps 1 and 4. Step 2 draws from the Log alone. Step 3 offers only drafting. Close-out runs any catalog adds, writes Learnings, sets `status: closed` and `closed`, and stops: no git commands of its own, no branch action, and no outcome choice (an added skill is still recorded in Outcome).

1. **Tidy:** first run steps 1 and 2 of "Skills to the catalog" (find and stage), so any skill folder is copied to temp before a discard can delete it. Then run `git status --porcelain -- . ':!.spec/rambles'` in the ramble's checkout. If it shows changes, ask **checkpoint** (`wip: <what>` commit, staging the listed paths with `git add <paths>`) or **discard** (list the files, then ask again). If the user has already chosen throwaway, skip the checkpoint offer and say the listed changes will be discarded with the branch. Never drop changes silently.
2. **Learnings:** draft at most 10 lines from the Log and `git log main..ramble/<slug> --oneline`: what you now know; what didn't work and why; what's still open. Show the draft and let the user edit it. Save it to the notes file's Learnings section; it's committed in close-out.
3. **Skills to the catalog:** follow section "Skills to the catalog" below, from its ask step (the folders were staged in step 1).
4. **Outcome:** ask for one choice and its details. Nothing is changed in git yet.
   - **Throwaway:** show the count from `git log main..ramble/<slug> --oneline | wc -l` and any uncommitted files, then ask yes. Status `closed`; Outcome `throwaway`.
   - **Keep:** the branch and worktree stay. Status `parked`; `closed` stays empty. Outcome `kept`.
   - **Merge:** show the commit count, ask yes, and confirm the squash commit message (draft one from the Question and Learnings). The yes also covers deleting the branch and worktree. Status `closed`; Outcome `merged`.
   - **PR:** ask for the title (default from the Question) and confirm the push. Branch and worktree kept. If there is no remote, say so and offer another choice. Status `closed`; Outcome `pr: <url>`, added after `gh` returns (see close-out).
   - **Promote to phase:** ask whether the branch should also be thrown away. If yes, show the throwaway count and ask yes now, but delete only after `phase-new` returns. Status `closed`; Outcome `promoted: phase<N>-<slug>`, added after `phase-new` returns.
5. **Close-out,** in this order:
   1. Leave the ramble's checkout first: `cd` to the main checkout by absolute path. Removing a worktree the session is inside breaks the session. In place, run `git switch main` after the clean-tree check (same exclusion as step 1; stop on any other change).
   2. Run the catalog adds from section "Skills to the catalog", then remove its temp folder. If the clone is the main checkout (the Donatello repo) and the user declines that add's commit, the catalog change stays uncommitted on main: report it. If the outcome is merge and such a change is uncommitted, ask the user to commit it before the squash, since a squash touching the same paths (e.g. `catalog/skills.yaml`) fails against a dirty tree.
   3. Write Learnings and Outcome into the notes. Set `status`. Set `closed` to today (`date +%F`), except for keep, which leaves it empty.
   4. Commit the notes on main: `git -C <main checkout> add .spec/rambles/<file>` and `git -C <main checkout> commit -m "docs: close ramble <slug>"` (`docs: park ramble <slug>` for keep). The untracked in-place notes come along with the switch.
   5. Run the branch action for the chosen outcome. If a step fails, report it and leave the notes as committed; don't retry or resolve it without asking.
      - **Throwaway:** if there is a worktree, `git -C <main checkout> worktree remove <path>` first, then `git -C <main checkout> branch -D ramble/<slug>`. Never `--force` unless asked; if the removal refuses (e.g. untracked files), report it and ask.
      - **Keep:** nothing.
      - **Merge:** `git -C <main checkout> merge --squash ramble/<slug>`, then `git -C <main checkout> commit -m "<message>"`, then remove the worktree and branch as in throwaway. If the squash conflicts, there is no `MERGE_HEAD`, so `git merge --abort` won't work: report the state from `git -C <main checkout> status`, and don't resolve it without asking.
      - **PR:** `git push -u origin ramble/<slug>`, then from the main checkout `gh pr create --head ramble/<slug> --title "<title>" --body "<Learnings>"` (`--head` is required, or `gh` picks `main`). Then add `pr: <url>` to Outcome and commit on main as `docs: record PR for ramble <slug>`. Branch and worktree kept.
      - **Promote:** run `phase-new` with the notes path as context. If the user also chose throwaway, run the throwaway removal now. Then add `promoted: phase<N>-<slug>` to Outcome and commit on main as `docs: record promotion of ramble <slug>`.

## Skills to the catalog

Find and stage run in End step 1, before Tidy can discard anything. Ask and draft run in End step 3, and the adds in close-out step 2. Notes-only rambles skip find and stage; only drafting is offered.

1. **Find:** in a git ramble, run `git diff --name-only --diff-filter=A main...HEAD` and `git ls-files --others --exclude-standard` in the ramble's checkout. Keep the `SKILL.md` paths outside `.spec/rambles/`. Each folder is a candidate: id (folder name), description (from frontmatter), and file count.
2. **Stage:** copy every candidate folder to `$TMPDIR/donatello-ramble-<slug>/<id>` (`mkdir -p` the parent, then `cp -R`). Nothing is removed from the checkout here.
3. **Ask:** for each staged candidate, **add to catalog** or **skip**. Don't pick for them. Skipped copies are removed with the temp folder at close-out.
4. **Draft (optional):** offer **draft a skill from these Learnings**. Write `SKILL.md` into `$TMPDIR/donatello-ramble-<slug>/<id>/` with `name` and `description` frontmatter and a body from the Learnings and Log, show it for edit, and treat it as a chosen candidate.
5. **Add (close-out step 2):** after returning to main, follow the `add` section in `../skills/SKILL.md` with each chosen temp folder as the ref. Then `rm -rf $TMPDIR/donatello-ramble-<slug>`, whether or not the adds succeeded. A refused or failed add is reported and doesn't stop close-out. Each added id goes in Outcome as `skill added: <id>` (close-out step 3).
