# Phase 5: Walkthrough

## Current state

Tasks 1–3 are done: `/donatello:tour` is complete (Load, six stops, area focus, navigation, read-only); onboard asks whether you're new and offers the tour on onboarded repos.
Next: Task 4 (onboard newcomer mode on non-onboarded repos).
Open: non-onboarded + area may dispatch the scout twice (Task 2 NOTE); an unset `git config user.email` makes the newcomer default "no" (Task 3 NOTE).
Donatello agents weren't installed in this session; tasks ran on general-purpose agents (Haiku implementer, Sonnet reviewer) briefed with `agents/*.md`.

## How to verify

- `claude plugin validate .` passes
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
- Reviewer NOTE: with no `user.email` set, `--author=""` matches every commit, so the default becomes "no"

## Deviations from spec

None.
