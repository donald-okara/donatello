# Phase 2: Walkthrough

## Current state

Shipped: `catalog/skills.yaml`, `/donatello:skills` (list, add, remove, import), onboard's skills step (6g, which runs before plugin pruning in 6h), and the docs. Fixtures were reviewed as text only. On 2026-10-18 a real onboarding of a new project ran the skills step without problems.
Open: tag refs in git URLs aren't resolved and fall back to asking. In resume mode, onboard must compute the already-present list before it offers candidates.
Carried over (Task 5, live checks):
- `add` / `list` / `remove` from a scratch project (duplicate refused, local fixture folder copied and removed)
- import in both scopes ("committed" and "just me"), re-import reporting skills as already present, and an android entry failing when `android` is off `PATH`
- onboard filtering by `applies-to` on an Android fixture, resume mode offering the remaining candidate, and `--defaults` importing nothing
- log and fix anything unexpected, then commit with `fix: adjust skill import from fixture trial`

## How to verify

- `ruby -ryaml -e 'd=YAML.load_file("catalog/skills.yaml"); abort unless d.all?{|e| %w[id source ref summary].all?{|k| e.key?(k)}}'` passes (PyYAML isn't installed here; the Python check in tasks.md is equivalent)
- `claude plugin validate .` passes
- `/donatello:skills` add/list/remove and import, plus the onboard skills step: the live checks under "Carried over" above

## Changes by task

### Task 1: Catalog file
- Added `catalog/skills.yaml`: a header comment documenting the fields, sources, and ref forms; android entries `adaptive`, `navigation-3`, `styles` (summaries condensed from `android skills list`, `applies-to: [android]`); plugin entry `superpowers` (`superpowers@claude-plugins-official`, marketplace `anthropics/claude-plugins-official`)
- Added `catalog/skills/.gitkeep`
- Check ran with Ruby's YAML instead of PyYAML (not installed)
- Reviewer note: the superpowers summary paraphrases the marketplace description ("planning", "finishing branches" vs "brainstorming", "authoring skills"); non-blocking

### Task 2: `/donatello:skills` — list, add, remove
- Added `skills/skills/SKILL.md` (explicit-only): find the catalog (clone from cwd or `~/.claude/donatello.json`; ask-and-save only for add/remove; list/import fall back to the installed copy), list, add for the four ref forms, remove, commit/push offer; `## import` placeholder for Task 3
- git refs: parses `github.com/<owner>/<repo>/tree/<ref>/<path>`, 40-hex used as-is, branches with `/` resolved by prefix via `git ls-remote`, no `tree/` = HEAD; `SKILL.md` fetched from raw.githubusercontent.com; stored as `tree/<sha>/<path>`
- Live headless Check blocked (writes outside the project refused; bypass run refused by auto mode); per Donald, replaced with a reviewer walk-through, live run moved to Task 5
- Fix round: clone prompt limited to add/remove, spec-ID reference removed, git ref handling specified, not-found failures stop explicitly
- Reviewer notes: tag refs aren't resolved (`refs/heads/` only) and fall back to asking; the import placeholder must be replaced in Task 3

### Task 3: `/donatello:skills` — import
- Replaced the import placeholder in `skills/skills/SKILL.md` with four separable parts: Candidates (lists, asks nothing), Ask (all / pick / none, then scope; unknown ids rejected), Install (per-source table for committed and just me), Summary (imported by scope, already present, failed with fallback commands, committed paths, new-session note)
- Install re-checks `.claude/skills/<id>/` before each entry and skips it if present; settings JSON merges missing keys only, and invalid JSON fails the entry untouched
- Git fetch: `GIT_TERMINAL_PROMPT=0`, `git init` + `fetch --depth 1 <sha>` + checkout of the path in a `mktemp` dir (always removed), `SKILL.md` checked before copying, partial copies removed
- Fix round: per-source fallback commands, separable parts for onboard, "stop" → go to Summary, explicit skip for existing folders and unknown picks; plus the robustness notes and a `git check-ignore` step for `.claude/settings.local.json` on just-me plugins
- Not run live: git fetch and `/plugin` commands (Task 5)

### Task 4: Onboard integration
- `skills/onboard/SKILL.md` (94 → 100 lines): step 3 builds skill candidates via the import's Candidates section; step 4 shows the list and asks all / pick / none and the scope; step 5 lists candidates and choices; new 6g Skills runs Install; plugin pruning moved to 6h and never proposes an imported or already-present catalog plugin; `--defaults` skips import and points to `/donatello:skills import`; resume mode offers candidates; step 7 includes the import summary and commits committed skill files plus `.gitignore` lines, not just-me files
- Differs from the task text: Skills is 6g and pruning 6h (letters follow text order) instead of a new 6h before 6g
- Fix round: list shown before the question, proposal wording, explicit pruning exclusion, letter order, `.gitignore` in the commit
- Reviewer note: in resume mode, make sure the already-present list is computed when candidates are offered (check in Task 5)

### Task 6: Docs and README
- README: `/donatello:skills` in Commands and Quick start, a Skill catalog section (format, sources with `add` examples, import scopes, failures), onboard's skill step, `.claude/skills/` in the project tree, `catalog/` and `skills/skills/` in the repo layout
- SPECIFICATION: the catalog replaces "domain skills ship in the plugin"; workflow 5 (`/donatello:skills`); onboard's skill step; repo and project trees
- ARCHITECTURE: `skills` component, plugin layout, this phase's decisions, 17 edge-case rows
- Fix round: duplicate README heading, "re-adding refreshes the pin" corrected to remove + add (also fixed in this phase's design.md), list/import read the clone when found, just-me marketplace entry only when unknown

## Deviations from spec

None
