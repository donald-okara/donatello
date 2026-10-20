# Donatello Architecture

## Overview

Donatello is a Claude Code plugin with no runtime code. It has four parts:

1. **Workflow skills**: instructions Claude follows (`onboard`, `phase-new`, `phase-discuss`, `phase-run`, `phase-review`, `phase-archive`, `skills`, `ramble`, `tour`)
2. **Agents**: subagents with fixed models for legwork and review (`implementer`, `reviewer`, `scout`)
3. **Templates**: the shape of the files the skills write
4. **Data**: your preferences, the skill catalog (`catalog/skills.yaml`), and your own skill folders (`catalog/skills/`)

Claude is the engine: it reads the target project, runs the interview, fills the templates, and writes the code. The skills set the steps, guardrails, and output format; the agents decide which model does which work.

---

## Plugin Layout

```
.claude-plugin/
  plugin.json           # name: "donatello", version, author
  marketplace.json      # one plugin, source: "./"
skills/
  onboard/SKILL.md
  phase-new/SKILL.md
  phase-new/templates/{overview,design,spec,tasks,walkthrough,review}.md
  phase-discuss/SKILL.md
  phase-run/SKILL.md
  phase-review/SKILL.md
  phase-archive/SKILL.md
  ramble/SKILL.md
  ramble/templates/notes.md
  tour/SKILL.md
  skills/SKILL.md
agents/
  implementer.md
  reviewer.md
  scout.md
preferences/*.md
catalog/
  skills.yaml           # the skill catalog: one entry per offerable skill
  skills/<id>/SKILL.md  # your own skills (only folders with a donatello source)
templates/claude-block.md
```

Skills and agents are namespaced by the plugin: `skills/onboard/` is invoked as `/donatello:onboard`, `skills/skills/` as `/donatello:skills`, and `agents/implementer.md` is the `donatello:implementer` agent.

### Referencing files across the plugin

Claude Code gives a skill its base directory, so skills reference supporting files with relative paths:
- `phase-new` → `templates/overview.md`
- `onboard` → `../../preferences/`, `../../templates/claude-block.md`, `../phase-new/` for phase 1, and `../skills/SKILL.md` for the import flow
- `skills` → `../../catalog/skills.yaml` (the installed copy, when no clone is found) and `../../catalog/skills/<id>/`
- `ramble` → `templates/notes.md`, and `../skills/SKILL.md` for the catalog adds at wrap-up
- `phase-discuss` → `../phase-new/SKILL.md` for the "Writing tasks" rules, which it follows without restating
- `phase-review` → `../phase-new/templates/review.md` for the file it creates when `review.md` is missing

Templates live in `phase-new` because that is their only writer; `onboard` delegates to it, so there is one source of truth for phase files. The same holds for the import flow: it is defined once in `skills/skills/SKILL.md`, and onboard follows it.

---

## Components

### `onboard` skill
- **Frontmatter:** `disable-model-invocation: true`, `argument-hint: "[--defaults]"`
- **Reads:** the `donatello:scout` report on the target repo (not the repo directly), `preferences/`, the skill catalog (via the import's Candidates step), the list of enabled plugins
- **Writes:** `docs/preferences.md` (general preferences), `.claude/rules/<category>.md` (preferences with `paths`), `docs/architecture.md`, `docs/requirements.md` (existing projects, only when behavior is clear from the code), phase 1 (via phase-new steps), the CLAUDE.md block, `.claude/skills/<id>/` and the plugin entries for chosen catalog skills (skills step, 6g), `.gitignore` lines for "just me" imports, and `.claude/settings.local.json` with the plugins you chose to disable
- **Newcomer mode:** the first question is "Are you new to this repo?", defaulting to yes when there's no git repo or no commits, `user.email` is empty, or under 5% of commits are yours (always asked, so a wrong guess costs one answer). On a non-onboarded repo, "new" makes findings confirm-or-"don't know" in the interview, writes unresolved architecture questions under a final `## Open questions` section (at most 5 lines), keeps the project's rule on conflicts without asking, offers "docs only" or "docs + phase 1" at the proposal, and offers `/donatello:tour` at the wrap-up. On an onboarded repo, "new" gets a pointer to the tour and the usual resume mode; newcomer mode never runs there
- **Guardrails:** propose before writing; never overwrite; never duplicate (adopt existing docs for the architecture, requirements, and preferences roles, and drop preferences the project already states); resume if `.spec/` already exists; ask rather than guess when the project type is ambiguous (monorepos); skills step runs before plugin pruning so pruning never proposes an imported plugin; `--defaults` skips the newcomer question and skill import

### `skills` skill
- **Frontmatter:** `disable-model-invocation: true`, `argument-hint: "[add <ref> | remove <id> | list | import]"`
- **Reads:** `catalog/skills.yaml` from the clone when one is found (cwd or `~/.claude/donatello.json`), else the installed copy; `list` and `import` never ask for a clone; the Android CLI's `skills list` and the marketplace files under `~/.claude/plugins/` (to fill `add` entries); `.claude/skills/`, `.claude/settings*.json` (to find what is already present)
- **Writes:** on approval only: a catalog entry (and `catalog/skills/<id>/` for your own skills) in the clone; for `import`, `.claude/skills/<id>/`, plugin entries in `.claude/settings.json` or `.claude/settings.local.json`, and `.gitignore` lines for "just me"
- **Clone path:** `~/.claude/donatello.json` (`{"clone": "<path>"}`), asked once for add and remove only
- **Import shape:** four separable parts (Candidates, Ask, Install, Summary) so onboard can run Candidates and Ask as its own questions and Install and Summary later
- **Guardrails:** refuses duplicate ids; pins git refs to a commit SHA; fetches git folders with `GIT_TERMINAL_PROMPT=0` into a `mktemp` directory that is always removed; never replaces an existing `.claude/skills/<id>/` folder or settings value; never edits the installed plugin copy; never commits or pushes without a yes

### `phase-new` skill
- **Frontmatter:** `disable-model-invocation: true`, `argument-hint: "[scope]"`
- **Reads:** `docs/`, all `.claude/rules/*.md` (path-scoped rules don't load during planning, since no matching file is touched), `.spec/ongoing/`, `.spec/archive/` (to find the next N and prior context), and the frontmatter of `.spec/rambles/*.md`. Closed rambles with a `closed` date after the newest archive date (all of them if there is no archive) have their Learnings read, at most the 5 most recent; a promoted ramble's full notes are read
- **Writes:** `.spec/ongoing/phase<N>-<slug>/` from the four templates; decisions settled by the read ramble Learnings go in `design.md`, naming the ramble
- **Task shaping:** follows `docs/preferences.md` (test-first ordering, preview tasks, etc.); a task names a skill when one applies

### `phase-discuss` skill
- **Frontmatter:** `disable-model-invocation: true`, `argument-hint: "[topic]"`; no `model` (it runs in the main session)
- **Reads:** the ongoing phase's `overview.md`, `spec.md`, `design.md`, `tasks.md`, and the "Current state" section of `walkthrough.md`; the current branch name (for the `ramble/*` check). No source code; a fact about the code comes from a `donatello:scout` report
- **Writes:** on approval only, the phase's `spec.md`, `tasks.md`, `design.md` (dated entries for requirement or scope changes), `overview.md` (`Deferred:` lines under Out of Scope), and the walkthrough's "Current state". Then `git add <phase dir>` and a commit whose pathspec is the phase directory: `docs: revise phase <N> plan (<summary>)`
- **Numbering:** new tasks and requirements take the next free number; removed tasks stay as `(removed: …)` or `(deferred)` stubs and dropped requirements as `(dropped <date>)` stubs; no number is reused or renumbered, and the dependency sketch lists only live tasks
- **Guardrails:** ticked task text is never changed (rework is a `(revises Task M)` task); the consistency checks run before the diff is shown; nothing is written before approval, and a decline writes nothing; `git add -A` is never used, so uncommitted source changes stay uncommitted; refuses on a `ramble/*` branch; only `.spec/ongoing/` is considered, never archived phases

### `phase-run` skill
- **Frontmatter:** `disable-model-invocation: true`, `argument-hint: "[task | --all]"`; no `model` (it runs on the session model, Opus)
- **Reads:** the phase's `tasks.md` and the spec requirements the task references; agent reports. Never full diffs or build logs.
- **Loop per task:** implementer → reviewer → (one fix round) → escalate the implementer to Sonnet via the Agent tool's per-call `model` → stop and ask
- **Writes:** commit per task, tick in `tasks.md`, walkthrough "Changes by task" and "Current state"
- **Handoff to agents:** passes paths, not content (phase directory, `docs/preferences.md`, requirement IDs), so each agent reads only what it needs
- **Stop step:** when a task or spec looks wrong, suggests `/donatello:phase-discuss`; reviewer `NOTE:` lines that imply a plan change go into its report
- **Guardrails:** refuses on a `ramble/*` branch. Its clean-tree check and its commits both ignore `.spec/rambles/`, since worktree rambles write their notes into the main checkout. It never edits `tasks.md` or `spec.md` beyond ticking boxes

### `phase-review` skill
- **Frontmatter:** `disable-model-invocation: true`, `argument-hint: "[phase]"`; no `model` (it runs in the main session)
- **Reads:** the phase's `tasks.md` (unticked check), `overview.md`, `walkthrough.md` (each only if present), `review.md` (latest `## Review` round), and git log and `--stat` output for the commit list. No source files, diffs, or agents
- **Writes:** on approval only, one `## Review <n>` round appended to `<phase>/review.md`, created from `phase-new`'s template if missing. Then an optional commit `docs: review phase <N> (<verdict>)` with the phase directory as its pathspec
- **Guardrails:** a record only: no task, spec, design, or walkthrough change, and no comment becomes a task; earlier rounds are never edited; the round number is one above the highest existing one; refuses on a `ramble/*` branch; a partial review (unticked tasks) is marked `(partial)`; the reviewer name comes from `git config user.name`, asked when unset; outside git, no commit offer and no brief commit list

### `phase-archive` skill
- **Frontmatter:** `disable-model-invocation: true`, `argument-hint: "[phase]"`
- **Reads:** the phase's `tasks.md`, `review.md` (latest round's verdict), `design.md`, `spec.md`, `overview.md` (for `Deferred:` lines), `walkthrough.md`
- **Gate:** soft. Missing review or changes requested asks "archive anyway / stop"; approved continues. It never blocks, and the wrap-up states the review outcome
- **Writes:** final "Current state" summary in `walkthrough.md`, with `Carried over:` also listing the `Deferred:` items from `overview.md`; promoted decisions in `docs/architecture.md`; promoted requirements (adjusted for deviations) in `docs/requirements.md`; then moves the directory to `.spec/archive/<date>-phase<N>-<slug>/`

### `ramble` skill
- **Frontmatter:** `disable-model-invocation: true`, `argument-hint: "[topic | end]"`; no `model` (it runs in the main session)
- **Reads:** `git worktree list --porcelain` and `git branch --list 'ramble/*'` to find the main checkout and the rambles; the frontmatter of each `.spec/rambles/*.md` in the main checkout (picker); on resume, the Question and last 5 Log lines; on end, the Log and `git log main..ramble/<slug>` (Learnings draft); `templates/notes.md` at start; `../skills/SKILL.md` for the catalog adds. `donatello:scout` only when the user asks for a survey
- **Writes:** `.spec/rambles/<date>-<slug>.md` in the main checkout (status, closed, Learnings, Outcome); branch `ramble/<slug>` and worktree `../<repo>-ramble-<slug>`; `wip:` checkpoints on the ramble branch; on main, only the notes file, catalog adds, and the squash commit for a merge. Commit messages: `docs: start|resume|abandon|close|park ramble <slug>`, `docs: record PR for ramble <slug>`, `docs: record promotion of ramble <slug>`. Skill folders are copied to `$TMPDIR/donatello-ramble-<slug>/` and removed after the adds
- **Guardrails:** main is touched only through the notes file and the commits above, and only from the main checkout on `main` (asks first if main isn't checked out there); the notes file is never staged on the branch; changes are staged by path, never `git add -A`; a slug is never reused; uncommitted changes are checkpointed or discarded only after a yes; merge, PR, and promote each need an explicit yes; worktrees are removed without `--force`; a failed branch action is reported and not retried; no implementer or reviewer agents

### `tour` skill
- **Frontmatter:** `argument-hint: "[area]"`; no `disable-model-invocation`, so Claude may start it on "show me around"-style requests (no `model`; it runs in the main session)
- **Reads:** on an onboarded repo (Donatello block in `CLAUDE.md` or `AGENTS.md`): the architecture, requirements, and preferences docs at the block's paths, the file names and `paths` frontmatter of `.claude/rules/*.md`, each ongoing phase's `overview.md` and the "Current state" of its `walkthrough.md`, directory names in `.spec/archive/`, and ramble frontmatter. Otherwise, the `donatello:scout` report. No source file is read while loading; "deeper" may read specific files
- **Writes:** nothing. No files, notes, or commits; chat only. It ends by saying nothing was changed
- **Stops:** six, one at a time, each about 10 lines and ending "questions, deeper, or next?". Stop 5 is skipped on a non-onboarded repo. Stop 6's next step: onboarded, the Current state's next step then `/donatello:phase-run` (or `/donatello:phase-new` with no ongoing phase, or the overview's goal when there's no walkthrough); not onboarded, `/donatello:onboard`
- **Area:** `$ARGUMENTS` limits every stop to that area. Uncovered by the docs: the scout is dispatched once with the area path as its focus. No match: say so and ask for another
- **Dispatch:** `donatello:scout` (Haiku, read-only); a brief may name a focus path, which narrows what it reads but not the report format
- **Guardrails:** read-only and chat-only; facts come only from the docs, the scout report, or files read for "deeper", and anything else is "unknown"; the only code reads are "deeper" reads

### Walkthrough upkeep (no skill)
- Driven by the CLAUDE.md block, not a command, so it happens during normal work
- On the first finished task the agent creates `walkthrough.md`. The CLAUDE.md block can't reference plugin paths, so it names the four sections inline; `skills/phase-new/templates/walkthrough.md` is the fuller version that `phase-archive` uses when it builds a missing walkthrough.

### Agents

| Agent | Model | Tools | maxTurns | Returns |
|---|---|---|---|---|
| `implementer` | `haiku` | All except Agent | 40 | ≤ 15 lines: files changed, build/test result, blockers |
| `reviewer` | `sonnet`, `effort: medium` | Read, Grep, Glob, Bash (git, tests) | 20 | `PASS`, or ≤ 10 issues as `file:line: problem` |
| `scout` | `haiku` | Read, Grep, Glob, Bash (read-only commands) | 25 | ≤ 60 lines: stack, modules, build/test commands, observed conventions, existing docs |

`scout` is dispatched by `onboard` and by `tour` (on a non-onboarded repo, or for an area the docs don't cover). A focused survey keeps the same report format; only the reading narrows.

Shared rules in each agent's prompt:
- Follow `docs/preferences.md` and any `.claude/rules/` that load
- Run builds and tests in quiet mode (e.g. `./gradlew -q`, `--console=plain`) and report only failures, never full logs
- Implementer: do only the given task; don't tick tasks, edit `.spec/` files, or commit (phase-run does all three after review)
- Reviewer: judge against the task, spec requirements, and preferences; no style nitpicks beyond the preferences

### Templates
| File | Content |
|---|---|
| `overview.md` | Goal, what the phase delivers, out of scope, success criteria (intent only, not updated after planning) |
| `design.md` | Decisions as *Decision / Why / Tradeoff*, and nothing else |
| `spec.md` | Numbered requirements (R1…), each with scope and a verifiable acceptance check |
| `tasks.md` | Plain `- [ ]` checklist grouped by task; no status emoji, since the checkbox *is* the status |
| `walkthrough.md` | Current state (≤ 10 lines) / How to verify / Changes by task / Deviations from spec |
| `claude-block.md` | The marked CLAUDE.md block (see below) |
| `ramble/templates/notes.md` | Ramble notes: frontmatter (`status`, `mode`, `branch`, `worktree`, `started`, `closed`), then Question, Log, Learnings, Outcome. HTML-comment guidance is removed when a ramble starts |

### CLAUDE.md block (shape)
```markdown
<!-- donatello:start -->
## Spec workflow (Donatello plugin)
- Preferences: @docs/preferences.md (file-specific rules live in .claude/rules/)
- Models: plan and discuss in the main session; implement via donatello:implementer, review via donatello:reviewer, survey via donatello:scout
- Orientation: architecture in docs/architecture.md, requirements in docs/requirements.md (read when needed)
- Current work: .spec/ongoing/. Read the active phase's overview, spec, tasks, and the "Current state" section of walkthrough.md
- After each task: tick it in tasks.md and update the phase's walkthrough.md (create if missing, with sections: Current state, How to verify, Changes by task, Deviations from spec)
- End of session: rewrite walkthrough.md "Current state"
- Rambles: on `ramble/*`, read only the frontmatter, Question and last 5 Log entries of that ramble's notes in `.spec/rambles/` (frontmatter `branch` matches); follow ramble mode; `/donatello:ramble` resumes
- Commands: /donatello:phase-new · /donatello:phase-discuss · /donatello:phase-run · /donatello:phase-review · /donatello:phase-archive · /donatello:ramble · /donatello:tour
<!-- donatello:end -->
```

### Preferences
- One file per category with frontmatter:
  ```markdown
  ---
  category: testing
  summary: Test-first; unit tests next to code; fakes over mocks
  applies-to: [android, web, python]   # optional; omit = all
  paths: ["**/*.kt"]                   # optional; present = path-scoped rule
  ---
  ```
- `onboard` picks files whose `applies-to` matches the project:
  - no `paths` → joined into `docs/preferences.md` in sections (always loaded; keep the total ≤ 100 lines)
  - with `paths` → written to `.claude/rules/<category>.md` with the `paths` frontmatter, so it loads only when Claude touches matching files
- `model-usage.md` ships by default: Opus for brainstorming/planning in the main session, Sonnet for review and fallback implementation, Haiku for implementation and legwork, plus the escalation rule. It's the human-readable version of what the agents' frontmatter enforces.

---

## Design Decisions

### Plugin instead of CLI
- **Why:** interviewing, phase breakdown, and task suggestion are reasoning tasks. A rule-based CLI would imitate them badly; Claude already does them well.
- **Tradeoff:** output varies more between runs than a templating CLI's would. Mitigated by strict templates and guardrails in each skill.

### Preferences are copied, not linked
- **Why:** projects stay self-contained and can diverge; no lookup at runtime; works for collaborators who don't have Donatello installed.
- **Tradeoff:** drift. Re-running onboard offers a diff against the current preferences.

### Walkthrough as a separate record
- **Why:** intent (overview, spec), reasoning (design), and what actually happened are different kinds of content; mixing them made `design.md` and `overview.md` change after planning. A walkthrough also gives reviewers a "how to verify" path. This follows Antigravity's artifacts (task list / implementation plan / walkthrough).
- **Tradeoff:** one more file. Mitigated by creating it lazily and reading only "Current state" on resume.

### Living requirements in `docs/requirements.md`
- **Why:** without it, current behavior can only be pieced together from archived specs. This borrows OpenSpec's always-current `specs/` without its ADDED/MODIFIED/REMOVED delta format; the agent merges in prose at archive time.
- **Tradeoff:** merges are judgment calls, so they're shown as a diff for approval.

### Model routing through agents, not skill frontmatter
- **Why:** a skill's `model:` lasts only for the turn it's invoked in, so a multi-turn interview or task loop would drift back to the session model. An agent's `model:` holds for its whole run, and verbose work stays in the agent's context.
- **Tradeoff:** Haiku can miss on underspecified tasks. The reviewer step and the escalation to Sonnet are always on to cover this; the precise tasks that phase-new writes help too.

### Path-scoped rules for stack preferences
- **Why:** `@imports` load at launch, so everything in `docs/preferences.md` costs tokens every session. `.claude/rules/` files with `paths` load only when matching files are touched.
- **Tradeoff:** preferences are split across two places; onboard writes both, and the CLAUDE.md block points to the rules directory.

### Disabling irrelevant plugins per project
- **Why:** every enabled plugin's skill descriptions are listed on every turn. On a machine with many plugins, most are noise in any one project.
- **Where:** `.claude/settings.local.json` (personal, not committed), so teammates' setups aren't affected.

### Marked block in CLAUDE.md
- **Why:** many projects already have a CLAUDE.md; the markers let Donatello update its part idempotently.

### Adopt existing docs; the block holds the paths
- **Why:** a fact written in two files drifts, and duplicated rules in always-loaded files cost tokens every session.
- **How:** onboard maps each role (architecture, requirements, preferences) to an existing file when one covers it and writes the paths into the block. Skills and agents read them from there, with `docs/*.md` as defaults.

### Catalog stores references, not copies
- **Why:** Android CLI skills and plugins have their own install and update channels; copying them into Donatello would fork them and go stale.
- **Tradeoff:** importing needs the network and the relevant tools (`android`, `git`). Failures are skipped and listed with the command to run later.

### Own skills live outside `skills/`
- **Why:** everything in the plugin's `skills/` directory loads in every project. Keeping domain skills in `catalog/skills/` means a project pays only for the skills it imported, and matches how preferences are copied rather than linked.
- **Tradeoff:** copies drift from the catalog version. Accepted for now; a later `import --update` could refresh them.

### Edits go to your clone, not the installed plugin
- **Why:** the installed plugin is a cached copy that a marketplace update overwrites. The clone is the source of truth and syncs across machines through git.
- **Tradeoff:** one-time setup of the clone path. Other machines see new entries only after a push and a `/plugin marketplace update`; reading from the clone when one is configured hides that lag on the machine where you added them.

### Scope is asked per import
- **Why:** team projects want shared skills, and personal experiments in shared repos shouldn't leak into teammates' setups.
- **Where:** "committed" writes `.claude/skills/` and `.claude/settings.json` (meant to be committed); "just me" writes the same folders and `.claude/settings.local.json`, then lists them in `.gitignore`.
- **Tradeoff:** one more question per onboard.

### Git skills are pinned to a commit
- **Why:** imports are reproducible, and an upstream change can't silently alter a project's instructions.
- **Tradeoff:** no automatic updates. To move the pin, `remove` the entry and `add` the URL again, since `add` refuses an existing id.

### Plugins install whole
- **Why:** Claude Code enables plugins, not individual skills inside them.
- **Tradeoff:** a plugin can bring more skills than wanted; plugin pruning and the import summary make that visible.

### The import flow lives in the `skills` skill
- **Why:** one definition of the flow for onboard and for later `/donatello:skills import` runs; keeps onboard's always-read text short.
- **Tradeoff:** onboard depends on a sibling skill's path (`../skills/`), the same pattern as `../phase-new/`.

### Repo is both plugin and marketplace
- **Why:** one repo and one install command, with no separate marketplace repo to maintain.
- **Tradeoff:** splitting into several plugins later means restructuring into subdirectories.

### Phase templates live in `phase-new`
- **Why:** one writer and one source of truth for phase files. Onboard reuses phase-new's steps for phase 1.
- **Tradeoff:** onboard depends on a sibling skill's path (`../phase-new/`).

### One plugin to start
- **Why:** simplest to install and iterate on.
- **Later:** split domain skills into per-stack plugins (`donatello-android`, `donatello-web`) in the same marketplace so onboard can enable only the relevant ones via the project's `.claude/settings.json`.

### One `ramble` command with modes
- **Why:** one thing to remember, matching `/donatello:skills` subcommands. The picker makes pending rambles visible instead of relying on the branch you happen to be on.
- **Tradeoff:** a longer skill file. The wrap-up sits behind an explicit `end`, so it isn't triggered by accident.

### Rambles are not phases
- **Why:** the point is low ceremony. Folding rambles into phases would muddle numbering and bring back the process a ramble skips.
- **Tradeoff:** a second kind of record in `.spec/`; `phase-new` has to read it explicitly.

### Notes always survive; code survives only by choice, decided at `end`
- **Why:** you don't know whether an experiment is worth keeping until it's done, but what was learned is worth keeping either way. Every ramble commits its notes to main.
- **Tradeoff:** every ramble leaves a file on main, even trivial ones. Kept small: the Log is terse and Learnings are at most 10 lines.

### Worktree or in place, asked each time
- **Why:** worktrees keep main free for phase work in parallel; in place avoids a second checkout and a cold build.
- **Tradeoff:** two code paths. In place relies on the notes file never being staged on the branch, so ramble mode stages by path.

### Ramble mode runs in the main session, preferences relaxed
- **Why:** it's a learning tool, so the reasoning should stay visible, and experiments shouldn't pay for conventions they may throw away.
- **Tradeoff:** kept code needs cleanup. Merge squashes the history, and promote-to-phase redoes the work properly.

### Catalog adds run from main
- **Why:** inside the Donatello repo, `skills` treats the current directory as the clone, so an add on the ramble branch would be deleted with a throwaway branch. Chosen folders are copied to a temp directory and added from main, without changing `skills`.
- **Tradeoff:** a temp copy step, removed after the add.

### Draft a skill from the Learnings
- **Why:** a ramble often teaches something skill-worthy without a skill having been written. A draft goes through the same catalog path.
- **Tradeoff:** a draft is only as good as the notes, so it's shown for edit before anything is added.

### Merge squashes
- **Why:** `wip:` checkpoints are for rolling back during the ramble, not for main's history.
- **Tradeoff:** the checkpoint history is lost on main; choose "kept" if it matters.

### Branch action runs last
- **Why:** if deleting or merging fails, the notes and catalog adds are already safe on main.
- **Tradeoff:** a failed branch action leaves the notes recording the chosen outcome. It's reported, and not retried without asking.

### `phase-run` refuses on a ramble branch
- **Why:** phase work mixed into an experiment branch would be lost with a throwaway.
- **Tradeoff:** phase-run's clean-tree check excludes `.spec/rambles/`, so notes that worktree rambles write into the main checkout don't block phase work.

### Talk first, then one diff (phase-discuss)
- **Why:** it matches onboard and phase-archive (propose, then write), and one diff shows spec, design, and task changes together.
- **Tradeoff:** more turns than editing by hand. A hand-edit made beforehand is still caught by the consistency checks.

### Ticked tasks are frozen
- **Why:** each commit keeps matching the task text it was made for, and the walkthrough's "Changes by task" stays true.
- **Tradeoff:** a phase can grow `(revises Task M)` tasks instead of tidy edits.

### Task numbers are never reused or renumbered
- **Why:** commits, the walkthrough, and reviewer notes cite task numbers. Removed and deferred tasks keep their heading as a stub, and the dependency sketch gives the order.
- **Tradeoff:** the file keeps numbers that no longer have a live task.

### Where the reason goes
- **Why:** `design.md` already collects decisions made during the phase, and phase-archive promotes the lasting ones. After a revision the spec matches the plan, so it isn't a deviation.
- **Tradeoff:** task-only changes leave no in-file trace beyond git history.

### Deferred work lives in the overview
- **Why:** a walkthrough may not exist yet when a phase is revised, but the overview always does, and phase-new already reads `Carried over:`.

### Commit only the phase files
- **Why:** its main use is right after phase-run stops mid-task, when uncommitted changes are expected and belong to the user.

### Review the phase, not each task
- **Why:** a phase is the unit the maintainer cares about, and per-task sign-off would stall `phase-run --all`. Per-task checks stay with `donatello:reviewer`.
- **Tradeoff:** problems in early tasks surface only at the end of the phase.

### Record only (phase-review)
- **Why:** the maintainer decides what a comment means; `phase-discuss` already handles plan changes.
- **Tradeoff:** acting on comments is a separate, manual step.

### Separate `review.md`, appended by round
- **Why:** the walkthrough is Claude's record and gets rewritten in places; the maintainer's judgment stays separate and keeps its history.
- **Tradeoff:** one more file per phase. A maintainer who prefers to write by hand can edit `review.md` directly.

### Guided review in chat
- **Why:** matches the other commands, and the brief saves the maintainer from assembling context across five files.
- **Tradeoff:** Claude writes the file, so the verdict is typed in chat rather than edited in place.

### Soft archive gate
- **Why:** keeps the maintainer in control, consistent with archive's handling of unticked tasks.
- **Tradeoff:** an unreviewed phase can still be archived after one answer.

### phase-run suggests, never edits the plan
- **Why:** plan changes need the user's agreement and a recorded reason. Editing `tasks.md` and `spec.md` by hand mid-run is what phase-discuss replaces.

### A separate tour, not an onboard mode
- **Why:** onboard writes shared files and is already long; a read-only tour is safe to repeat and to run on any repo.
- **Tradeoff:** one more command in the block and the README.

### Chat only
- **Why:** a shared doc would duplicate the architecture doc, and the docs plus the scout already hold the facts; repeating the tour is cheap.
- **Tradeoff:** a newcomer who wants notes has to ask for them.

### Model-invocable tour
- **Why:** it writes nothing, so triggering on "show me around" is harmless and useful. It is the only workflow skill without `disable-model-invocation`.
- **Tradeoff:** its description is always loaded (~30 tokens).

### Newcomer default from git authorship
- **Why:** it's the cheapest signal available, and the question is still asked, so a wrong guess costs one answer.
- **Tradeoff:** shared or changed emails can skew the guess.

### "Don't know" goes to Open questions
- **Why:** the team can fill them in later, and the agents don't build on guesses.
- **Tradeoff:** the architecture doc carries a short unresolved list until someone answers it.

---

## Edge Cases

| Case | Behavior |
|---|---|
| CLAUDE.md exists, no block | Append the block; leave other content alone |
| CLAUDE.md has a block | Replace only between the markers, after showing a diff |
| `.spec/` exists | Resume: report what's there, offer to add missing pieces |
| `docs/preferences.md` exists | Show a diff against current preferences; merge only with approval |
| Existing doc covers a role under another name (`docs/DESIGN.md`, ADRs, `CONTRIBUTING.md`) | Adopt it: no new file; the block names its path; additions go in as a diff |
| Architecture or requirements only described inside CLAUDE.md / AGENTS.md | Create the default file anyway (those files load every session), without repeating what they say |
| CLAUDE.md / rules already state a preference | Drop that rule; on conflict the project wins unless the user switches |
| File name differs only in case (`ARCHITECTURE.md`) | Treated as the same file |
| Monorepo / ambiguous stack | Ask which part(s) to onboard |
| Archiving with unchecked tasks | List them; archive anyway, carry over, or stop |
| Archiving with no `walkthrough.md` | Build one from `tasks.md` and git history since the phase started; ask about "How to verify" |
| Archiving with no `review.md`, or changes requested | Says so and asks: archive anyway or stop. Approved continues. Never blocks |
| Review: not a git repo | No commit list in the brief and no commit offer; the round is still recorded |
| Review: unticked tasks | Lists them; "review anyway" records a `(partial)` round, "stop" leaves the phase unreviewed |
| Review: no `user.name` in git config | Asks for the reviewer's name before writing the round |
| Review: several rounds | Round number is one above the highest; earlier rounds are never edited, and the brief shows the latest earlier round |
| Review: on a `ramble/*` branch | Stops and points to `/donatello:ramble end` or switching to main |
| Review: no ongoing phase, several ongoing | Says none and stops; asks which one when several |
| No `docs/requirements.md` yet | Create it on this archive |
| Not a git repo | Use plain moves; mention that history won't be tracked. `phase-run` skips commits, and the reviewer gets a list of changed files instead of `git diff` |
| Implementer blocked or review fails twice | Re-dispatch the implementer on Sonnet; if it still fails, stop and present the issues in the main session |
| `docs/preferences.md` over 100 lines after merge | Onboard points out the longest sections and proposes moving file-specific ones into `.claude/rules/` |
| Project already has `.claude/settings.local.json` | Merge the plugin entries only; leave other keys alone |
| Project already has `.claude/settings.json` (committed import) | Merge the plugin entries only; leave other keys alone |
| Settings file is not valid JSON | Don't touch it; fail that import entry and print the entries to add by hand |
| `android` CLI missing or failing | Skip that entry; list it under Failed with `android skills add <id> --agent=claude-code --project=.` |
| Plugin marketplace unknown to this machine | `add` stops with "not found" if it's not in `known_marketplaces.json`; `import` fails the entry with `/plugin marketplace add <owner/repo>` then `/plugin install` |
| Plugin missing from the marketplace's `marketplace.json` | `add` stops with "not found"; nothing is written |
| Android skill id not listed by `android skills list` | `add` stops with "not found" |
| Git ref whose branch name contains `/` | Resolve by trying the longest prefix with `git ls-remote`; if none or several match, ask which is meant |
| Git ref that is a tag, or a URL with no `tree/` segment | Tags aren't resolved (only `refs/heads/`) and fall back to asking; no `tree/` means `HEAD` |
| Git repo private, missing, or offline | `GIT_TERMINAL_PROMPT=0` makes it fail instead of prompting; entry fails with the manual fetch command |
| Fetched folder has no `SKILL.md` | Fail the entry; nothing is copied; scratch directory removed |
| Copy fails partway | Remove the partial `.claude/skills/<id>/` and fail the entry |
| `add` with an id that already exists in the catalog | Refuse |
| `add` with a local folder that has no `SKILL.md` | Stop; nothing is copied into `catalog/skills/` |
| Skill folder already in `.claude/skills/<id>/` | Skip it at Install, list as already present, and don't add it to `.gitignore` (even if it appeared after Candidates) |
| No clone configured, `add` or `remove` | Ask once for the clone path and save it to `~/.claude/donatello.json`; decline stops the command |
| No clone found, `list` or `import` | Read the installed copy; never prompt. A clone found in the cwd or `~/.claude/donatello.json` is read instead |
| `import` in a project that isn't a git repo | The `git check-ignore` step is skipped (exit 128); `.gitignore` lines are still written for "just me" |
| Onboard with `--defaults` | Skill import skipped; the wrap-up points to `/donatello:skills import` |
| Ramble: not a git repo | Notes file only, with `mode: notes-only`; no branch, commits, or outcome. Wrap-up drafts Learnings and any catalog adds, then closes |
| Ramble: slug already used (notes file or `ramble/<slug>` exists) | Ask for another topic or slug; never reuse one |
| Ramble: dirty tree when starting in place | Offer stash (noted in the Log), worktree instead, or stop |
| Ramble: dirty tree when resuming in place | Offer stash or stop, then `git switch ramble/<slug>` on a clean tree |
| Ramble: two rambles open at once | Each has its own branch and notes; the picker lists all open and parked rambles. Worktree rambles run side by side; in-place rambles share one checkout, so switching needs a clean tree |
| Ramble: `end` outside a ramble | The picker lists open and parked rambles to choose from |
| Ramble: uncommitted changes at `end` | Checkpoint as `wip:` or discard after confirmation. In place, the switch back to main stops on any change other than the notes |
| Ramble: `ramble/*` branch with no notes file (orphan) | The picker creates its notes file (mode from `git worktree list`), then resumes it |
| Ramble: notes open, branch gone | The picker offers to mark it `abandoned`. A tracked file is committed on main; an untracked one is left for the user |
| Ramble: worktree path gone on resume | Say so; offer `git worktree add <path> ramble/<slug>` to recreate it |
| Ramble: worktree removal refused (e.g. untracked files) | Report it and ask; no `--force` |
| Ramble: squash merge conflicts | No `MERGE_HEAD`, so `git merge --abort` isn't used. Report `git status` and don't resolve without asking |
| Ramble: branch action fails | Report it; the notes stay committed with the chosen outcome; not retried |
| Ramble inside the Donatello repo | Catalog adds run from main, so a throwaway branch can't delete them |
| `phase-run` on a `ramble/*` branch | Stops and points to `/donatello:ramble end` or switching to main |
| `phase-discuss`: no ongoing phase | Suggests `/donatello:phase-new` and stops; archived phases are never revised |
| `phase-discuss`: several ongoing phases | Asks which one |
| `phase-discuss` on a `ramble/*` branch | Refuses and points to `/donatello:ramble end` or switching to main |
| `phase-discuss` with uncommitted source changes | Revision commit holds only the phase directory (pathspec); source changes stay uncommitted |
| `phase-discuss`: change that is really a new phase | Says so and offers a `Deferred:` line under Out of Scope instead of new tasks |
| `phase-discuss`: every task ticked | Nothing unticked to edit; changes arrive as new `(revises Task M)` tasks, and requirement changes need a revisit task or a recorded "no change needed" |
| `phase-discuss`: diff declined | Nothing written; the discussion is summarised in chat |
| `phase-discuss`: consistency check fails | Fixed before the diff is shown |
| Onboard: no git repo, no commits, or no `user.email` | Newcomer question defaults to yes; still asked |
| Onboard: monorepo or several stacks | Asks which part to onboard before going further |
| Onboard: "new" on an onboarded repo | Offers `/donatello:tour` or resume mode; no newcomer mode |
| Onboard: `--defaults` | No newcomer question; output as before |
| Tour: not a git repo | The tour reads no git history itself; git facts come only through the scout report, and anything it lacks is "unknown" |
| Tour: monorepo or several stacks | No separate rule; an area argument limits every stop to that part |
| Tour: area the docs don't cover | Scout dispatched once with the area path as focus |
| Tour: area matches nothing | Says so and asks for another area |
| Tour: no docs at all, not onboarded | Scout report is the only source; facts it lacks are "unknown" |
| Tour: no ongoing phase | Stop 5 says so; stop 6 suggests `/donatello:phase-new` |