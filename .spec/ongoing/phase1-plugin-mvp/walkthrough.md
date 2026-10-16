# Phase 1: Walkthrough

## Current state

Tasks 1–2 are done. The plugin validates and loads (4 stub skills, 3 stub agents). Real preferences are written: `model-usage`, `testing`, `naming` (general, 43 lines total) and `android-ui` (path-scoped to `**/*.kt`). Next: Task 3, the phase templates and the CLAUDE.md block.

## How to verify

- `claude plugin validate .` and `claude plugin validate .claude-plugin/plugin.json` both pass
- In a scratch git repo: `claude plugin marketplace add <path-to-donatello> --scope project`, then `claude plugin install donatello@donatello --scope project`, then `claude plugin details donatello@donatello`
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
- General files (no `paths`) total 43 lines, under the 100-line cap

## Deviations from spec

- None yet.
