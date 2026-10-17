# Phase 2: Walkthrough

## Current state

Tasks 1–4 are done: the catalog, `/donatello:skills` (list, add, remove, import), and onboard's skill import step, all text-reviewed. Live checks are in Task 5 because headless test runs can't write outside their project and auto mode refuses bypass runs. Next: Task 6 (docs), then Task 5 with Donald. Open: resume mode should compute the already-present list when offering candidates (reviewer note).

## How to verify

- `ruby -ryaml -e 'd=YAML.load_file("catalog/skills.yaml"); abort unless d.all?{|e| %w[id source ref summary].all?{|k| e.key?(k)}}'` passes (PyYAML isn't installed here; the Python check in tasks.md is equivalent)
- `claude plugin validate .` passes
- `/donatello:skills` add/list/remove and import: run interactively in Task 5 (see tasks.md)

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

## Deviations from spec

None
