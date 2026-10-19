# Phase 5: Walkthrough

## Current state

Tasks 1–2 are done: `skills/tour/SKILL.md` has Load, six stops (incl. no-phase and no-walkthrough cases), area focus and navigation, and a read-only section.
Next: Task 3 (onboard newcomer question, onboarded branch).
Open: non-onboarded + area may dispatch the scout twice; Load could pass the area as focus (reviewer NOTE, optional).
Donatello agents weren't installed in this session; tasks ran on general-purpose agents (Haiku implementer, Sonnet reviewer) briefed with `agents/*.md`.

## How to verify

- `claude plugin validate .` passes
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

## Deviations from spec

None.
