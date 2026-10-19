# Phase 3: Ramble

## Goal

Give you a sandbox for R&D inside any project: a place to learn, ask questions, break eggs, and try out system designs or new skills without the spec/task/review ceremony of a phase. What you learn always survives; whether the code survives is decided at the end.

## What This Phase Delivers

- `/donatello:ramble [topic | end]`: a picker for pending rambles or a new one, start, resume, and wrap-up
- Each ramble on its own `ramble/<slug>` branch, in a worktree or in place (asked each time)
- A notes file per ramble in `.spec/rambles/`, with a running log and closing Learnings, always committed to main
- Ramble mode: main session only, teaching on, preferences relaxed, checkpoint commits on the branch
- Wrap-up outcomes: throwaway, keep, squash merge, PR, or promote to a phase
- Skills built or learned during a ramble go to the catalog through `/donatello:skills add`, and survive a throwaway
- `phase-new` reads recent ramble Learnings; `phase-run` refuses to run on a ramble branch; the CLAUDE.md block keeps ramble mode across `/clear`
- Updated `docs/SPECIFICATION.md`, `docs/ARCHITECTURE.md`, and README

## Out of Scope

- Sharing rambles with teammates beyond what git already does (they're committed notes and ordinary branches)
- Cleaning up old parked branches automatically
- Changing how `/donatello:skills` finds the clone (the catalog add happens from main instead)
- Rambles in non-git directories beyond a notes-only fallback

## Success Criteria

- `/donatello:ramble` with no argument lists open and parked rambles plus "new ramble"; picking one resumes it, in its worktree when it has one
- A ramble survives `/clear`: a new session on the branch is back in ramble mode after reading only the notes header and last log entries
- `end` with "throwaway" deletes the branch (and worktree) and leaves the closed notes, Learnings included, committed on main
- A skill built at `.claude/skills/<id>/` on a throwaway ramble ends up in `catalog/skills/<id>/` in the clone, including when the ramble ran inside the Donatello repo itself
- "Draft a skill from these Learnings" produces a reviewed `SKILL.md` that goes through the same catalog path
- `phase-new` after a closed ramble cites its Learnings; a phase promoted from a ramble reads the full notes

## Related Docs

`docs/SPECIFICATION.md` (workflows) · `docs/ARCHITECTURE.md` (components, edge cases) · `skills/skills/SKILL.md` ("Find the catalog" and `add`, which wrap-up calls) · `skills/phase-new/SKILL.md` (context gathering, which reads rambles)
