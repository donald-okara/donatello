# Phase 4: Phase Discuss

## Goal

Let you rethink an ongoing phase's plan (requirements, scope, and tasks) through a conversation, then apply the result as one approved diff. Done work stays exactly as it was recorded, and the reason for each change is written down.

## What This Phase Delivers

- `/donatello:phase-discuss [topic]`: load the phase cheaply, discuss, propose one diff across `spec.md`, `design.md`, `tasks.md` (and `overview.md` for deferrals), check it, apply on approval, commit only the phase files
- Rules: ticked tasks are frozen (rework becomes a new "revises Task M" task); task numbers are never reused or renumbered; a changed requirement flags every task citing it
- Requirement and scope changes recorded as dated `design.md` decisions; deferred work marked in `overview.md` and carried over by `phase-archive`
- `phase-run` points to phase-discuss when its ladder stops, and reports plan-change hints instead of editing `tasks.md`
- `/donatello:phase-discuss` in the CLAUDE.md block's Commands
- Updated `docs/SPECIFICATION.md`, `docs/ARCHITECTURE.md`, and README

## Out of Scope

- Revising archived phases
- Editing the project's docs (architecture, requirements); that stays with `phase-archive`
- Moving tasks between two ongoing phases
- Automatic plan changes without a conversation

## Success Criteria

- Splitting an unticked task in a fixture keeps every other task number, adds the new task at the next free number, and updates the dependency sketch
- Changing a requirement cited by a ticked and an unticked task revises the unticked one and proposes a revisit task for the ticked one, with a dated `design.md` entry
- A deferred task disappears from `tasks.md`, shows as "Deferred: …" in `overview.md`, and appears under `Carried over:` after `phase-archive`
- With uncommitted code changes present, the revision commit contains only `.spec/` files
- No ticked task's text changes in any of the above

## Related Docs

`skills/phase-new/SKILL.md` ("Writing tasks", which phase-discuss follows) · `skills/phase-run/SKILL.md` (ladder) · `skills/phase-archive/SKILL.md` (Carried over) · `docs/SPECIFICATION.md` · `docs/ARCHITECTURE.md`
