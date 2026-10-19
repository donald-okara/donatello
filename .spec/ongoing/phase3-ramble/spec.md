# Phase 3: Specification

## Requirements

### R1. Command and picker
- Explicit-only skill at `skills/ramble/SKILL.md`; argument hint `[topic | end]`
- **No argument:**
  - On a `ramble/*` branch or in a ramble worktree: offer "continue `<slug>`" first
  - Otherwise list rambles from `.spec/rambles/`: **open** ones (slug, start date, question, last log time), then **parked** ones, then **new ramble**, which asks what to explore
  - Orphan branches (`ramble/*` with no notes file) are listed as "orphan branch"; ending one creates its notes first
  - An open notes file whose branch no longer exists is offered as "mark abandoned"
- **Topic:** starts a new ramble without the picker
- **`end`:** wraps up the current ramble (R7–R10); when not in one, shows the picker to choose which to end
- **Check:** with one open, one parked, and one orphaned ramble in a fixture, the picker shows all three in the right groups plus "new ramble"

### R2. Starting a ramble
- Slug: kebab-case, at most 4 words, from the topic; if `.spec/rambles/<date>-<slug>.md` or branch `ramble/<slug>` exists, ask for another
- Asks **worktree / in place** every time
  - Worktree: `git worktree add ../<repo>-ramble-<slug> -b ramble/<slug>`; the session moves into the worktree
  - In place: requires a clean tree (else offer stash, worktree instead, or stop); `git switch -c ramble/<slug>`
- Creates the notes file (R3) in the main checkout's `.spec/rambles/`; in worktree mode commits it to main at once (`docs: start ramble <slug>`); in place leaves it untracked
- Not a git repo: notes-only ramble (notes file, log, Learnings; no branch, mode question, or outcome step)
- **Check:** each mode in a fixture gives the branch, the worktree (or not), and the notes file in the stated place; `git log main` shows the start commit only in worktree mode

### R3. Notes file
- Path: `.spec/rambles/<YYYY-MM-DD>-<slug>.md`, from `skills/ramble/templates/notes.md`
- Frontmatter: `status` (`open` | `parked` | `closed` | `abandoned`), `mode` (`worktree` | `in-place` | `notes-only`), `branch`, `worktree` (path or empty), `started`, `closed` (date or empty)
- Sections: **Question** (what you set out to explore), **Log** (terse timestamped bullets), **Learnings** (≤ 10 lines, written at wrap-up), **Outcome** (one line per action taken, e.g. `throwaway`, `skill added: <id>`, `promoted: phase<N>-<slug>`)
- **Check:** a started ramble's file has valid frontmatter and all four headings; a closed one has `status: closed`, a `closed` date, Learnings, and Outcome

### R4. Ramble mode
- Runs in the main session; no implementer or reviewer agents; `donatello:scout` only when the user asks for a survey
- Teaching on: explain why things work or break, compare designs with trade-offs, follow tangents
- Preferences relaxed: tests, previews, and naming rules skipped unless the experiment is about them; said once at the start
- Main is never touched except the notes file
- Checkpoint commits on the branch when something works or before something risky, with `wip:` messages; stage with `git add <paths>`, never `git add -A`, and never stage the notes file on the branch
- Log: append a timestamped one-line bullet when something is learned, a design is tried, a dead end is hit, or a question is answered; no code or raw output
- Building a skill: suggest `.claude/skills/<id>/SKILL.md` so it loads and can be tested live
- **Check:** after a short ramble in a fixture, the branch has `wip:` commits, none containing the notes file, and the log has bullets but no code blocks

### R5. Resume and the CLAUDE.md block
- Resuming reads only the notes frontmatter, Question, and the last 5 log entries, then re-enters ramble mode (moving into the worktree if there is one)
- `templates/claude-block.md` gains one line: on a `ramble/*` branch, read that ramble's notes header in `.spec/rambles/` and follow ramble mode (`/donatello:ramble` resumes)
- **Check:** after `/clear` on a ramble branch, a new session states the ramble's question and that it is in ramble mode; the block stays under 300 tokens

### R6. Wrap-up: tidy and Learnings
- Uncommitted changes on the branch: ask checkpoint or discard; nothing is dropped silently
- Draft Learnings (≤ 10 lines) from the log and `git log main..ramble/<slug> --oneline`: what you now know, what didn't work and why, what's still open; the user edits before it's saved
- **Check:** ending a ramble with uncommitted changes asks first; the saved Learnings is ≤ 10 lines

### R7. Skills to the catalog
- Detect skill folders new on the branch: `SKILL.md` paths in `git diff --name-only main...HEAD` plus untracked files; show each as id, description, and file count; ask **add to catalog / skip** for each
- Offer **draft a skill from these Learnings**: write a `SKILL.md` (frontmatter `name`, `description`; body from Learnings and log), show it for edit, and treat it as a found skill
- Chosen folders are copied to `$TMPDIR/donatello-ramble-<slug>/<id>/` before any branch action
- After returning to main (R9), run `/donatello:skills add <temp folder>` for each, following that skill's own clone lookup and approval; then remove the temp folder
- Each added skill is recorded in Outcome as `skill added: <id>`
- **Check:** a skill built at `.claude/skills/<id>/` on a ramble ended as throwaway ends up in `catalog/skills/<id>/` and `catalog/skills.yaml` in the clone; the same works for a ramble inside the Donatello repo, where the catalog change is on main, not the deleted branch

### R8. Outcomes

| Choice | Effect | Status |
|---|---|---|
| Throwaway | show the count of commits not on main, confirm, then remove the worktree and `git branch -D` | closed |
| Keep | branch and worktree stay | parked |
| Merge | `git merge --squash ramble/<slug>` on main with one written commit message, then delete the branch | closed |
| PR | push the branch and open a PR with `gh`; branch kept | closed |
| Promote to phase | run `phase-new` with the notes path as context; branch kept, or deleted if also throwaway | closed |

- **Check:** each outcome in a fixture leaves the branch, worktree, and status as in the table

### R9. Close-out order
1. Leave the worktree, or switch back to main if in place
2. Run the catalog adds (R7)
3. Write Learnings and Outcome into the notes; set `status` and `closed`
4. Commit the notes on main: `docs: close ramble <slug>`
5. Run the branch action (R8)
- **Check:** if the branch action fails (e.g. a merge conflict), the closed notes and any catalog changes are already committed

### R10. Integration with phases
- `phase-new` reads the **Learnings** of rambles whose `closed` date is after the newest `.spec/archive/` directory date (all closed rambles if there is no archive), at most the 5 most recent; when called from a promote, it reads that ramble's full notes
- `phase-run` stops with a message when the current branch is `ramble/*`
- **Check:** `phase-new` after closing a ramble mentions its Learnings in the new phase's overview or design; `phase-run` on a ramble branch refuses

### R11. Documentation
- `docs/SPECIFICATION.md`: workflow 6 for `/donatello:ramble`; `.spec/rambles/` in the project tree; the block line
- `docs/ARCHITECTURE.md`: a `ramble` component, this phase's decisions, edge cases (not a git repo, dirty tree in place, two rambles at once, `end` outside a ramble, orphan branch, missing branch, ramble inside the Donatello repo)
- `README.md`: the command in Commands, a short "Ramble" section, and `skills/ramble/` in the layout
- **Check:** each doc mentions `/donatello:ramble`, `.spec/rambles/`, and the five outcomes

## Non-Functional Requirements

- **NF1. Token cost:** explicit-only, so always-on cost rises by ~50 tokens; the CLAUDE.md block stays under 300 tokens; resume never reads the whole log
- **NF2. Safety:** no branch or worktree is deleted without showing unmerged commits and a yes; nothing is merged, pushed, or added to the catalog without a yes
- **NF3. Portability:** `git` and `gh` only; works on macOS and Linux
