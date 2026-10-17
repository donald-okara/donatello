# Phase 2: Walkthrough

## Current state

Tasks 1–2 are done: the catalog holds four entries, and `/donatello:skills` has list, add, and remove (text-reviewed; live check deferred). Live checks for Tasks 2–4 moved to Task 5 because headless test runs can't write outside their project and auto mode refuses bypass runs. Next: Task 3, import.

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

## Deviations from spec

None
