# Donatello Specification

## Vision

Donatello is a Claude Code plugin that sets up and runs spec-driven project structure. It interviews you about a project, organizes the work into phases, carries your coding preferences into every project, and leaves a small, consistent context footprint so Claude can pick up work across sessions without re-explaining.

Donatello is **a set of skills and templates, not a program.** Claude does the reasoning (interviewing, phase breakdown, task suggestion); Donatello supplies the workflow, the file conventions, and your preferences.

---

## Core Concepts

### Two Components

**1. Donatello repo** (this project)
- A Claude Code plugin *and* its own marketplace (`.claude-plugin/marketplace.json` with `source: "./"`)
- Holds workflow skills, phase templates, and your preferences
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
│   ├── phase-archive/
│   │   └── SKILL.md              # /donatello:phase-archive
│   └── <domain skills>/          # e.g. compose-patterns/, added over time
├── preferences/                  # Your coding preferences, one file per category
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
├── docs/
│   ├── architecture.md           # High-level system overview and durable decisions
│   ├── requirements.md           # How the system behaves now; grows as phases are archived
│   └── preferences.md            # Your preferences, copied and tailored to this project
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

1. **Inspect** the repo: README, manifests (`build.gradle*`, `package.json`, `pyproject.toml`, …), existing `CLAUDE.md`, `docs/`, `.spec/`, git history.
2. **Interview** the user, conversationally and briefly: what's being built, current state, where it's going, which preferences apply. Skippable: "use defaults" scaffolds immediately with what was inferred.
3. **Propose** a phase breakdown (phase 1 in detail, later phases as one-liners). The user approves or edits before anything is written.
4. **Write**:
   - `docs/preferences.md` from the relevant files in `preferences/`, tailored per the interview
   - `docs/architecture.md` (existing projects: drawn from the code; new: from the interview)
   - Phase 1 via the `phase-new` workflow
   - The Donatello block in `CLAUDE.md`
5. **Never overwrite.** Existing files are left alone or merged with the user's approval; an existing `.spec/` means onboarding resumes instead of starting over.

### 2. Start a phase — `/donatello:phase-new [scope]`

1. Read `docs/`, the archive, and ongoing phases to work out the next `<N>` and context.
2. Discuss scope if it isn't clear from the argument.
3. Fill the four planning templates: requirements in `spec.md`, decisions in `design.md`, and tasks in `tasks.md` shaped by `docs/preferences.md` (e.g. test-first tasks when TDD is preferred). Note in a task when a specific skill applies. `walkthrough.md` is **not** created yet.
4. Show the result for review.

### During a phase (no command; driven by the CLAUDE.md block)

- After finishing a task, the agent ticks it in `tasks.md` and updates `walkthrough.md`, creating it from the template on the first finished task.
- At the end of a session, the agent rewrites the walkthrough's "Current state" section.

### 3. Archive a phase — `/donatello:phase-archive [phase]`

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
- Copied (not linked) into `docs/preferences.md` so each project is self-contained and can diverge
- Changing a preference in Donatello does not touch existing projects; re-run onboard to pull updates (it shows a diff)

### Skills
- Workflow skills (onboard, phase-new, phase-archive) are explicit-only (`disable-model-invocation: true`); they run when you type them
- Domain skills (e.g. Compose patterns) are model-invocable and ship in the same plugin
- Third-party skills are installed from their own marketplaces; Donatello doesn't wrap or mirror them

### Documentation split
- `docs/` answers "what is this, how does it behave, and why": architecture, requirements, preferences. No step-by-step.
- `.spec/` answers "what are we doing now": phase-scoped and actionable, and refers to `docs/` rather than repeating it.

---

## Non-Goals

- **Not a code generator.** It produces structure, specs, and tasks, not application code.
- **Not a skill mirror.** Third-party skills come from their own plugins.
- **No standalone CLI.** If something later needs deterministic tooling (e.g. validation), it ships as a script or hook inside the plugin.

---

## Success Criteria

- Installing the plugin and running `/donatello:onboard` on a new and an existing project produces `docs/`, `.spec/ongoing/phase1-*/`, and a CLAUDE.md block, with nothing overwritten
- A fresh Claude session in an onboarded project knows the current phase, its current state, and your preferences without being told
- After a phase is archived, `docs/requirements.md` reflects what it shipped
- `/donatello:phase-new` and `/donatello:phase-archive` keep numbering and naming consistent across several phases
- The Donatello block is under 300 tokens
