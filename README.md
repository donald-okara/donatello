# donatello

**A Claude Code plugin for spec-driven projects.** Donatello onboards Claude to a new or existing project, splits the work into small, reviewable phases, carries your coding preferences into every project, and routes work so Opus thinks, Haiku builds, and Sonnet reviews.

It's a set of skills, agents, and templates, not a program. Claude does the reasoning; Donatello supplies the workflow, the file conventions, and the guardrails.

```
/donatello:onboard  →  /donatello:phase-run  →  /donatello:phase-review  →  /donatello:phase-archive  →  /donatello:phase-new  →  …
   set up once          build task by task         record the review          close the phase             plan the next one
```

> **Status:** v0.4.0. The core workflow (onboard, phase-new, phase-run, phase-archive), `/donatello:skills`, and the three agents have been tested on fixture projects, and a first run on a real greenfield project went well. Trials on existing projects are still in progress, so expect rough edges there. The newer commands (`/donatello:ramble`, `/donatello:phase-discuss`, `/donatello:tour` with onboard's newcomer mode, and `/donatello:phase-review`) still have their live fixture checks pending (see `.spec/ongoing/`).

---

## Contents

- [Why Donatello](#why-donatello)
- [Requirements](#requirements)
- [Install](#install)
- [Quick start](#quick-start)
- [New to a repo?](#new-to-a-repo)
- [Commands](#commands)
- [Ramble](#ramble)
- [Revising a phase](#revising-a-phase)
- [How a phase works](#how-a-phase-works)
- [What Donatello adds to your project](#what-donatello-adds-to-your-project)
- [Existing projects and existing docs](#existing-projects-and-existing-docs)
- [Models and agents](#models-and-agents)
- [Preferences](#preferences)
- [Skill catalog](#skill-catalog)
- [Working without the commands](#working-without-the-commands)
- [Token cost](#token-cost)
- [Customizing](#customizing)
- [FAQ and troubleshooting](#faq-and-troubleshooting)
- [Repository layout](#repository-layout)

---

## Why Donatello

Without some structure, every new Claude session starts cold. You re-explain the project, re-state your conventions, and re-discover what was done last time. Larger features drift from what you asked for, and the expensive model ends up reading build logs.

Donatello fixes that with a few conventions:

- **A small, always-loaded entry point.** A marked block in `CLAUDE.md`, under 300 tokens, tells every session where the current work is and how to keep it up to date.
- **Phases instead of one long to-do list.** Each phase has its intent (`overview.md`), its requirements (`spec.md`), its decisions (`design.md`), its tasks (`tasks.md`), and a record of what actually happened (`walkthrough.md`).
- **Cheap resumes.** A new session reads only the walkthrough's "Current state", about ten lines, so `/clear` between tasks costs almost nothing.
- **Model routing.** Opus plans and coordinates, a Haiku agent implements each task, and a Sonnet agent reviews it against the spec. Build output and diffs stay inside the agents.
- **Living docs.** When a phase closes, lasting decisions and requirements are promoted into the project docs, so current behavior is always written down in one place.
- **Your preferences everywhere.** Your coding conventions are copied into each project, and file-specific ones load only when Claude touches matching files.

---

## Requirements

- [Claude Code](https://claude.com/claude-code) with plugin support
- A git repository is strongly recommended. Without one, Donatello still works, but `phase-run` can't commit per task and the reviewer gets a file list instead of a diff.
- Access to Opus, Sonnet, and Haiku. The main session should run on Opus, and the agents pin their own models.

---

## Install

Donatello is its own plugin marketplace. Inside Claude Code:

```
/plugin marketplace add donald-okara/donatello
/plugin install donatello@donatello
```

Then restart Claude Code or start a new session, and type `/donatello:` to see the commands.

**Update** to the latest version. It takes two steps:

```
/plugin marketplace update donatello
/plugin update donatello@donatello
```

1. `marketplace update` fetches the new code into the marketplace copy.
2. `plugin update` installs it into your plugin cache. If it says Donatello is already current, run `/plugin uninstall donatello@donatello` and then `/plugin install donatello@donatello`.

Then start a new session.

**Uninstall:** `/plugin uninstall donatello@donatello`. The files Donatello wrote into your projects (`docs/`, `.spec/`, the CLAUDE.md block) are ordinary files and stay where they are.

> **Using it yourself?** The bundled [preferences](#preferences) are the author's (Android and Compose focused). To use your own, fork the repo, edit `preferences/`, and install from your fork: `/plugin marketplace add <you>/donatello`.

---

## Quick start

In the project you want to work on:

1. **Onboard:** `/donatello:onboard`
   Donatello surveys the repo, asks about five questions, offers skills from the [catalog](#skill-catalog) that fit your stack, shows a proposal (docs, preferences, phase plan, files it will touch), and writes everything once you approve.
2. **Run the first task:** `/donatello:phase-run`
   Haiku implements the next task, Sonnet reviews it, and the task gets ticked, recorded in the walkthrough, and committed.
3. **Keep going:** `/donatello:phase-run` again, or `/donatello:phase-run --all` to run every remaining task, stopping only when something needs you.
4. **Review the phase:** `/donatello:phase-review`
   Briefs you on what the phase delivered, takes your verdict (approved or changes requested) and comments, and records them in the phase's `review.md`.
5. **Close the phase:** `/donatello:phase-archive`
   Finalizes the walkthrough, promotes decisions and requirements into the project docs (after showing you the diff), and moves the phase to the archive. It asks first if the phase isn't approved.
6. **Plan the next one:** `/donatello:phase-new "offline sync"`

You can `/clear` between any of these steps; the next session picks up from the walkthrough.

To add skills later, `/donatello:skills import` offers the catalog entries for this project's stack. `/donatello:skills list` shows every entry.

---

## New to a repo?

Run `/donatello:tour` in any repo, onboarded or not. It walks you through six stops in chat: what the project is, its modules, how to run it, how code is written here, where work stands, and where to start. Each stop ends with "questions, deeper, or next?"

- It's read-only. It writes no files, notes, or commits, and says so at the end.
- `/donatello:tour auth` limits every stop to one area.
- In a repo without Donatello, it works from a survey by the `scout` agent, skips the "where work stands" stop, and points you to `/donatello:onboard`.
- If you're joining a project that Donatello already set up, `/donatello:onboard` offers the tour when you say you're new.

---

## Commands

All commands run only when you type them, except `/donatello:tour`, which Claude may also start when you ask to be shown around. It writes nothing.

### `/donatello:onboard [--defaults]`

Sets up Donatello in the current project, new or existing.

1. **Checks what's already there:** `CLAUDE.md`, `AGENTS.md`, existing docs, `.claude/rules/`, `.spec/`. If the project is already onboarded, it switches to **resume mode** and only offers what's missing. It also asks whether you're new to the repo (a guess from your git commits, which you confirm). If you are, the interview asks you to confirm each finding or mark it "don't know"; unanswered questions go under `Open questions` in the architecture doc, and conflicting preferences keep the project's rule.
2. **Surveys the repo** with the `scout` agent (stack, modules, exact build and test commands, conventions, existing docs). The main session doesn't read your code.
3. **Picks preferences** that match your stack and removes any that your project already states.
4. **Interviews you** briefly: what's being built, open questions from the survey, which preferences to keep, and how to resolve conflicts.
5. **Offers skills** from the [skill catalog](#skill-catalog) that fit your stack: you pick all, some, or none, then choose committed or just me.
6. **Proposes** the docs, the preferences split, a phase breakdown (phase 1 in detail, later ones as one-liners), the skills to install, plugins to disable, and every file it will create or touch. Nothing is written until you say yes.
7. **Writes** the docs, `.claude/rules/`, phase 1, the CLAUDE.md block, and the chosen skills. It can also turn off plugins that don't fit the project (only for you, via `.claude/settings.local.json`).
8. Offers to commit with `docs: onboard project to Donatello`. Committed skill files are included; "just me" skills stay out of the commit.

`--defaults` skips the interview, the approval step, skill import, and plugin pruning, and writes new files straight from the survey. It still never changes existing content. The summary points to `/donatello:skills import` for later.

### `/donatello:phase-new [scope]`

Plans the next phase in `.spec/ongoing/phase<N>-<slug>/`.

```
/donatello:phase-new "user auth with email magic links"
```

- Reads the project docs, `.claude/rules/`, and only the closing summary of the last archived phase.
- If the scope is clear, it restates it and moves on; otherwise it asks one or two questions.
- Writes `overview.md`, `spec.md` (numbered requirements R1, R2, … each with a **Check**), `design.md` (decisions only), and `tasks.md`.
- Tasks are sized for one Haiku run (a few files, 3–6 concrete steps) and include explicit test steps, a Check, and a commit message.
- Numbering continues across ongoing and archived phases and never resets.

### `/donatello:phase-discuss [topic]`

Revises the ongoing phase's plan through a conversation. See [Revising a phase](#revising-a-phase).

```
/donatello:phase-discuss "split task 3 into a store and a screen"
```

- Talks the change through in chat, then shows one diff across the phase files and applies it only after you approve.
- Ticked tasks are never edited. Rework becomes a new task, `Task <N>: <title> (revises Task <M>)`.
- Task and requirement numbers are never reused or renumbered. Removed tasks stay as `(removed: …)` stubs, deferred ones as `(deferred)`, and dropped requirements as `(dropped <date>)`.
- Commits only the phase folder, as `docs: revise phase <N> plan (…)`. Your other uncommitted changes are left alone.

### `/donatello:phase-run [task | --all]`

Runs tasks from the active phase.

```
/donatello:phase-run          # the next task whose dependencies are done
/donatello:phase-run 4        # a specific task
/donatello:phase-run --all    # every remaining task, stopping when you're needed
```

For each task:

```
implementer (Haiku) ──► reviewer (Sonnet) ──PASS──► tick task · update walkthrough · commit
                              │
                           ISSUES
                              ▼
                  fix round (Haiku, with the issues)
                              │ still failing
                              ▼
                  escalate (implementer on Sonnet)
                              │ still failing
                              ▼
                  stop and show you the issues (changes left uncommitted)
```

- Requires a clean working tree, so each commit contains only that task.
- The main session never reads source, diffs, or build logs. It works from short agent reports.
- Stops and asks you when a task is blocked on a decision only you can make, or when the environment (not the code) is broken.

### `/donatello:phase-review [phase]`

Records a maintainer's review of a finished phase in `<phase>/review.md`. See [How a phase works](#how-a-phase-works).

```
/donatello:phase-review
/donatello:phase-review 6
```

- Shows a brief in chat: the goal, how to verify, deviations, the commits by task, and the latest earlier round.
- Asks for a verdict, **approved** or **changes requested**, then for comments, including none. Comments are recorded in your words; Claude adds none.
- Appends one `## Review <n>` round after your approval. Earlier rounds are never edited.
- Nothing is acted on: no tasks, plans, or code change. Only `review.md` changes, and it's committed only if you say so.

### `/donatello:phase-archive [phase]`

Closes a phase.

1. If any tasks are unticked, asks whether to archive anyway (recorded as dropped), carry them over to the next phase, or stop.
2. Checks the latest review. A missing review, or changes requested, asks whether to archive anyway or stop. An approved review continues without asking.
3. Rewrites the walkthrough's "Current state" as a closing summary and makes sure "How to verify" covers the whole phase. If there's no walkthrough, it builds one from `tasks.md` and git history.
4. Promotes **lasting decisions** to the architecture doc and **lasting behavior** to the requirements doc, replacing entries they supersede. You see one diff and approve it first.
5. Moves the folder to `.spec/archive/<YYYY-MM-DD>-phase<N>-<slug>/` with `git mv`. The archived folder keeps `review.md`.

### `/donatello:skills [list | add <ref> | remove <id> | import]`

Manages the skill catalog and installs catalog skills into the current project. With no argument, it shows the list and asks what to do.

```
/donatello:skills list
/donatello:skills add android:navigation-3
/donatello:skills remove styles
/donatello:skills import
```

- **`list`** prints each entry as `id · applies-to · summary`, grouped by source.
- **`add <ref>`** takes one of four ref forms (see [Sources](#sources)). It fills in the id and summary, refuses an id that's already in the catalog, shows the entry, and writes only after you approve. It offers to commit `feat: add <id> to skill catalog` and push.
- **`remove <id>`** confirms first, then deletes the entry (and the folder, for your own skills). It offers to commit `chore: remove <id> from skill catalog`.
- **`import`** runs the same flow as onboard's skill step. See [Importing into a project](#importing-into-a-project).

`add` and `remove` edit your Donatello clone. If the current directory is the Donatello repo, or `~/.claude/donatello.json` names a clone, that clone is used. Otherwise `add` and `remove` ask once for the clone's path and save it in `~/.claude/donatello.json`. `list` and `import` never ask for a clone: they read the clone when one is found, and the installed copy otherwise.

### `/donatello:ramble [topic | end]`

Explores a question on a branch or in a worktree, with notes in `.spec/rambles/`, then decides what survives. See [Ramble](#ramble).

```
/donatello:ramble "does the sync layer need CRDTs"
/donatello:ramble            # picker: continue, resume, or start new
/donatello:ramble end        # wrap up and choose an outcome
```

### `/donatello:tour [area]`

A read-only orientation in chat for someone new to a repo. See [New to a repo?](#new-to-a-repo).

```
/donatello:tour
/donatello:tour auth         # limit every stop to one area
```

- Reads the project's docs, the ongoing phase's overview and "Current state", and ramble frontmatter. In a repo without Donatello, it runs the `scout` agent instead.
- Reads source files only when you ask for "deeper" on a point.
- Writes nothing, and says so at the end.

---

## Ramble

A ramble is a short exploration you can throw away. You pick a question, work on it in a branch or worktree, keep notes as you go, and decide at the end what survives. Rambles aren't phases: no number, spec, or review loop.

**Start.** `/donatello:ramble <topic>` (or `/donatello:ramble` and choose a new ramble). Donatello asks whether to use a **worktree** (`../<repo>-ramble-<slug>`, so your main checkout stays free) or **in place** (a branch in the current checkout, which needs a clean tree). Either way it creates the branch `ramble/<slug>` and a notes file in `.spec/rambles/`. In a folder that isn't a git repo, you get the notes file only.

**Ramble mode.** Claude teaches as you go: it explains why things work or break and compares designs with their trade-offs. Preferences are relaxed unless the experiment is about them. Main is never touched except the notes file. Work happens on the ramble branch, checkpointed as `wip:` commits, and each finding adds one line to the notes' Log.

**Resume.** `/donatello:ramble` with no argument shows a picker: continue the current ramble, resume an open or parked one, or start a new one. Rambles whose branch is gone can be marked abandoned, and stray `ramble/*` branches can be adopted.

**End.** `/donatello:ramble end` wraps up the current ramble. It checks for uncommitted changes, drafts Learnings (at most 10 lines) for you to edit, and then asks for one of five outcomes:

| Outcome | What happens | Status |
|---|---|---|
| **throwaway** | Branch and worktree are deleted | closed |
| **kept** | Branch and worktree stay for later | parked |
| **merged** | Squash-merged into main as one commit, with a message you approve; branch and worktree are deleted | closed |
| **PR** | Branch is pushed and a pull request is opened with `gh`; branch and worktree stay | closed |
| **promoted to phase** | `phase-new` starts a phase with the notes as context | closed |

The notes are committed to main whatever the outcome, so what you learned survives even when the code doesn't.

**Skills to the catalog.** Wrap-up can add skill folders made during the ramble to the [skill catalog](#skill-catalog), and can draft a new skill from the Learnings. Adds run from main, so a throwaway branch can't take them along.

**Phases read rambles.** `phase-new` reads the Learnings of recently closed rambles and names them in the decisions they settled. `phase-run` refuses to run on a `ramble/*` branch.

---

## Revising a phase

Plans change. When `phase-run` stops and the task or spec looks wrong, it suggests `/donatello:phase-discuss`. You can also run it any time to change an ongoing phase.

- **Ticked tasks stay as they are.** Their commits match their text, so a change to finished work becomes a new `(revises Task M)` task.
- **Numbers are stable.** A new task takes the next free number. Removed or deferred tasks stay in `tasks.md` as stubs, so a number always points to the same thing.
- **Requirement changes are recorded.** Edits to `spec.md` get a dated entry in `design.md`, and a deferred task gets a `Deferred:` line in `overview.md`. Tasks that cite a changed requirement are revised, or a revisit task is proposed for a ticked one.
- **Nothing is written before you approve.** You see the whole change as one diff, and declining writes nothing.

---

## How a phase works

Each phase is one folder with five files, each with a single job, plus `review.md` once the phase is reviewed:

| File | Answers | Written | Changes after planning? |
|---|---|---|---|
| `overview.md` | What and why: goal, deliverables, out of scope, success criteria | `phase-new` | No |
| `spec.md` | What must be true: numbered requirements, each with a Check | `phase-new` | Rarely |
| `design.md` | Why this way: Decision / Why / Tradeoff | `phase-new`, then as decisions are made | Yes |
| `tasks.md` | What to do: checklist of self-contained tasks | `phase-new`, ticked by `phase-run` | Ticked |
| `walkthrough.md` | What actually happened: Current state, How to verify, Changes by task, Deviations from spec | After the first finished task | Every task |

Once every task is ticked, `/donatello:phase-review` records the maintainer's verdict and comments in `review.md`, one `## Review <n>` round per review, never rewritten. It's a record only: a comment doesn't create a task, and nothing else changes. `/donatello:phase-archive` checks the latest verdict before it moves the phase, but it asks rather than blocks.

A task in `tasks.md` looks like this:

```markdown
### Task 3: Farewell greeting
**Serves:** R2
**Files:** app/src/main/kotlin/greeting/

- [ ] Add `farewell(name)` returning `"Bye, <name>"`
- [ ] Test: `farewell("Ada")` returns `"Bye, Ada"`
- [ ] Check: `./gradlew :app:test`
- [ ] Commit: `feat: add farewell greeting`
```

**`docs/` vs `.spec/`:** `docs/` answers "what is this, how does it behave, and why" over the project's lifetime. `.spec/` answers "what are we doing right now". Phase files point to `docs/` instead of repeating it.

---

## What Donatello adds to your project

```
project/
├── CLAUDE.md                     # your content untouched; Donatello block appended
├── .claude/
│   ├── rules/                    # path-scoped preferences, e.g. android-ui.md for **/*.kt
│   ├── skills/                   # catalog skills imported with onboard or /donatello:skills import
│   ├── settings.json             # committed plugin entries from the import (only if you chose committed)
│   └── settings.local.json       # personal: plugins disabled for this project, and "just me" imports (gitignored)
├── docs/
│   ├── architecture.md           # what the project is, modules, patterns, build/test commands, decisions
│   ├── requirements.md           # how the system behaves now; grows with each archived phase
│   └── preferences.md            # general preferences, always loaded (≤ 100 lines)
└── .spec/
    ├── ongoing/
    │   └── phase1-<slug>/        # overview, spec, design, tasks, walkthrough
    ├── rambles/
    │   └── <date>-<slug>.md      # ramble notes: Question, Log, Learnings, Outcome
    └── archive/
        └── 2026-10-16-phase1-<slug>/
```

Any of the `docs/` files may be **an existing file of yours** instead (see the next section). `requirements.md` is created only when there's real behavior to describe, at the latest on the first archive.

### The CLAUDE.md block

This is the entry point every session loads. It's wrapped in markers so Donatello can update it without touching anything else in the file:

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

The paths in it are filled in for your project. The skills and agents look up the docs through these lines.

---

## Existing projects and existing docs

Donatello has two rules for projects that already have content: **never overwrite** and **never duplicate**.

**Never overwrite**
- Existing files change only after you see a diff and say yes.
- The one exception is appending the Donatello block to an existing `CLAUDE.md`, which adds without changing a byte of your content.
- An existing `.spec/` or Donatello block puts onboard in resume mode, so it never starts over.

**Never duplicate**
- **Docs you already have are adopted.** Donatello needs three docs: architecture, requirements, and preferences. If an existing file already covers one (`docs/DESIGN.md`, `CONTRIBUTING.md`, an `adr/` folder, …), Donatello uses it instead of creating its own. The CLAUDE.md block points to it, `phase-archive` writes to it in its existing format, and anything missing is proposed as a diff. Names are matched case-insensitively, so `docs/ARCHITECTURE.md` counts.
- **CLAUDE.md stays lean.** If your architecture or requirements are described only inside `CLAUDE.md` or `AGENTS.md`, Donatello still creates a separate file, because those files load every session and these docs grow over time. The new file leaves out anything `CLAUDE.md` already says and points to it instead.
- **Preferences are deduplicated rule by rule.** Any preference your project already states (in `CLAUDE.md`, `AGENTS.md`, `.claude/rules/`, or an existing conventions doc) is dropped. If your project says the opposite, your rule wins by default and the conflict comes up in the interview. If nothing new is left, no `preferences.md` is created.
- **AGENTS.md-only projects:** onboard asks whether the block goes into `AGENTS.md` or a new `CLAUDE.md`.
- **Monorepos:** onboard asks which app or stack to onboard before going further.

---

## Models and agents

| Who | Model | Does | Can edit? | Turn cap |
|---|---|---|---|---|
| Main session | Opus (your session) | Interviews, planning, coordinating, architecture questions | Plans and docs | – |
| `donatello:scout` | Haiku | Surveys the repo for onboard; returns ≤ 60 lines | No (read-only) | 25 |
| `donatello:implementer` | Haiku (Sonnet when escalated) | Implements exactly one task, runs its Check and tests | Code only; never `.spec/` or commits | 40 |
| `donatello:reviewer` | Sonnet | Reviews one task's changes against the task, spec, and preferences; `PASS` or ≤ 10 issues | No (read-only) | 20 |

- Models are pinned in each agent's frontmatter, so they hold for the whole agent run.
- Agents run builds and tests in quiet mode and report only failures, never full logs.
- The reviewer judges against the spec and your preferences, not general style taste.
- Haiku implementation is safe because tasks are precise and **the review and the escalation ladder are always on**.

---

## Preferences

Preferences live in `preferences/`, one Markdown file per category. Each file has frontmatter:

```yaml
---
category: testing
summary: Test results, not implementation; JUnit + Mockito, Robolectric for UI; previews count as tests
applies-to: [android]        # optional: only offered for matching stacks
paths: ["**/*.kt"]           # optional: makes it a path-scoped rule
---
```

How onboard uses them:

- **No `paths`:** goes into the project's preferences doc, imported by the CLAUDE.md block and always loaded. Kept at 100 lines or less.
- **With `paths`:** becomes `.claude/rules/<category>.md` and loads only when Claude touches matching files. This keeps always-loaded context small.
- They're **copied, not linked**, so each project can diverge freely. Changing a preference in Donatello doesn't touch existing projects. Re-run `/donatello:onboard` to pull updates as a diff.

Bundled preferences:

| File | Applies to | Summary |
|---|---|---|
| `model-usage.md` | all | Opus plans and coordinates; Haiku implements; Sonnet reviews and takes over when Haiku can't finish |
| `testing.md` | Android | Test results, not implementation; JUnit + Mockito, Robolectric for UI; previews count as tests |
| `naming.md` | Android | Case by case; screens use Route + Content; components are `[Title][Type]` |
| `android-ui.md` | Android, `**/*.kt` | Compose only; mandatory previews wrapped and grouped via `:core:ui`; screens own their Scaffold; hoisted state |

To add your own, drop a new file into `preferences/` in your fork with a `summary` (and `applies-to` / `paths` if needed).

---

## Skill catalog

Skills are a separate kind of content from preferences. Donatello keeps a **catalog** of skills that onboard and `/donatello:skills import` can offer. Each entry records where a skill comes from. Only your own skills are stored as files; Android CLI skills, plugins, and GitHub skill folders keep their own install and update channels.

### Catalog format

The catalog is `catalog/skills.yaml` in the Donatello repo:

```yaml
- id: adaptive
  source: android
  ref: adaptive
  summary: Make or update Android UI adapt to phones, tablets, foldables, desktop, TV, Auto and XR.
  applies-to: [android]
```

| Field | Meaning |
|---|---|
| `id` | Unique, kebab-case. Also the folder name in the project |
| `source` | `android`, `plugin`, `git`, or `donatello` |
| `ref` | Where to install from: the Android CLI id, `<plugin>@<marketplace>`, a GitHub folder URL pinned to a commit, or `catalog/skills/<id>` |
| `summary` | One line, shown during onboard |
| `applies-to` | Optional stack list. Left out means every stack |
| `marketplace` | Plugins only: the marketplace's `owner/repo` |

### Sources

| Source | Ref form | Installs | `add` example |
|---|---|---|---|
| `android` | `android:<id>` | One skill, through the `android` CLI | `/donatello:skills add android:navigation-3` |
| `plugin` | `<plugin>@<marketplace>` | The whole plugin (Claude Code enables plugins, not single skills) | `/donatello:skills add superpowers@claude-plugins-official` |
| `git` | GitHub URL of a folder containing `SKILL.md` | That folder, pinned to the commit `add` resolved | `/donatello:skills add https://github.com/owner/repo/tree/main/skills/my-skill` |
| `donatello` | Local folder containing `SKILL.md` | A copy of the folder, kept in `catalog/skills/<id>/` | `/donatello:skills add ./my-skill` |

- **`android`** and **`plugin`** need the skill or marketplace to be available on this machine. `add` reads the name and description from `android skills list` or the marketplace's `marketplace.json`.
- **`git`** refs are resolved to a commit SHA and stored as `.../tree/<sha>/<path>`, so an upstream change can't silently change a project. To move the pin to a newer commit, `remove` the entry and `add` the URL again (`add` refuses an id that already exists).
- **`donatello`** folders are copied into `catalog/skills/<id>/` in your clone, and the copy is offered for commit.

### Importing into a project

Run `/donatello:skills import` in the project root, or accept the skill step during onboard. Standalone, it first asks which stacks the project uses (for example `android`, `web`, or none). Then it:

1. **Lists the candidates:** entries whose `applies-to` includes the project's stack, plus entries with no `applies-to`. Skills already present are shown separately and not offered again.
2. **Asks** all / pick / none, then **committed** or **just me**. Once per import, not per skill.
3. **Installs** the chosen entries and prints a summary.

What each scope writes:

| Source | Committed (shared through git) | Just me (this machine only) |
|---|---|---|
| `android` | Runs `android skills add <id> --agent=claude-code --project=.` | The same command, then the skill folder goes in `.gitignore` |
| `plugin` | Adds the marketplace (if missing) to `extraKnownMarketplaces` and `"<plugin>@<marketplace>": true` to `enabledPlugins` in `.claude/settings.json` | Writes `enabledPlugins` to `.claude/settings.local.json`, and adds the marketplace there only if `~/.claude/plugins/known_marketplaces.json` doesn't already list it. The file then goes in `.gitignore` |
| `git` | Fetches the pinned folder into `.claude/skills/<id>/` | The same folder, listed in `.gitignore` |
| `donatello` | Copies `catalog/skills/<id>/` into `.claude/skills/<id>/` | The same folder, listed in `.gitignore` |

Import never replaces an existing `.claude/skills/<id>/` folder, and it adds only the settings keys that are missing. If a plugin was enabled, start a new session to load it.

### When an import fails

A failed entry is skipped and the rest carry on. The summary lists each failure with a command to fix it by hand:

| Failure | What to do |
|---|---|
| `android` CLI missing or failing | Run `android skills add <id> --agent=claude-code --project=.` |
| Plugin marketplace unknown here | Run `/plugin marketplace add <owner/repo>`, then `/plugin install <plugin>@<marketplace>` |
| Settings file isn't valid JSON | Fix the file. The summary names the entries to add by hand. Donatello leaves the file untouched |
| Git repo private, missing, or offline | Fix access, then re-run `/donatello:skills import`. The manual alternative is to fetch the folder at its SHA and copy it |
| `SKILL.md` missing from a catalog folder | Fix the catalog folder, then re-run the import. Or copy it: `cp -R catalog/skills/<id> .claude/skills/<id>` |

---

## Working without the commands

You don't have to use `phase-run`. Because the CLAUDE.md block tells every session how the workflow goes, normal conversation keeps the structure intact:

- Ask Claude to do the next task in plain words. After finishing it, Claude ticks it in `tasks.md` and updates `walkthrough.md` (creating it on the first task).
- At the end of a session, Claude rewrites the walkthrough's "Current state", so the next session knows where things stand.

`phase-run` adds the Haiku/Sonnet routing, the review, and per-task commits on top.

---

## Token cost

Measured after installing from GitHub:

| | Approx. tokens |
|---|---|
| Always on, every session | ~391 |
| `/donatello:onboard` when invoked | ~2.1k |
| `/donatello:phase-new` | ~1.5k |
| `/donatello:phase-run` | ~1.3k |
| `/donatello:phase-review` | ~1.0k |
| `/donatello:phase-archive` | ~1.2k |
| Each agent definition | ~0.5–0.8k |

What keeps it low:

- Workflow skills except `/donatello:tour` are explicit-only, so they don't sit in the model's skill list (the tour's short description is the one exception).
- The CLAUDE.md block rarely changes, which keeps the prompt cache warm. The walkthrough is deliberately not imported.
- Resumes read about ten lines ("Current state"), not the whole phase.
- Surveys, builds, tests, and diffs happen inside agents, and only short reports come back.
- Onboard offers to disable plugins that are irrelevant to the project, since every enabled plugin's skill descriptions are sent on every turn.

---

## Customizing

- **Edit project files freely.** Everything Donatello writes into a project is plain Markdown owned by that project. Edit `docs/preferences.md`, rules, phase files, or anything else outside the block markers.
- **Override an agent for one project:** add `.claude/agents/implementer.md` (or `reviewer.md`, `scout.md`) to the project. Project agents take priority over plugin agents, so you can change the model, tools, or instructions.
- **Change the defaults for all projects:** fork the repo and edit `preferences/`, `agents/`, the phase templates in `skills/phase-new/templates/`, or the block in `templates/claude-block.md`.
- **Add your own skill:** put a folder with a `SKILL.md` somewhere on disk and run `/donatello:skills add ./path/to/folder`. It's copied into `catalog/skills/<id>/` in your clone, and the catalog entry is offered for commit. Other projects can then import it.

---

## FAQ and troubleshooting

**Will onboarding mess up my existing CLAUDE.md?**
No. It appends the block at the end and changes nothing else. If a block is already there, it shows a diff of the area between the markers and replaces it only if you approve.

**I ran onboard twice.**
That's fine. The second run detects the block or `.spec/` and switches to resume mode: it reports what exists and offers only missing pieces, a preferences refresh, or a block update.

**`phase-run` says my working tree isn't clean.**
Commit or stash your changes first. Each task is committed on its own, and uncommitted changes would end up in the review and the commit.

**A task keeps failing review.**
After a fix round and a Sonnet retry, `phase-run` stops and shows you the issues with its take on whether the task or the spec needs changing. The changes are left uncommitted so you can inspect them. Usually the fix is to make the task more precise in `tasks.md`, or split it.

**Can I run two phases at once?**
Yes. `phase-new` asks if a phase is already ongoing, and `phase-run`, `phase-discuss`, and `phase-archive` ask which one when there are several.

**The commands don't show up.**
Make sure the plugin is installed and enabled (`/plugin`), then start a new session. If a newer command is missing after an update, you probably ran only the first of the two [update](#install) steps: `/plugin marketplace update donatello` fetches the code, but the old cached copy stays in use until `/plugin update donatello@donatello` (or an uninstall and reinstall).

**My project isn't Android.**
Onboard works on any stack. The scout reads `package.json`, `pyproject.toml`, Gradle files, and more. Only preferences whose `applies-to` matches your stack are offered, so a web project gets just the general ones until you add your own.

---

## Repository layout

```
donatello/
├── .claude-plugin/
│   ├── plugin.json               # plugin manifest
│   └── marketplace.json          # makes this repo its own marketplace
├── skills/
│   ├── onboard/                  # /donatello:onboard
│   ├── phase-new/                # /donatello:phase-new, plus the phase templates (incl. review.md)
│   ├── phase-discuss/            # /donatello:phase-discuss
│   ├── phase-run/                # /donatello:phase-run
│   ├── phase-review/             # /donatello:phase-review
│   ├── phase-archive/            # /donatello:phase-archive
│   ├── ramble/                   # /donatello:ramble, plus the notes template
│   ├── tour/                     # /donatello:tour (read-only orientation)
│   └── skills/                   # /donatello:skills (list, add, remove, import)
├── agents/                       # scout, implementer, reviewer
├── catalog/
│   ├── skills.yaml               # skill catalog: what onboard and import can offer
│   └── skills/<id>/              # your own skills, copied into projects on import
├── preferences/                  # coding preferences copied into projects
├── templates/claude-block.md     # the CLAUDE.md block
├── docs/
│   ├── SPECIFICATION.md          # what Donatello does and why
│   └── ARCHITECTURE.md           # how it's built, decisions, edge cases
└── .spec/                        # Donatello's own phases (it's built with itself)
```

For the full design, read [`docs/SPECIFICATION.md`](docs/SPECIFICATION.md) and [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

## License

MIT, as declared in `.claude-plugin/plugin.json`.
