# Phase 1: Specification

## Requirements

### R1. Plugin packaging
- `.claude-plugin/plugin.json`: `name: "donatello"`, version, description, author, repository
- `.claude-plugin/marketplace.json`: marketplace `donatello`, one plugin with `source: "./"`
- **Check:** `claude plugin validate .` passes; `claude --plugin-dir .` lists `/donatello:onboard`, `/donatello:phase-new`, `/donatello:phase-archive`

### R2. Templates
- `skills/phase-new/templates/`: `overview.md`, `design.md`, `spec.md`, `tasks.md`, `walkthrough.md` as described in ARCHITECTURE.md
- `overview.md` has no Outcome section and `design.md` has no handoff notes; both moved to the walkthrough
- `templates/claude-block.md`: marked block with the `@docs/preferences.md` import, pointers, and the walkthrough upkeep rules (sections named inline)
- `tasks.md` uses plain `- [ ]` checkboxes only (no status emoji)
- **Check:** the block is under 300 tokens; templates contain section headings and short guidance comments, not filler content

### R3. Preferences
- `preferences/*.md`, one per category, each with `category`, `summary`, optional `applies-to` frontmatter
- `model-usage.md` ships by default: Opus for brainstorming/planning (main session), Sonnet for review and fallback implementation, Haiku for implementation and legwork, plus the escalation rule
- At least `testing.md`, `naming.md`, and one stack-specific file with `paths`, written from your real practice
- **Check:** every file has valid frontmatter; content is rules, not essays; general files (no `paths`) total ≤ 100 lines

### R4. `onboard` skill
- Explicit-only (`disable-model-invocation: true`); supports a `--defaults` argument to skip the interview
- Steps: inspect (via `donatello:scout`) → interview → propose phases → write (on approval)
- Writes `docs/preferences.md` (general preferences filtered by `applies-to`), `.claude/rules/<category>.md` (preferences with `paths`), `.claude/settings.local.json` (plugins you chose to disable), `docs/architecture.md`, `docs/requirements.md` (existing projects, when behavior is clear), phase 1 (following phase-new's steps and templates), and the CLAUDE.md block
- Follows every row of the Edge Cases table in ARCHITECTURE.md
- **Check:** on a project with an existing CLAUDE.md and README, those files keep their original content byte-for-byte outside the Donatello markers

### R5. `phase-new` skill
- Tasks are written precisely enough for a Haiku implementer: files involved, expected behavior, how to check
- Explicit-only; optional `[scope]` argument
- Works out N as max(ongoing, archived) + 1; slug from scope (kebab-case, ≤ 4 words)
- Task order and content follow `docs/preferences.md`; tasks name a skill when one applies
- Does not create `walkthrough.md`
- **Check:** with `phase1-*` archived and nothing ongoing, creates `phase2-<slug>/` with all four files filled

### R5a. Agents
- `implementer` (`model: haiku`, `maxTurns: 40`), `reviewer` (`model: sonnet`, `effort: medium`, read-only plus Bash, `maxTurns: 20`), `scout` (`model: haiku`, read-only, `maxTurns: 25`), as in the Agents table in ARCHITECTURE.md
- Builds and tests run quietly; reports have fixed shapes and length caps
- **Check:** each agent runs on its declared model; a failing build yields only error lines in the report

### R5b. `phase-run` skill
- Explicit-only; `[task | --all]`; runs on the session model
- Loop: implementer → reviewer → one fix round → re-dispatch implementer on Sonnet → stop and ask
- On PASS: commit with the task's commit line, tick the task, update the walkthrough
- Passes paths and requirement IDs to agents, not file contents
- **Check:** after one task, the main session's context contains agent reports but no full diffs or build logs; an underspecified task reaches escalation

### R6. `phase-archive` skill
- Explicit-only; optional `[phase]` argument (defaults to the only ongoing phase, otherwise asks)
- Handles unchecked tasks (archive / carry over / stop); finalizes `walkthrough.md` (or builds one from tasks and git history if missing); promotes durable decisions to `docs/architecture.md` and lasting requirements (adjusted for deviations) to `docs/requirements.md`, with a diff for approval; `git mv` into `.spec/archive/<YYYY-MM-DD>-phase<N>-<slug>/`
- **Check:** after archiving, the directory exists under archive with today's date, the walkthrough's Current state is a closing summary, and `docs/requirements.md` contains the phase's lasting requirements without duplicating superseded ones

### R6a. Walkthrough upkeep
- Done by `phase-run` automatically; when working without it, the CLAUDE.md block drives it: after each task, tick it and update the walkthrough (create on first task); rewrite Current state at session end
- **Check:** in a fixture, finishing one task with no prompting beyond the task creates `walkthrough.md` with all four sections, and Current state is ≤ 10 lines

### R7. Repo self-consistency
- README describes install and the three commands
- Donatello's own `.spec/` follows its own naming conventions
- **Check:** running `/donatello:phase-archive` on this repo at the end of phase 1 works

## Non-Functional Requirements

- **NF1. Small context footprint:** CLAUDE.md block < 300 tokens; `docs/preferences.md` ≤ 100 lines; each SKILL.md body under ~500 lines, with detail in supporting files; agent descriptions one or two sentences
- **NF4. Model routing:** no skill depends on a `model:` frontmatter field for multi-turn behavior; model choice is enforced through agents
- **NF2. Safety:** no file is overwritten or deleted without showing the change and getting approval
- **NF3. Portability:** no shell-specific commands in skills beyond `git`; works on macOS and Linux
