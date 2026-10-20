# Phase 6: Walkthrough

## Current state

Tasks 1–4 done: `/donatello:phase-review` built, wired into `phase-archive` (soft gate) and `phase-run`, listed in the block, and documented in SPECIFICATION, ARCHITECTURE, and README.
Next: Task 5, fixture verification (interactive, needs Donald).
Open: the block is at ~293 of 300 tokens. Released as 0.4.0.

## How to verify

- Task 1: walk no phase, two phases, a ramble branch, an unticked task, and an earlier round through sections 1–2 of `skills/phase-review/SKILL.md` against R1 and R2
- Task 2: walk an approved round with no comments, a partial changes-requested round with three comments, and a second round through sections 3–5 against R3–R5
- Task 3: walk `phase-archive` with no `review.md`, changes requested, and approved; block under 300 tokens (`sed -n '/donatello:start/,/donatello:end/p' templates/claude-block.md | wc -c` / 4); `claude plugin validate .`
- Task 4: `grep -n "phase-review" README.md docs/*.md` covers the command, the archive gate, and `review.md`; block copies in README and ARCHITECTURE match `templates/claude-block.md`

## Changes by task

### Task 1: phase-review — load and brief
- Added `skills/phase-review/SKILL.md` with frontmatter, intro, "1. Load" (worded like phase-archive step 1, ramble stop matching phase-run), and "2. Brief" (four parts per R2, git commands, outside-git skip, no source/diffs/agents rule)
- Sections 3–5 left for Task 2
- NOTE: phase-start command (copied from phase-archive) doesn't say what to do if `tasks.md` isn't committed or yields several hashes; same as archive, not blocking

### Task 2: phase-review — verdict, record, commit
- `skills/phase-review/SKILL.md`: added "3. Verdict and comments", "4. Record", "5. Wrap up" per R3–R5
- Added `skills/phase-new/templates/review.md` (heading plus a comment on the round format)
- Fix round: section 4 now replaces `<N>` with the phase number from the directory name; the commit message uses the same number

### Task 3: Integration
- `skills/phase-archive/SKILL.md`: new "3. Check the review" (no review / changes requested → "archive anyway / stop"; approved → continue); later steps renumbered 4–7; wrap-up adds the review line
- `skills/phase-run/SKILL.md` step 6: suggests `/donatello:phase-review`, then `/donatello:phase-archive`
- `templates/claude-block.md`: `/donatello:phase-review` in Commands after `phase-run`
- NOTE: block measures ~293 tokens; about 7 tokens of headroom left

### Task 4: Docs and README
- `docs/SPECIFICATION.md`: workflow 9 "Review a phase"; archive gate in workflow 4 step 1; phase-review in the explicit-only list and the directory tree
- `docs/ARCHITECTURE.md`: `phase-review` component; archive entry updated with the gate; five decisions; edge-case rows (no git, unticked, no `user.name`, several rounds, ramble branch, no or several phases); block copy synced
- `README.md`: command section between phase-run and phase-archive; flow diagram; "How a phase works"; archive steps renumbered; layout tree; block copy synced
- Follow-up: Quick start step and token-cost row for phase-review; SPECIFICATION command-list wording

## Deviations from spec

None.
