# Phase 1: Walkthrough

## Current state

Tasks 1–5 are done. Agents are verified on their models; `/donatello:phase-new` writes correctly numbered phases whose tasks follow the preferences, including path-scoped rules (e.g. `:core:ui` preview wrappers first). Next: Task 6, the `onboard` skill (scout → interview → propose → write, with the preferences split, plugin pruning, and the CLAUDE.md block).

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

## Deviations from spec

- `tasks.md` template adds a Serves / Files / Skill header per task (Task 3); not in the original template description, added so agents need less context.
