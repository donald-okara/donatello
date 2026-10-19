# Phase 5: Walkthrough

## Current state

Task 1 is done: `skills/tour/SKILL.md` exists with Load (onboarded docs or scout) and the six stops; model-invocable.
Next: Task 2 (area focus, navigation, read-only guardrail).
Open: the onboarded tour has no instruction for "no ongoing phase" or "phase without a walkthrough yet" (reviewer NOTE); worth a clause in Task 2.
Donatello agents weren't installed in this session; tasks ran on general-purpose agents (Haiku implementer, Sonnet reviewer) briefed with `agents/*.md`.

## How to verify

- `claude plugin validate .` passes
- Task 1: walk an onboarded and a non-onboarded repo through "Load" and "Stops" in `skills/tour/SKILL.md` against R1 and R2

## Changes by task

### Task 1: tour — load and stops
- Added `skills/tour/SKILL.md`: frontmatter (`[area]`, triggers "new to this repo", "show me around", "walk me through this project"; no `disable-model-invocation`), intro, "1. Load", "2. Stops", and the facts-or-"unknown" rule
- Stop 6 (onboarded) takes the next step from the walkthrough's Current state and points to `/donatello:phase-run` rather than reading `tasks.md`; the reviewer judged this meets R2, since phase-run picks the next unticked task
- Reviewer NOTE: no instruction for an onboarded repo with no ongoing phase, or a phase whose walkthrough doesn't exist yet

## Deviations from spec

None.
