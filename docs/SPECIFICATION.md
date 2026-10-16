# Donatello Specification

## Vision

Donatello is a Claude Code plugin that sets up and runs spec-driven project structure. It interviews you about a project, organizes the work into phases, carries your coding preferences into every project, and leaves a small, consistent context footprint so Claude can pick up work across sessions without re-explaining.

Donatello is **a set of skills, agents, and templates, not a program.** Claude does the reasoning (interviewing, phase breakdown, task suggestion, implementation); Donatello supplies the workflow, the file conventions, your preferences, and a model-routing setup that spends Opus on thinking and cheaper models on legwork.

---

## Core Concepts

### Two Components

**1. Donatello repo** (this project)
- A Claude Code plugin *and* its own marketplace (`.claude-plugin/marketplace.json` with `source: "./"`)
- Holds workflow skills, subagents, phase templates, and your preferences
- Installed once per machine: `/plugin marketplace add donald-okara/donatello` → `/plugin install donatello@donatello`

**2. Per-project structure**
- Created by `/donatello:onboard` in any project
- `docs/` for long-lived orientation, `.spec/` for phase work, a marked block in `CLAUDE.md` as the entry point

### Directory Structure

#### Donatello repo

```
donatello/
├── .claude-plugin/
│   ├── plugin.json
│   └── marketplace.json
├── skills/
│   ├── onboard/
│   │   └── SKILL.md              # /donatello:onboard
│   ├── phase-new/
│   │   ├── SKILL.md              # /donatello:phase-new
│   │   └── templates/            # overview, design, spec, tasks, walkthrough
│   ├── phase-run/
│   │   └── SKILL.md              # /donatello:phase-run
│   ├── phase-archive/
│   │   └── SKILL.md              # /donatello:phase-archive
│   └── <domain skills>/          # e.g. compose-patterns/, added over time
├── agents/
│   ├── implementer.md            # donatello:implementer (Haiku): does one task
│   ├── reviewer.md               # donatello:reviewer (Sonnet): read-only review against spec
│   └── scout.md                  # donatello:scout (Haiku): read-only repo survey
├── preferences/                  # Your coding preferences, one file per category
│   ├── model-usage.md
│   ├── testing.md
│   ├── naming.md
│   └── ...
├── templates/
│   └── claude-block.md           # The block inserted into a project's CLAUDE.md
├── docs/
│   ├── SPECIFICATION.md          # This file
│   └── ARCHITECTURE.md
└── .spec/                        # Donatello's own phases (dogfooding)
    ├── ongoing/
    └── archive/
```

#### A project after `/donatello:onboard`

```
project/
├── CLAUDE.md                     # Existing content preserved; Donatello block added
├── .claude/
│   ├── rules/                    # Path-scoped preferences (e.g. android-ui.md, paths: **/*.kt)
│   └── settings.local.json       # Personal: disables plugins irrelevant to this project
├── docs/
│   ├── architecture.md           # High-level system overview and durable decisions
│   ├── requirements.md           # How the system behaves now; grows as phases are archived
│   └── preferences.md            # General preferences (always loaded; ≤ 100 lines)
└── .spec/
    ├── ongoing/
    │   └── phase1-<slug>/
    │       ├── overview.md       # Intent: what this phase accomplishes and why
    │       ├── design.md         # Decisions only (Decision / Why / Tradeoff)
    │       ├── spec.md           # Requirements and acceptance criteria
    │       ├── tasks.md          # Checklist; agent ticks items as it goes
    │       └── walkthrough.md    # Record: current state, how to verify, what changed (after first task)
    └── archive/
        └── 2026-10-16-phase1-<slug>/
```

**Naming conventions**
- Ongoing phase: `.spec/ongoing/phase<N>-<kebab-slug>/`
- Archived phase: `.spec/archive/<YYYY-MM-DD>-phase<N>-<kebab-slug>/` (date = archive date)
- `<N>` continues across ongoing and archived phases; it never resets

---

## Workflows

### 1. Onboard a project — `/donatello:onboard`

Works on both new and existing projects.

1. **Inspect** the repo by dispatching `donatello:scout`, which reads README, manifests (`build.gradle*`, `package.json`, `pyproject.toml`, …), existing `CLAUDE.md`, `docs/`, `.spec/`, and git history, and returns a short summary. The main session doesn't read the repo itself.
2. **Interview** the user, conversationally and briefly: what's being built, current state, where it's going, which preferences apply. Skippable: "use defaults" scaffolds immediately with what was inferred.
3. **Propose** a phase breakdown (phase 1 in detail, later phases as one-liners). The user approves or edits before anything is written.
4. **Write**:
   - `docs/preferences.md` from the relevant general files in `preferences/`, tailored per the interview
   - `.claude/rules/<category>.md` for preference files that declare `paths` (stack- or file-specific rules)
   - `docs/architecture.md` (existing projects: drawn from the code; new: from the interview)
   - Phase 1 via the `phase-new` workflow
   - The Donatello block in `CLAUDE.md`
   - `.claude/settings.local.json`: proposes disabling installed plugins that don't fit the project's stack (shown as a list; you pick)
5. **Never overwrite.** Existing files are left alone or merged with the user's approval; an existing `.spec/` means onboarding resumes instead of starting over.

### 2. Start a phase — `/donatello:phase-new [scope]`

1. Read `docs/`, the archive, and ongoing phases to work out the next `<N>` and context.
2. Discuss scope if it isn't clear from the argument.
3. Fill the four planning templates: requirements in `spec.md`, decisions in `design.md`, and tasks in `tasks.md` shaped by `docs/preferences.md` (e.g. test-first tasks when TDD is preferred). Note in a task when a specific skill applies. `walkthrough.md` is **not** created yet.
4. Show the result for review.

### 3. Run a phase — `/donatello:phase-run [task | --all]`

The main session (Opus) coordinates; it doesn't write the code or read full diffs and build logs.

1. Pick the next unchecked task (or the one named).
2. Dispatch `donatello:implementer` (Haiku) with the task, the relevant spec requirements, and the phase path. It implements, runs quiet builds/tests, and returns a short report: files changed, test result, anything blocked.
3. Dispatch `donatello:reviewer` (Sonnet) with the task, spec requirements, and the commit range. It returns PASS or a short list of issues with file:line.
4. On issues: send them back to the implementer once. If review fails again, or the implementer reports it's blocked, re-dispatch the implementer on Sonnet. If that fails too, stop and bring it to you.
5. On PASS: commit (using the task's commit line), tick the task, append to the walkthrough's "Changes by task" (creating `walkthrough.md` on the first task), refresh "Current state".
6. Stop after one task, or continue through the list with `--all` until done or a step needs you.

### Working without phase-run (driven by the CLAUDE.md block)

- After finishing a task, the agent ticks it in `tasks.md` and updates `walkthrough.md`, creating it on the first finished task.
- At the end of a session, the agent rewrites the walkthrough's "Current state" section.

### 4. Archive a phase — `/donatello:phase-archive [phase]`

1. Check `tasks.md`: if items are unchecked, list them and ask whether to archive anyway, carry them into the next phase, or stop.
2. Finalize `walkthrough.md`: rewrite "Current state" as a closing summary (what shipped, what was deferred, where carried-over tasks went) and make sure "How to verify" is complete.
3. Promote what outlives the phase, showing the diff for approval:
   - durable decisions from `design.md` → `docs/architecture.md`
   - requirements from `spec.md` that describe lasting behavior (not one-off migration or setup steps), adjusted for any "Deviations from spec" → `docs/requirements.md`, grouped by area, replacing older entries they supersede
4. Move the directory to `.spec/archive/<today>-phase<N>-<slug>/` (via `git mv` when in a git repo).

---

## Key Behaviors

### The CLAUDE.md block
- Wrapped in `<!-- donatello:start -->` / `<!-- donatello:end -->` so it can be updated in place without touching the user's own content
- Imports preferences with `@docs/preferences.md` (always relevant, so always loaded)
- Points to `.spec/ongoing/` as current work and tells the agent to keep `tasks.md` and `walkthrough.md` up to date
- On resume, the agent reads only the walkthrough's "Current state" section, not the whole file
- Target: under 300 tokens for the block itself

### Walkthrough
- The record of what actually happened in a phase, kept separate from intent (`overview.md`, `spec.md`) and reasoning (`design.md`)
- Sections:
  - **Current state**: ≤ 10 lines, rewritten each session; the only part read on resume
  - **How to verify**: commands to run, screens to open, previews or screenshots to check; what a reviewer needs to accept the work without replaying it
  - **Changes by task**: appended as tasks finish; files touched and notable details
  - **Deviations from spec**: what changed relative to `spec.md` and why
- Replaces handoff notes in `design.md` and the Outcome section in `overview.md`
- Written by the agent; finalized by `phase-archive`

### Living requirements
- `docs/requirements.md` describes how the system behaves *now*. Archived phases keep their own `spec.md` as history, so nobody has to read the archive to learn current behavior.
- Created on the first archive (or seeded by onboard for existing projects when behavior is clear from the code)
- Entries are plain requirement statements grouped by area; superseded entries are replaced, not kept alongside

### Preferences
- Live in `preferences/` in this repo, one Markdown file per category, each with a `summary:` frontmatter line
- Copied (not linked) into the project so each project is self-contained and can diverge:
  - general files → `docs/preferences.md`, imported by the CLAUDE.md block, so always loaded. Kept ≤ 100 lines
  - files with `paths` frontmatter → `.claude/rules/<category>.md`, loaded only when Claude touches matching files
- Changing a preference in Donatello does not touch existing projects; re-run onboard to pull updates (it shows a diff)

### Model routing
- Set out in `preferences/model-usage.md`, which ships by default:
  - **Opus** (main session): brainstorming, interviews, planning, coordinating, architecture questions
  - **Sonnet**: reviews; fallback implementer for tasks Haiku can't finish
  - **Haiku**: implementation of well-specified tasks, repo surveys, other legwork
- Enforced through the plugin's agents (`model:` in their frontmatter persists for the whole run), not through skill `model:` settings, which last only one turn
- A project can override any agent by adding `.claude/agents/<name>.md`, which takes priority over plugin agents
- Haiku implementation relies on precise tasks; the reviewer step and the escalation rule are what keep it safe, so both are always on

### Token discipline
Built into the workflow rather than left to habit:
- Workflow skills are explicit-only, so they cost nothing in the skill listing; long detail lives in supporting files loaded on demand
- Always-loaded context is capped: CLAUDE.md block < 300 tokens, `docs/preferences.md` ≤ 100 lines; anything file-specific goes to path-scoped rules
- The CLAUDE.md block rarely changes, so the prompt cache stays warm; the walkthrough is deliberately not imported
- On resume, only the walkthrough's "Current state" is read, which makes `/clear` between tasks cheap
- Verbose work (repo surveys, builds, tests, Gradle output) happens inside agents; only short reports come back
- Agents have `maxTurns` caps; reviewer and scout are read-only
- Onboard offers to disable plugins irrelevant to the project, since every enabled plugin's skill descriptions are listed every turn

### Skills
- Workflow skills (onboard, phase-new, phase-run, phase-archive) are explicit-only (`disable-model-invocation: true`); they run when you type them
- Domain skills (e.g. Compose patterns) are model-invocable and ship in the same plugin
- Third-party skills are installed from their own marketplaces; Donatello doesn't wrap or mirror them

### Documentation split
- `docs/` answers "what is this, how does it behave, and why": architecture, requirements, preferences. No step-by-step.
- `.spec/` answers "what are we doing now": phase-scoped and actionable, and refers to `docs/` rather than repeating it.

---

## Non-Goals

- **Not a code generator.** Donatello contains no generation logic; `phase-run` hands tasks to Claude agents, which write the code.
- **Not a skill mirror.** Third-party skills come from their own plugins.
- **No standalone CLI.** If something later needs deterministic tooling (e.g. validation), it ships as a script or hook inside the plugin.

---

## Success Criteria

- Installing the plugin and running `/donatello:onboard` on a new and an existing project produces `docs/`, `.spec/ongoing/phase1-*/`, and a CLAUDE.md block, with nothing overwritten
- A fresh Claude session in an onboarded project knows the current phase, its current state, and your preferences without being told
- After a phase is archived, `docs/requirements.md` reflects what it shipped
- `/donatello:phase-new` and `/donatello:phase-archive` keep numbering and naming consistent across several phases
- `/donatello:phase-run` completes a task with Haiku implementing and Sonnet reviewing, and the main session never loads full diffs or build logs
- The Donatello block is under 300 tokens; `docs/preferences.md` is ≤ 100 lines
