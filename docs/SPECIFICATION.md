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
│   │   └── templates/            # overview.md, design.md, spec.md, tasks.md
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
│   ├── architecture.md           # High-level system overview
│   └── preferences.md            # Your preferences, copied and tailored to this project
└── .spec/
    ├── ongoing/
    │   └── phase1-<slug>/
    │       ├── overview.md       # What this phase accomplishes and why
    │       ├── design.md         # Phase decisions and handoff notes
    │       ├── spec.md           # Requirements and acceptance criteria
    │       └── tasks.md          # Checklist; agent ticks items as it goes
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
3. Fill the four templates: requirements in `spec.md`, decisions in `design.md`, and tasks in `tasks.md` shaped by `docs/preferences.md` (e.g. test-first tasks when TDD is preferred). Note in a task when a specific skill applies.
4. Show the result for review.

### 3. Archive a phase — `/donatello:phase-archive [phase]`

1. Check `tasks.md`: if items are unchecked, list them and ask whether to archive anyway, carry them into the next phase, or stop.
2. Add a short "Outcome" section to `overview.md` (what shipped, what was deferred).
3. Promote durable decisions from the phase's `design.md` into `docs/architecture.md` if they matter beyond the phase.
4. Move the directory to `.spec/archive/<today>-phase<N>-<slug>/` (via `git mv` when in a git repo).

---

## Key Behaviors

### The CLAUDE.md block
- Wrapped in `<!-- donatello:start -->` / `<!-- donatello:end -->` so it can be updated in place without touching the user's own content
- Imports preferences with `@docs/preferences.md` (always relevant, so always loaded)
- Points to `.spec/ongoing/` as current work and tells the agent to keep `tasks.md` up to date
- Target: under 300 tokens for the block itself

### Preferences
- Live in `preferences/` in this repo, one Markdown file per category, each with a `summary:` frontmatter line
- Copied (not linked) into `docs/preferences.md` so each project is self-contained and can diverge
- Changing a preference in Donatello does not touch existing projects; re-run onboard to pull updates (it shows a diff)

### Skills
- Workflow skills (onboard, phase-new, phase-archive) are explicit-only (`disable-model-invocation: true`); they run when you type them
- Domain skills (e.g. Compose patterns) are model-invocable and ship in the same plugin
- Third-party skills are installed from their own marketplaces; Donatello doesn't wrap or mirror them

### Documentation split
- `docs/` answers "what is this and why": architecture, preferences. No step-by-step.
- `.spec/` answers "what are we doing now": phase-scoped and actionable, and refers to `docs/` rather than repeating it.

---

## Non-Goals

- **Not a code generator.** It produces structure, specs, and tasks, not application code.
- **Not a skill mirror.** Third-party skills come from their own plugins.
- **No standalone CLI.** If something later needs deterministic tooling (e.g. validation), it ships as a script or hook inside the plugin.

---

## Success Criteria

- Installing the plugin and running `/donatello:onboard` on a new and an existing project produces `docs/`, `.spec/ongoing/phase1-*/`, and a CLAUDE.md block, with nothing overwritten
- A fresh Claude session in an onboarded project knows the current phase and your preferences without being told
- `/donatello:phase-new` and `/donatello:phase-archive` keep numbering and naming consistent across several phases
- The Donatello block is under 300 tokens
