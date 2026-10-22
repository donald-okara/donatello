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
- Updated in two steps: `/plugin marketplace update donatello` → `/plugin update donatello@donatello` (or uninstall and reinstall if it reports current), then a new session

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
│   │   └── templates/            # overview, design, spec, tasks, walkthrough, review
│   ├── phase-discuss/
│   │   └── SKILL.md              # /donatello:phase-discuss (revise an ongoing phase's plan)
│   ├── phase-run/
│   │   └── SKILL.md              # /donatello:phase-run
│   ├── phase-review/
│   │   └── SKILL.md              # /donatello:phase-review (brief, verdict, comments into review.md)
│   ├── phase-archive/
│   │   └── SKILL.md              # /donatello:phase-archive
│   ├── ramble/
│   │   ├── SKILL.md              # /donatello:ramble (explore on a branch, notes, wrap-up)
│   │   └── templates/            # notes.md: the notes file written to .spec/rambles/
│   ├── tour/
│   │   └── SKILL.md              # /donatello:tour [area] (read-only, chat-only orientation)
│   └── skills/
│       └── SKILL.md              # /donatello:skills (catalog list, add, remove, import)
├── catalog/
│   ├── skills.yaml               # Catalog: which skills onboard and import can offer, and where each comes from
│   └── skills/
│       └── <id>/                 # Your own skill folders (SKILL.md); copied into projects on import
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
│   ├── settings.local.json       # Personal: disables plugins irrelevant to this project
│   ├── settings.json             # Committed plugin entries for catalog plugins chosen as "committed"
│   └── skills/                   # Catalog skills imported into this project (one folder each)
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
    ├── rambles/
    │   └── 2026-10-19-<slug>.md  # ramble notes: frontmatter, Question, Log, Learnings, Outcome
    └── archive/
        └── 2026-10-16-phase1-<slug>/
```

**Naming conventions**
- Ongoing phase: `.spec/ongoing/phase<N>-<kebab-slug>/`
- Archived phase: `.spec/archive/<YYYY-MM-DD>-phase<N>-<kebab-slug>/` (date = archive date)
- `<N>` continues across ongoing and archived phases; it never resets
- Ramble notes: `.spec/rambles/<YYYY-MM-DD>-<kebab-slug>.md` (date = start date); rambles have no number

---

## Workflows

### 1. Onboard a project — `/donatello:onboard`

Works on both new and existing projects.

1. **Inspect** the repo by dispatching `donatello:scout`, which reads README, manifests (`build.gradle*`, `package.json`, `pyproject.toml`, …), existing `CLAUDE.md`, `docs/`, `.spec/`, and git history, and returns a short summary. The main session doesn't read the repo itself.
2. **Interview** the user, conversationally and briefly: what's being built, current state, where it's going, which preferences apply. Skippable: "use defaults" scaffolds immediately with what was inferred. First, unless `--defaults`, it asks "Are you new to this repo?". The default comes from git authorship (no commits, an empty `user.email`, or under 5% of commits by you means yes), and the answer is always confirmed. "New" on a non-onboarded repo switches on newcomer mode: findings are shown to confirm or mark "don't know", unresolved ones go to an `## Open questions` section of the architecture doc, conflicting preferences keep the project's rule, and the wrap-up offers "docs only" or "docs + phase 1" and points to `/donatello:tour`. On an already onboarded repo, "new" gets a pointer to the tour, never newcomer mode.
3. **Propose** a phase breakdown (phase 1 in detail, later phases as one-liners). The user approves or edits before anything is written.
4. **Write**:
   - `docs/preferences.md` from the relevant general files in `preferences/`, tailored per the interview
   - `.claude/rules/<category>.md` for preference files that declare `paths` (stack- or file-specific rules)
   - `docs/architecture.md` (existing projects: drawn from the code; new: from the interview)
   - Phase 1 via the `phase-new` workflow
   - The Donatello block in `CLAUDE.md`
   - `.claude/settings.local.json`: proposes disabling installed plugins that don't fit the project's stack (shown as a list; you pick)
   - Skills chosen from the catalog (see workflow 5): `.claude/skills/<id>/` folders, plugin entries in `.claude/settings.json` (committed) or `.claude/settings.local.json` (just me), and `.gitignore` lines for "just me" folders
   - Skip the skills step with `--defaults`; the wrap-up points to `/donatello:skills import` instead
5. **Never overwrite.** Existing files are left alone or merged with the user's approval; an existing `.spec/` means onboarding resumes instead of starting over.
6. **Never duplicate.** If an existing doc already covers architecture, requirements, or conventions (any name, matched case-insensitively), onboard adopts it instead of creating `docs/<role>.md`, and the block names its path. Preferences the project already states are dropped; conflicting ones are raised in the interview, where the project's rule wins by default.

### 2. Start a phase — `/donatello:phase-new [scope]`

1. Read `docs/`, the archive, and ongoing phases to work out the next `<N>` and context.
2. Discuss scope if it isn't clear from the argument.
3. Fill the four planning templates: requirements in `spec.md`, decisions in `design.md`, and tasks in `tasks.md` shaped by `docs/preferences.md` (e.g. test-first tasks when TDD is preferred). Note in a task when a specific skill applies. `walkthrough.md` is **not** created yet.
4. Show the result for review.

### 3. Run a phase — `/donatello:phase-run [task | --all]`

The main session (Opus) coordinates; it doesn't write the code or read full diffs and build logs.

1. Pick the next unchecked task (or the one named).
2. Dispatch `donatello:implementer` (Haiku) with the task, the relevant spec requirements, and the phase path. It implements, runs quiet builds/tests, and returns a short report: files changed, test result, anything blocked.
3. Dispatch `donatello:reviewer` (Sonnet) with the task and spec requirements. It reviews the uncommitted changes (`git diff HEAD` plus untracked files) and returns PASS or a short list of issues with file:line.
4. On issues: send them back to the implementer once. If review fails again, or the implementer reports it's blocked, re-dispatch the implementer on Sonnet. If that fails too, stop and bring it to you, and if the task or spec looks wrong, suggest `/donatello:phase-discuss`. Reviewer notes that imply a plan change go into the report; phase-run never edits `tasks.md` or `spec.md` beyond ticking boxes.
5. On PASS: commit (using the task's commit line), tick the task, append to the walkthrough's "Changes by task" (creating `walkthrough.md` on the first task), refresh "Current state".
6. Stop after one task, or continue through the list with `--all` until done or a step needs you.

### 5. Manage the skill catalog — `/donatello:skills [list | add <ref> | remove <id> | import]`

The catalog (`catalog/skills.yaml` in this repo) lists skills that onboard and `import` can offer. Entries store where a skill comes from, not a copy of it, except for your own skills, which live in `catalog/skills/<id>/`.

1. **Find the catalog.** `add` and `remove` edit your Donatello clone: the current directory if it's the repo, else the path saved in `~/.claude/donatello.json`. If neither exists, `add` and `remove` ask once and save the answer. `list` and `import` never ask for a clone: they read the clone when one is found, and the installed copy otherwise.
2. **`list`** prints each entry as `id · applies-to · summary`, grouped by source.
3. **`add <ref>`** accepts four ref forms: `android:<id>`, `<plugin>@<marketplace>`, a GitHub folder URL, or a local folder with `SKILL.md`. It fills `id` and `summary` from the source, refuses duplicate ids, suggests `applies-to`, shows the entry, and writes only on approval. Git refs are pinned to a commit SHA.
4. **`remove <id>`** confirms, then deletes the entry (and `catalog/skills/<id>/` for your own skills).
5. After `add` or `remove`, it offers to commit and push in the clone. Nothing is committed without a yes.
6. **`import`** runs four parts, which onboard can also run separately:
   - **Candidates:** entries whose `applies-to` matches the project's stack (or has none), minus skills already present
   - **Ask:** all / pick / none, then committed / just me
   - **Install:** per source, into `.claude/skills/<id>/` or the plugin settings. Never replaces an existing folder or settings value
   - **Summary:** imported, already present, failed (each with a fallback command), and the committed paths

### Working without phase-run (driven by the CLAUDE.md block)

- After finishing a task, the agent ticks it in `tasks.md` and updates `walkthrough.md`, creating it on the first finished task.
- At the end of a session, the agent rewrites the walkthrough's "Current state" section.

### 4. Archive a phase — `/donatello:phase-archive [phase]`

1. Check `tasks.md`: if items are unchecked, list them and ask whether to archive anyway, carry them into the next phase, or stop. Then check the last `## Review` heading in `review.md`: if there is no review or the latest verdict is changes requested, it says so and asks whether to archive anyway or stop; an approved review continues without asking. The gate is soft: it never blocks.
2. Finalize `walkthrough.md`: rewrite "Current state" as a closing summary (what shipped, what was deferred, where carried-over tasks went) and make sure "How to verify" is complete. `Carried over:` also lists the `Deferred:` items in `overview.md`.
3. Promote what outlives the phase, showing the diff for approval:
   - durable decisions from `design.md` → `docs/architecture.md`
   - requirements from `spec.md` that describe lasting behavior (not one-off migration or setup steps), adjusted for any "Deviations from spec" → `docs/requirements.md`, grouped by area, replacing older entries they supersede
4. Move the directory to `.spec/archive/<today>-phase<N>-<slug>/` (via `git mv` when in a git repo).

### 6. Ramble — `/donatello:ramble [topic | end]`

Explores a question on a branch or in a worktree, with teaching on, then decides what survives. Rambles are not phases: they have no number, spec, tasks, or review loop, and `phase-archive` doesn't touch them.

1. **Start** (a topic, or "new" in the picker): the slug is kebab-case, at most 4 words, and rejected if `.spec/rambles/<date>-<slug>.md` or branch `ramble/<slug>` already exists. Claude asks **worktree** or **in place** every time.
   - **Worktree:** a worktree at `../<repo>-ramble-<slug>` on a new branch `ramble/<slug>`. The notes file is committed on main as `docs: start ramble <slug>` before the branch has any work on it.
   - **In place:** `git switch -c ramble/<slug>` on a clean tree. The notes file stays untracked until the ramble ends.
   - **Not a git repo:** the notes file only, with no branch, commits, or outcome.
2. **Picker** (no argument): offers to continue the current ramble when on `ramble/*`, then lists open and parked rambles, orphan `ramble/*` branches (their notes file is created on resume), and open rambles whose branch is gone (which can be marked `abandoned`). A new ramble is the last choice.
3. **Resume:** reads only the notes frontmatter, the Question, and the last 5 Log lines, then switches to the ramble's worktree or branch. A parked ramble becomes open again.
4. **Ramble mode** (see Key Behaviors) runs until the ramble ends.
5. **End** (`/donatello:ramble end`): wraps up the current ramble. With `end` outside a ramble, the picker lists open and parked rambles to choose from.
   1. **Tidy:** uncommitted changes are checkpointed as `wip:` or discarded after confirmation. Nothing is dropped silently.
   2. **Learnings:** a draft of at most 10 lines, drawn from the Log and the branch's commits, for the user to edit.
   3. **Skills to the catalog:** chosen skill folders from the ramble are added with `/donatello:skills add`, run from main. A new `SKILL.md` can be drafted from the Learnings first.
   4. **Outcome:** one of five:
      - **throwaway:** branch and worktree deleted; status `closed`
      - **kept:** branch and worktree stay; status `parked`
      - **merged:** squash-merged into main as one commit with an approved message; branch and worktree deleted; status `closed`
      - **PR:** branch pushed and a pull request opened with `gh`; branch and worktree stay; status `closed`
      - **promoted to phase:** `phase-new` runs with the notes as context; status `closed`. A throwaway choice is applied after the phase is created.
   5. **Close-out:** back to main, run the catalog adds, write Learnings and Outcome, and commit the notes on main (`docs: close ramble <slug>`, or `docs: park ramble <slug>` for kept). The branch action runs last, so the notes are safe if it fails.

### 7. Revise a phase — `/donatello:phase-discuss [topic]`

Changes the ongoing phase's plan through a conversation, then applies one diff that you approve. Ticked tasks are never changed, and task and requirement numbers are never reused or renumbered.

1. **Load:** uses the one ongoing phase in `.spec/ongoing/` (asks when there are several; suggests `/donatello:phase-new` and stops when there are none). Archived phases are never revised, and on a `ramble/*` branch it refuses. It reads the overview, spec, design, tasks, and the walkthrough's "Current state" if it exists, and notes which tasks are ticked and which requirements each task serves. It reads no source code; a fact about the code comes from a `donatello:scout` report.
2. **Discuss:** starts from the topic or asks what to change, one question at a time, in chat. If a change contradicts the overview's goal it says so. If the request is really new work, it offers to record it as a `Deferred:` line in the overview instead of adding tasks.
3. **Propose:** builds one proposal from the discussion:
   - Unticked tasks can be edited, split, merged, deleted, reordered, or deferred. New and changed tasks follow the "Writing tasks" rules in phase-new.
   - A change to a ticked task is a new task titled `Task <N>: <title> (revises Task <M>)`.
   - A new task takes the next free number. A deleted task keeps its heading as `(removed: <reason>)` and a deferred one as `(deferred)`; these stubs have no checkboxes and are left out of the dependency sketch, so their numbers stay in the file.
   - An added requirement takes the next free R-number. A dropped one keeps its heading as `(dropped <YYYY-MM-DD>)`.
   - Tasks citing a changed or dropped requirement are listed. Unticked ones are revised in the same diff; a ticked one gets a revisit task, or a recorded "no change needed" if you agree.
   - A requirement or scope change adds a dated `design.md` entry (Decision / Why / Tradeoff, citing R-numbers). A task-only reshape adds nothing beyond the change itself.
   - A deferral becomes a `(deferred)` stub and a `Deferred:` line under the overview's Out of Scope. If `walkthrough.md` exists, its "Current state" is rewritten; "Deviations from spec" is never touched.
4. **Check and apply:** before showing anything, it checks that no ticked task's text changed, that heading numbers are unique and every `Serves:` ID exists in `spec.md`, that every live requirement is served by a live task, and that the dependency sketch lists exactly the live tasks. It shows a summary table of the changes (what, why, what it affects) followed by the detailed diff across the phase files, and writes nothing until you approve. On approval it stages and commits only the phase directory (`docs: revise phase <N> plan (<summary>)`), leaving other uncommitted changes as they are. On decline it writes nothing and summarises the discussion in chat.

### 8. Tour a repo — `/donatello:tour [area]`

A read-only orientation for someone new to a repo, given in six stops (what it is, the map, how to run it, how code is written, where work stands, where to start). It writes nothing: no files, notes, or commits.

1. **Load:** on an onboarded repo (Donatello block in `CLAUDE.md` or `AGENTS.md`), reads only the architecture, requirements, and preferences docs the block names, `.claude/rules` names and paths, each ongoing phase's overview and walkthrough "Current state", archived phase names, and ramble frontmatter. Otherwise it dispatches `donatello:scout` and works from its report. No source files are read while loading.
2. **Stops:** one at a time, each about 10 lines, ending "questions, deeper, or next?". Stop 5 (where work stands) is skipped on a non-onboarded repo. Stop 6 points to `/donatello:phase-run` (or `/donatello:phase-new` with no ongoing phase) when onboarded, and to `/donatello:onboard` when not.
3. **Area:** with an argument, every stop is limited to that area. If the loaded docs don't cover it, the scout is dispatched with that path as its focus. If nothing matches, it says so and asks for another area.
4. **Navigation:** "next", "skip to <stop>", and "deeper". "Deeper" may read specific files, which are the only code reads in the tour.
5. **Facts:** only from the docs, the scout report, or files read for "deeper"; anything else is "unknown".

The tour is model-invocable (it leaves model invocation on), since it writes nothing and triggering on "show me around" is harmless.

### 9. Review a phase — `/donatello:phase-review [phase]`

Records a maintainer's review of a finished phase in `<phase>/review.md`. It's a record: no tasks, plan files, or code change, and no agents run.

1. **Load:** uses the named phase or the one in `.spec/ongoing/` (asks when there are several). Refuses on a `ramble/*` branch. Reads `tasks.md`; if any boxes are unticked, asks **review anyway** (the round is recorded as `(partial)`) or **stop**.
2. **Brief:** about 15 lines in chat: the goal and success criteria from `overview.md`, the verification, deviations, and current state from `walkthrough.md`, the commits by task from git (skipped outside a git repo), and the latest earlier round if `review.md` exists. No source files or diffs are read.
3. **Verdict and comments:** the maintainer gives **approved** or **changes requested**, then any number of comments, including none. Each comment is recorded as one bullet in the maintainer's words. Claude adds none of its own.
4. **Record:** creates `review.md` from the template if missing, numbers the round after the highest existing one, and dates it. It shows the round and writes only after approval, appending it and leaving earlier rounds unchanged. The reviewer name comes from `git config user.name`, asked when unset.
5. **Wrap up:** offers to commit `docs: review phase <N> (<verdict>)` with a pathspec limited to the phase directory. The next step is `/donatello:phase-archive` when approved, or `/donatello:phase-discuss` or `/donatello:phase-run` when changes are requested.

---

## Key Behaviors

### The CLAUDE.md block
- Wrapped in `<!-- donatello:start -->` / `<!-- donatello:end -->` so it can be updated in place without touching the user's own content
- Imports preferences with `@docs/preferences.md` (always relevant, so always loaded)
- Points to `.spec/ongoing/` as current work and tells the agent to keep `tasks.md` and `walkthrough.md` up to date
- Rambles: on a `ramble/*` branch, the agent reads only the frontmatter, Question, and last 5 Log entries of that ramble's notes in `.spec/rambles/` (matched by `branch`) and follows ramble mode; `/donatello:ramble` resumes
- Lists the workflow commands, including `/donatello:phase-review`, `/donatello:ramble`, and `/donatello:tour`
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
- Workflow skills are explicit-only, so they cost nothing in the skill listing (except `tour`, whose short description is listed); long detail lives in supporting files loaded on demand
- Always-loaded context is capped: CLAUDE.md block < 300 tokens, `docs/preferences.md` ≤ 100 lines; anything file-specific goes to path-scoped rules
- The CLAUDE.md block rarely changes, so the prompt cache stays warm; the walkthrough is deliberately not imported
- On resume, only the walkthrough's "Current state" is read, which makes `/clear` between tasks cheap
- Verbose work (repo surveys, builds, tests, Gradle output) happens inside agents; only short reports come back
- Agents have `maxTurns` caps; reviewer and scout are read-only
- Onboard offers to disable plugins irrelevant to the project, since every enabled plugin's skill descriptions are listed every turn

### Skills
- Workflow skills (onboard, phase-new, phase-discuss, phase-run, phase-review, phase-archive, skills, ramble) are explicit-only (`disable-model-invocation: true`); they run when you type them. `tour` is the exception: it writes nothing, so Claude may start it on "show me around"-style requests
- Your own domain skills (e.g. Compose patterns) live in `catalog/skills/<id>/`, not in `skills/`, so they don't load in every project. `/donatello:skills import` and onboard copy the chosen ones into the project's `.claude/skills/<id>/`, the same way preferences are copied
- Third-party skills are referenced in `catalog/skills.yaml` (Android CLI skills, plugins, and GitHub skill folders pinned to a commit) and installed through their own channels; Donatello doesn't wrap or mirror them

### Ramble mode
- Runs in the main session. No implementer or reviewer; `donatello:scout` runs only when the user asks for a survey
- Teaching is on: explain why things work or break, and compare designs by their trade-offs. Tests, previews, and naming rules are relaxed unless the experiment is about them
- Main is never touched except the notes file. Experiments happen on `ramble/<slug>` or in its worktree
- Checkpoints are `wip: <what>` commits. Changes are staged by path, never with `git add -A`, and the notes file is never staged on the branch
- Each thing learned, design tried, dead end, or answered question adds one terse Log line (`HH:MM — <what>`), with no code or raw output
- Building a skill: suggests `.claude/skills/<id>/SKILL.md`, so it loads and can be tested in the same ramble
- Rambles feed the phase workflow: `phase-new` reads the Learnings of recently closed rambles and records the decisions they settled, naming the ramble. `phase-run` refuses to run on a `ramble/*` branch, and its clean-tree check ignores `.spec/rambles/`

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
