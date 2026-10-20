# Phase 6: Specification

## Requirements

### R1. Review command and phase selection
- Skill at `skills/phase-review/SKILL.md`; argument hint `[phase]`; `disable-model-invocation: true`
- Phase: the one named, else the single directory in `.spec/ongoing/`; several → ask which; none → say so and stop
- On a `ramble/*` branch, stops with the same message as `/donatello:phase-run`
- If `tasks.md` has unticked boxes, lists those tasks by number and title and asks "review anyway / stop"; a continued review is marked partial (R4)
- **Check:** the reviewer walks no phase, two phases, a ramble branch, and a phase with one unticked task through the skill text

### R2. Brief
- Shown in chat, about 15 lines, before anything is asked:
  - goal and success criteria from `overview.md`
  - "How to verify", "Deviations from spec", and "Current state" from `walkthrough.md` (each only if present)
  - per-task commits: phase start from `git log --format=%h --diff-filter=A -- <phase>/tasks.md`, then `git log --oneline <start>..HEAD` and `git show --stat` per commit, condensed to files per task
  - if `review.md` has earlier rounds: the latest round's verdict and comments, so the maintainer can say which are resolved
- Reads no source files and no diffs; no agents are dispatched
- Outside a git repo, the commit list is skipped and the brief says so
- **Check:** on a fixture, the session reads only the phase files and git log/stat output; the brief contains each listed part

### R3. Verdict and comments
- Asks for a verdict: exactly **approved** or **changes requested**
- Asks for comments in one exchange; any number, including none
- Each comment is recorded as one bullet in the maintainer's words, tidied but not changed in meaning; Claude adds no comments of its own
- **Check:** the reviewer walks an approved round with no comments and a changes-requested round with three comments; the written bullets match what was said

### R4. Recording a round
- `<phase>/review.md` is created from `skills/phase-new/templates/review.md` (HTML comments removed) if missing
- Each review appends one round, never editing earlier ones:
  ```
  ## Review <n>: <verdict> (<YYYY-MM-DD>, <reviewer>)
  - <comment>
  ```
  `<n>` is one above the highest existing round; `<reviewer>` is `git config user.name` (asked for when unset or outside git); a partial review adds ` (partial)` after the verdict; no comments writes `- No comments.`
- The round is shown and written only after the maintainer approves it
- **Check:** on a fixture, two runs give `## Review 1` and `## Review 2` with round 1 unchanged; date comes from `date +%F`

### R5. Commit and next step
- After writing, offers to commit with `docs: review phase <N> (<verdict>)` using `git add <phase dir>` and `git commit -m … -- <phase dir>`; never `git add -A`
- Suggests the next step: approved → `/donatello:phase-archive`; changes requested → `/donatello:phase-discuss` or `/donatello:phase-run`
- Nothing is turned into tasks or plan changes
- **Check:** with an unrelated staged change, the commit contains only `review.md`; `tasks.md`, `spec.md`, and `design.md` are unchanged

### R6. Archive soft gate
- In `/donatello:phase-archive`, after the task check: reads the last `## Review` heading in `<phase>/review.md`
  - missing file or no round → says "not reviewed" and asks "archive anyway / stop"
  - latest verdict changes requested → says so and asks "archive anyway / stop"
  - latest verdict approved (partial or not) → continues without asking
- The wrap-up report adds one line: `Review: approved (round <n>)`, `Review: none`, or `Review: changes requested (round <n>), archived anyway`
- `review.md` moves into the archive with the rest of the phase directory
- **Check:** the reviewer walks the three cases through the skill text; on a fixture, the archived folder contains `review.md`

### R7. Integration
- `/donatello:phase-run` step 6: when every task is ticked, suggests `/donatello:phase-review`, then `/donatello:phase-archive`
- `templates/claude-block.md`: `/donatello:phase-review` in Commands; the block stays under 300 tokens
- **Check:** the block measures under 300 tokens (chars / 4); `claude plugin validate .` passes

### R8. Documentation
- `docs/SPECIFICATION.md`: a workflow for `/donatello:phase-review`, the archive gate in the archive workflow, `skills/phase-review/` and the review template in the directory tree
- `docs/ARCHITECTURE.md`: a `phase-review` component, the archive gate, this phase's lasting decisions, edge cases (no git, unticked tasks, no `user.name`, several rounds); block copy synced
- `README.md`: the command in Commands, the review step in the flow diagram and "How a phase works", `skills/phase-review/` in the layout, block copy synced
- **Check:** `grep -n "phase-review" README.md docs/*.md` covers the command, the archive gate, and `review.md`; block copies match the template

## Non-Functional Requirements

- **NF1. Token cost:** user-invoked only, so no always-loaded description; the block grows by ~6 tokens; the brief reads phase files and git stat output only
- **NF2. Safety:** never edits earlier review rounds or any phase file other than `review.md`; writes only after approval
