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
- At least `testing.md`, `naming.md`, and one stack-specific file, written from your real practice
- **Check:** every file has valid frontmatter; content is rules, not essays

### R4. `onboard` skill
- Explicit-only (`disable-model-invocation: true`); supports a `--defaults` argument to skip the interview
- Steps: inspect → interview → propose phases → write (on approval)
- Writes `docs/preferences.md` (filtered by `applies-to`), `docs/architecture.md`, `docs/requirements.md` (existing projects, when behavior is clear), phase 1 (following phase-new's steps and templates), and the CLAUDE.md block
- Follows every row of the Edge Cases table in ARCHITECTURE.md
- **Check:** on a project with an existing CLAUDE.md and README, those files keep their original content byte-for-byte outside the Donatello markers

### R5. `phase-new` skill
- Explicit-only; optional `[scope]` argument
- Works out N as max(ongoing, archived) + 1; slug from scope (kebab-case, ≤ 4 words)
- Task order and content follow `docs/preferences.md`; tasks name a skill when one applies
- Does not create `walkthrough.md`
- **Check:** with `phase1-*` archived and nothing ongoing, creates `phase2-<slug>/` with all four files filled

### R6. `phase-archive` skill
- Explicit-only; optional `[phase]` argument (defaults to the only ongoing phase, otherwise asks)
- Handles unchecked tasks (archive / carry over / stop); finalizes `walkthrough.md` (or builds one from tasks and git history if missing); promotes durable decisions to `docs/architecture.md` and lasting requirements (adjusted for deviations) to `docs/requirements.md`, with a diff for approval; `git mv` into `.spec/archive/<YYYY-MM-DD>-phase<N>-<slug>/`
- **Check:** after archiving, the directory exists under archive with today's date, the walkthrough's Current state is a closing summary, and `docs/requirements.md` contains the phase's lasting requirements without duplicating superseded ones

### R6a. Walkthrough upkeep
- Driven by the CLAUDE.md block: after each task, tick it and update the walkthrough (create on first task); rewrite Current state at session end
- **Check:** in a fixture, finishing one task with no prompting beyond the task creates `walkthrough.md` with all four sections, and Current state is ≤ 10 lines

### R7. Repo self-consistency
- README describes install and the three commands
- Donatello's own `.spec/` follows its own naming conventions
- **Check:** running `/donatello:phase-archive` on this repo at the end of phase 1 works

## Non-Functional Requirements

- **NF1. Small context footprint:** CLAUDE.md block < 300 tokens; each SKILL.md body under ~500 lines, with detail pushed into supporting files
- **NF2. Safety:** no file is overwritten or deleted without showing the change and getting approval
- **NF3. Portability:** no shell-specific commands in skills beyond `git`; works on macOS and Linux
