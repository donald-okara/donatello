# Phase 5: Walkthrough

## Current state

Tasks 1–5 and 7 are done: `/donatello:tour` is complete and in the block (~286 tokens, 14 to spare); the scout takes a focus path; onboard asks whether you're new, offers the tour on onboarded repos, and runs newcomer mode on non-onboarded ones; SPECIFICATION, ARCHITECTURE, and README document it all.
Everything so far was reviewed as text; nothing has run live.
Next: Task 6 with Donald (live fixture checks). Riskiest to try first: newcomer mode on a non-onboarded fixture ("docs only", Open questions), and the tour on an onboarded repo reading no source files.
Not released: plugin.json is still 0.2.0, so bump it before `/plugin update` will pick this up.
Donatello agents weren't installed in this session; tasks ran on general-purpose agents (Haiku implementer, Sonnet reviewer) briefed with `agents/*.md`.

## How to verify

- `claude plugin validate .` passes
- Task 7: `grep -n "tour" README.md docs/*.md` covers the command, read-only, and onboard's newcomer question; block copies in README and ARCHITECTURE match `templates/claude-block.md`
- Task 5: block under 300 tokens (`sed -n '/donatello:start/,/donatello:end/p' templates/claude-block.md | wc -c` / 4); scout report format unchanged with a focus path
- Task 4: walk a non-onboarded repo with "new", one "don't know", and "docs only" through onboard steps 4–7 against R5; onboarded resume and `--defaults` unaffected
- Task 3: walk onboarded + new, onboarded + not new, and `--defaults` through onboard step 1 against R4
- Task 2: walk `/donatello:tour auth` where docs cover auth and where they don't (scout with focus path) against R3; no write, edit, or commit path (NF2)
- Task 1: walk an onboarded and a non-onboarded repo through "Load" and "Stops" in `skills/tour/SKILL.md` against R1 and R2

## Changes by task

### Task 1: tour — load and stops
- Added `skills/tour/SKILL.md`: frontmatter (`[area]`, triggers "new to this repo", "show me around", "walk me through this project"; no `disable-model-invocation`), intro, "1. Load", "2. Stops", and the facts-or-"unknown" rule
- Stop 6 (onboarded) takes the next step from the walkthrough's Current state and points to `/donatello:phase-run` rather than reading `tasks.md`; the reviewer judged this meets R2, since phase-run picks the next unticked task
- Reviewer NOTE: no instruction for an onboarded repo with no ongoing phase, or a phase whose walkthrough doesn't exist yet

### Task 2: tour — area focus, deeper, read-only
- `skills/tour/SKILL.md`: added "3. Area and navigation" (area limit, scout with focus path, "nothing matches" case, next / skip to / deeper) and "4. Read-only" (ends by saying nothing changed)
- Coordinator addition from Task 1's review: no ongoing phase → stop 5 says so, stop 6 suggests `/donatello:phase-new`; no `walkthrough.md` yet → stop 6 uses the overview's goal and `/donatello:phase-run`
- Reviewer NOTE: not onboarded with an area dispatches the scout in Load and again in §3; Load could carry the area as focus

### Task 3: onboard — newcomer question, onboarded branch
- `skills/onboard/SKILL.md` step 1: "Are you new to this repo?" (unless `--defaults`), default from git authorship; the answer carries forward as newcomer mode
- "Already onboarded" bullet: new → offers `/donatello:tour` or resume mode, with a shared-files warning before resume; not new and `--defaults` unchanged
- Reviewer NOTE: with no `user.email` set, `--author=""` matches every commit, so the default becomes "no" (fixed in Task 4)

### Task 4: onboard — newcomer mode
- `skills/onboard/SKILL.md`: newcomer clauses in steps 4 (next-work question, open questions as findings, conflicts keep the project's rule), 5 (kept conflicts, "docs only" / "docs + phase 1"), 6c (`## Open questions`, at most 5 lines), 6e (skipped for docs only), 7 (phase-run or phase-new, tour offer, shared-files/PR note)
- Coordinator addition: step 1 defaults to "new" when `git config user.email` is empty
- Step 6f untouched; the reviewer confirmed the block is already written unchanged in docs-only mode
- Fix round on PASS NOTEs: newcomer mode limited to non-onboarded repos (it could leak into resume mode), phase breakdown "unless docs only", tour offer scoped to newcomer mode
- Reviewer NOTE: step 5 doesn't say outright that "docs + phase 1" shows phase 1's detail

### Task 5: Integration
- `templates/claude-block.md`: `/donatello:tour` in Commands (block 1145 chars, ~286 tokens)
- `agents/scout.md`: description names `/donatello:tour`; a brief may name a focus path, which limits reads to it plus top-level docs and manifests; report format unchanged
- Coordinator addition: the tour's not-onboarded Load passes the area as the scout's focus, so the scout runs once (Task 2 NOTE)
- Reviewer NOTE: only ~14 tokens of block headroom remain

### Task 7: Docs and README
- SPECIFICATION: workflow 8 "Tour a repo", newcomer question in workflow 1, `skills/tour/` in the tree, the tour named as the one model-invocable skill
- ARCHITECTURE: `tour` component, onboard newcomer mode, scout focus path, this phase's decisions, edge cases (no git, monorepo, area not found, no docs); block copy synced
- README: "New to a repo?" section, tour command section, newcomer note in onboard, `skills/tour/` in the layout; block copy synced
- Fix round: two "workflow skills are explicit-only" lines (README, SPECIFICATION) now name the tour as the exception
- Reviewer NOTEs: the tour has no monorepo rule beyond `[area]` (documented as such); ARCHITECTURE's "~30 tokens" figure is an estimate

## Deviations from spec

None.
