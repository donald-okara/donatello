# Phase 6: Maintainer Review

## Goal

Give the maintainer a recorded, human review of a finished phase before it is archived. Add `/donatello:phase-review`, which briefs the maintainer on what the phase delivered, collects a verdict and comments, and appends them to `<phase>/review.md`. `/donatello:phase-archive` warns when a phase has no approval. The review is a record: what to do about comments stays the maintainer's call.

## What This Phase Delivers

- `/donatello:phase-review [phase]`: a guided review in chat
  - a brief built from the phase docs and the commit list (no source, no diffs)
  - a verdict, **approved** or **changes requested**, and freeform comments
  - one round appended to `<phase>/review.md` after approval, then a commit offer
- `skills/phase-new/templates/review.md`: the review file template
- `/donatello:phase-archive` soft gate: no review, or a latest verdict of changes requested, asks "archive anyway / stop"; the report states the review status
- `/donatello:phase-run` suggests `/donatello:phase-review` when every task is ticked
- `/donatello:phase-review` in the CLAUDE.md block's Commands
- Updated `docs/SPECIFICATION.md`, `docs/ARCHITECTURE.md`, and README

## Out of Scope

- Turning review comments into tasks, deferrals, or plan changes (the maintainer uses `/donatello:phase-discuss` or fixes by hand)
- Per-task human review inside `/donatello:phase-run`
- PR or `gh` integration
- Carrying review verdicts or open comments into the walkthrough's closing summary
- The `donatello:reviewer` agent writing to `review.md`

## Success Criteria

- On a fixture with all tasks ticked, `/donatello:phase-review` shows a brief without reading source files, and after approval writes `## Review 1: <verdict> (<date>, <reviewer>)` to `review.md` and commits only the phase directory
- A second run shows round 1's verdict and comments in the brief and appends round 2
- With unticked tasks, the review asks "review anyway / stop" and labels a continued round as partial
- `/donatello:phase-archive` asks "archive anyway / stop" with no `review.md` or a latest verdict of changes requested, and continues without asking when the latest is approved
- When the last task is ticked, `/donatello:phase-run` suggests the review before the archive

## Related Docs

`skills/phase-archive/SKILL.md` · `skills/phase-run/SKILL.md` · `skills/phase-discuss/SKILL.md` · `templates/claude-block.md` · `docs/SPECIFICATION.md` · `docs/ARCHITECTURE.md`
