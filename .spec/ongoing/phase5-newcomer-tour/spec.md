# Phase 5: Specification

## Requirements

### R1. Tour command and loading
- Skill at `skills/tour/SKILL.md`; argument hint `[area]`; model-invocable (no `disable-model-invocation`), so "show me around this repo" can trigger it
- Onboarded (a Donatello block in `CLAUDE.md` or `AGENTS.md`): reads the architecture, requirements, and preferences docs at the paths the block names (each only if it exists), the file names and `paths` frontmatter of `.claude/rules/*.md`, each ongoing phase's `overview.md` and walkthrough "Current state", the directory names in `.spec/archive/`, and the frontmatter of `.spec/rambles/*.md`
- Not onboarded: dispatches `donatello:scout` and works from its report
- No source files are read during loading
- **Check:** on an onboarded fixture, the session reads only the files listed above; on a non-onboarded fixture, the scout is dispatched and no source files are read by the main session

### R2. The stops
- Six stops in order, each at most ~10 lines, each ending with "questions, deeper, or next?":
  1. What this is: purpose, users, stage
  2. Map: modules, stack, how they connect
  3. Run it: exact build and test commands
  4. How code is written here: preferences summary and which `.claude/rules` apply to which paths
  5. Where work stands: ongoing phase and its Current state, recent archived phases, open rambles (skipped when not onboarded)
  6. Where to start: 3–5 files to read first, then the next step (onboarded: the next unticked task and `/donatello:phase-run`; not onboarded: `/donatello:onboard`)
- "Next" moves on; "skip to <stop>" jumps; "deeper" may read specific files to answer the question, the only time code is read
- Facts come from the docs, the scout report, or files read for "deeper"; anything else is said as "unknown", not guessed
- **Check:** a walk-through of an onboarded fixture shows six stops with the pause line; a non-onboarded walk shows five and ends at `/donatello:onboard`; an unanswerable question gets "unknown"

### R3. Area focus and read-only
- With `[area]`, every stop is limited to that area (a module, path, or feature name); if the docs don't cover it, the scout is dispatched with a brief to focus on that path
- The tour never writes, edits, or commits any file
- **Check:** `/donatello:tour <area>` on a fixture mentions only that area's modules, commands, and rules; `git status` is unchanged after a full tour

### R4. Onboard: newcomer question and already-onboarded newcomers
- In step 1, unless `--defaults`, onboard asks "Are you new to this repo?" with a default from git: "yes" when `git log --author="$(git config user.email)" --oneline | wc -l` is 0 or under 5% of all commits (or there is no git)
- Already onboarded and new: says the repo is set up and offers the tour (recommended) or resume mode; choosing resume mode first warns that preferences refresh and block update change shared, committed files
- Not new, or `--defaults`: behavior unchanged
- **Check:** on an onboarded fixture, answering "new" offers the tour; answering "not new" goes straight to resume mode as before; `--defaults` asks nothing

### R5. Onboard: newcomer mode on a non-onboarded repo
- Interview: "what's next" becomes "Do you know the next piece of work? 'No' is fine"; scout open questions are shown as findings to confirm, correct, or mark "don't know"
- "Don't know" answers go to an `## Open questions` section (at most 5 lines) at the end of the architecture doc (a new file, or proposed as a diff to an adopted one); nothing is guessed
- Preference conflicts keep the project's rule without asking and are listed in the proposal
- Proposal offers "docs only" or "docs + phase 1"; "docs only" skips step 6e; the block is written unchanged
- Wrap-up: next step is `/donatello:phase-run` if phase 1 was written, otherwise `/donatello:phase-new`; offers `/donatello:tour`; the commit offer notes that `CLAUDE.md` and `docs/` are shared, so a PR may suit better than a direct commit
- **Check:** on a non-onboarded fixture, answering "new", "don't know" to one open question, and "docs only" gives an architecture doc with that question under Open questions, no `.spec/ongoing/phase1-*`, and a wrap-up naming `/donatello:phase-new` and the tour

### R6. Integration
- `templates/claude-block.md`: `/donatello:tour` in Commands; the block stays under 300 tokens
- `agents/scout.md` description mentions `/donatello:tour` as a user, and the scout accepts an optional focus path from its brief without changing its report format
- **Check:** the block measures under 300 tokens (chars / 4); `claude plugin validate .` passes

### R7. Documentation
- `docs/SPECIFICATION.md`: workflow 8 for `/donatello:tour`; workflow 1 notes the newcomer question
- `docs/ARCHITECTURE.md`: a `tour` component, onboard's newcomer branches, this phase's lasting decisions, edge cases (no git, monorepo, area not found, no docs at all)
- `README.md`: the command in Commands, a short "New to a repo?" section, `skills/tour/` in the layout, block copies synced
- **Check:** `grep -n "tour" README.md docs/*.md` covers the command, read-only, and onboard's newcomer question; block copies match the template

## Non-Functional Requirements

- **NF1. Token cost:** the tour is model-invocable, so its description (~30 tokens) is always loaded; the block grows by ~5 tokens; the tour skill points to onboard and the scout rather than repeating them
- **NF2. Safety:** the tour writes nothing; onboard's never-overwrite and never-duplicate rules apply unchanged in newcomer mode
