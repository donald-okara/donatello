# Phase 1: Walkthrough

## Current state

Tasks 1–8 are done: all four skills and three agents are implemented and verified in fixtures. Carried to Task 9 (real projects, interactive): the fix round and Sonnet escalation in phase-run, onboard's approval and plugin-pruning paths, one continuous new → run → archive → new loop, Next: Task 9, which needs Donald to onboard a real Android project. The relative-path check and token cost were measured after the GitHub push: the cached copy holds `preferences/` and `templates/`, and always-on cost is ~391 tokens (on-invoke: onboard ~2.1k, phase-new ~1.5k, phase-run ~1.3k, phase-archive ~1.2k, agents ~0.5–0.8k).

## How to verify

- `claude plugin validate .` and `claude plugin validate .claude-plugin/plugin.json` both pass
- In a scratch git repo: `claude plugin marketplace add <path-to-donatello> --scope project`, then `claude plugin install donatello@donatello --scope project`, then `claude plugin details donatello@donatello`
- Agent models: in a scratch repo with the plugin installed, run `echo "Use the donatello:scout agent ..." | claude -p --model sonnet --output-format stream-json --verbose` and check that subagent messages (non-null `parent_tool_use_id`) report `claude-haiku-*`; for the reviewer, use `--model haiku` as the main model and expect `claude-sonnet-*`
- Typing `/donatello:` shows onboard, phase-new, phase-run, phase-archive (stubs reply that they aren't implemented)

## Changes by task

### Task 1: Plugin manifests and skeleton
- Added `.claude-plugin/plugin.json` (v0.1.0) and `.claude-plugin/marketplace.json` (marketplace `donatello`, `source: "./"`)
- Added stub skills `skills/{onboard,phase-new,phase-run,phase-archive}/SKILL.md`, all `disable-model-invocation: true`
- Added stub agents: `implementer` (haiku, `disallowedTools: Agent`, maxTurns 40), `reviewer` (sonnet, effort medium, Read/Grep/Glob/Bash, maxTurns 20), `scout` (haiku, Read/Grep/Glob/Bash, maxTurns 25)
- Path probe: a temporary skill read `../../preferences/probe.md` under `--plugin-dir` and under a project-scope install; both worked (probe removed)
- Prompts for `claude -p` with `--plugin-dir` must go through stdin; the flag swallows a trailing positional argument

### Task 2: Preferences
- `preferences/model-usage.md`: Opus / Haiku / Sonnet roles, escalation rule, cost habits
- `preferences/testing.md`: test results not implementation; JUnit, Mockito, Robolectric; Compose Preview Screenshot Testing; previews count as tests
- `preferences/naming.md`: Route + Content for screens, `[Title][Type]` for components
- `preferences/android-ui.md` (`paths: ["**/*.kt"]`): Compose only; previews mandatory, at the bottom, covering theme / font scale / screen size / `@PreviewParameter`; hoisted state; MVI components take only the fields they need
- `android-ui.md` revised: preview wrappers in `:core:ui` (component: theme + Surface + padding; screen: theme + full-size Surface, no padding, mirroring the nav host; scene wrappers for Navigation 3 dialogs/sheets/panes), per-project grouped multipreview annotations, and wrappers + annotations as the first `:core:ui` task
- General files (no `paths`) total 43 lines, under the 100-line cap

### Task 3: Templates
- `skills/phase-new/templates/`: `overview.md` (intent only), `design.md` (Decision / Why / Tradeoff), `spec.md` (R-numbered with a Check each, NFRs), `tasks.md` (Serves / Files / Skill header per task, steps written for a Haiku implementer, Check and Commit lines), `walkthrough.md` (four sections)
- Guidance lives in HTML comments so it doesn't render, and the agent removes it when filling a template
- `templates/claude-block.md`: marked block, ~208 tokens (833 chars / 4)

### Task 4: Agents
- `agents/implementer.md`: one task only; reads task, cited requirements, preferences; quiet builds (log to `$TMPDIR`, grep failures); never ticks, edits `.spec/`, or commits; fixed STATUS/FILES/CHECKS/NOTES/BLOCKER report
- `agents/reviewer.md`: read-only; reviews uncommitted changes (`git diff HEAD` plus untracked files); judges task, requirement Checks, preferences, correctness; PASS or ISSUES list; non-blocking points only as NOTE; flags environment failures without asking for code changes
- `agents/scout.md`: read-only survey that samples code rather than reading it all; fixed 8-heading report, 60 lines at most
- Verified in a fixture with a fake `gradlew` (400 noise lines plus an error): implementer ran on Haiku and returned BLOCKED with errors only (largest tool result 684 chars, about $0.12); reviewer ran on Sonnet, stayed read-only, and returned ISSUES (about $0.05, then $0.03 after tightening); scout ran on Haiku with a 55-line report (about $0.12)
- Tightened after the first run: paths relative to the repo root; non-blocking points moved to NOTE; environment-failure rule
- Docs fixed: the reviewer gets uncommitted changes, not a commit range (phase-run commits only after PASS)

### Task 5: `phase-new` skill
- `skills/phase-new/SKILL.md`: gather context cheaply (preferences, architecture, requirements, **all `.claude/rules/*.md`**, directory names only, plus the latest archived walkthrough's Current state) → compute N and slug → settle scope (no interview when the argument is specific) → fill templates and strip HTML comments → summary review → offer to commit
- Task-writing rules for a Haiku implementer: Serves/Files/Skill header, 3–6 steps, named functions and results, explicit test steps per preferences, UI per rules, exact Check, conventional commit, dependency ordering
- Found while testing: path-scoped rules don't load during planning (no `.kt` file is touched), so phase-new now reads `.claude/rules/` explicitly
- Verified in two fixtures with the real preferences (Sonnet main session, ~$0.21 each): an archived `phase1-*` gave `phase2-habit-list-screen`; a project with no `.spec/` gave `phase1-project-setup`; no walkthrough created; no HTML comments left; preview wrappers ordered first; result-based test steps; Route/Content split; real installed skills named (`android-cli`, `navigation-3`)
- Tightened: omit the Skill line instead of writing "none"

### Task 6: `onboard` skill
- `skills/onboard/SKILL.md`: check existing state (resume mode, AGENTS.md case) → scout (main session doesn't read the code) → pick preferences by `applies-to` and sort general vs `paths` → short interview → one proposal → write (preferences with date header, rules with `paths`-only frontmatter, architecture under 80 lines with exact build/test commands, requirements only when behavior is clear, phase 1 via phase-new, CLAUDE.md block, plugin pruning via `claude plugin disable <id> --scope local`) → wrap up and offer a commit
- `--defaults` writes new files without approval but never changes existing content, never replaces a block, and skips pruning
- Verified (Sonnet main session, ~$0.21–0.24 per onboard):
  - empty repo: scout skipped, phase1-project-foundation, CLAUDE.md created
  - existing Compose repo with its own CLAUDE.md: scout on Haiku, **zero** main-session file reads, original CLAUDE.md intact byte-for-byte with one block appended, its team rules carried into phase 1 tasks
  - re-run: resume mode, nothing changed (~$0.13)
  - fresh Haiku session with read tools names the active phase and its scope (~$0.007)
  - path-scoped rule: present after reading a `.kt` file, absent (NONE) after reading README.md
- Verified separately: `claude plugin disable <id> --scope local` writes `.claude/settings.local.json` and disables a synced plugin for that project only
- Fixed after testing: no placeholder `docs/requirements.md` for new projects; block heading renamed to "Spec workflow (Donatello plugin)" because a Haiku session mistook the project for "Donatello"
- Housekeeping: test fixtures are now uninstalled (`claude plugin uninstall --scope project`) before deletion; 4 stale entries from earlier tests were cleaned up

### Task 7: `phase-run` skill
- `skills/phase-run/SKILL.md`: coordinator only (no source reads, diffs, builds); requires a clean tree; picks the next ready task; implementer → reviewer → fix round → re-dispatch on Sonnet via the Agent tool's `model` → stop; environment failures and missing decisions go to the user; on PASS, tick, update walkthrough (create from template), commit with the task's line; `--all` loops
- Verified in a Python fixture (Sonnet main session):
  - `--all` on 2 tasks: Haiku implemented, Sonnet reviewed, 2 commits with the right messages, all boxes ticked, walkthrough created with all four sections; main session made no source reads or diffs (largest tool result 1,203 chars); $0.29 total
  - Reviewer NOTE surfaced a real spec gap (`format_amount(-5)` → `"$-1.95"`), carried into Current state
  - Trap tasks (requirement missing from task text; task text contradicting requirement): Haiku followed the requirements both times, so the fix round never ran (~$0.20)
  - Without phase-run, the CLAUDE.md block alone got Claude to implement, tick, update the walkthrough, and commit, and it flagged that review was skipped (~$0.18)
- Tightened: implementer follows requirements over conflicting task text and notes it; walkthrough "Deviations from spec" holds only real spec deviations

### Task 8: `phase-archive` skill
- `skills/phase-archive/SKILL.md`: pick phase → unticked tasks (archive anyway / carry over / stop) → finalize walkthrough (closing Current state with a `Carried over:` list that phase-new reads), or build it from tasks + git log/stat when missing → promote lasting decisions to `docs/architecture.md` `## Decisions` and lasting requirements (adjusted for deviations, superseded entries replaced) to `docs/requirements.md`, as one diff for approval → `git mv` with literal paths to `.spec/archive/<date>-<dir>` → offer commit
- Verified (Sonnet main session, ~$0.12–0.15 per run):
  - finished phase: promoted only the cross-phase decision (int cents), skipped the phase-local one (regex); created requirements.md with the padding deviation applied; skipped the one-off setup requirement; archived and committed
  - second phase changing format_amount: the existing requirement entry was **replaced** (merged), not duplicated
  - unticked tasks: listed them, offered the three choices, moved nothing
  - missing walkthrough: rebuilt from tasks and git history, and flagged a real gap (the phase added no tests)

## Deviations from spec

- `tasks.md` template adds a Serves / Files / Skill header per task (Task 3); not in the original template description, added so agents need less context.
