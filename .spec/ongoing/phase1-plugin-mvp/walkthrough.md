# Phase 1: Walkthrough

## Current state

Task 1 is done: the plugin validates, installs at project scope, and loads 4 stub skills and 3 stub agents (~405 always-on tokens). Relative paths from skills to `preferences/` work. Next: Task 2, writing real preferences (`model-usage`, testing, naming, one path-scoped stack file). This needs Donald's input, since they encode real practice.

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

## Deviations from spec

- None yet.
