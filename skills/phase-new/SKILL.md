---
name: phase-new
description: Start the next Donatello phase in .spec/ongoing/ from templates, with tasks shaped by the project's preferences doc.
argument-hint: "[scope]"
disable-model-invocation: true
---

# Start a new phase

Create `.spec/ongoing/phase<N>-<slug>/` with `overview.md`, `design.md`, `spec.md`, and `tasks.md`, filled from the templates in this skill's `templates/` directory. Scope from the argument: `$ARGUMENTS`

If `/donatello:onboard` sent you here for phase 1, skip step 1 (onboard already has the context) and go to step 2.

## 1. Gather context (cheaply)

- Read the preferences, architecture, and requirements docs at the paths the Donatello block in `CLAUDE.md` names (defaults: `docs/preferences.md`, `docs/architecture.md`, `docs/requirements.md`), each one only if it exists. The rest of this skill calls them by role ("the preferences doc").
- Read every `.claude/rules/*.md` too. Those rules load on their own only when code files are touched, and planning touches none, but tasks must follow them (e.g. UI setup order, preview requirements).
- List the directory names in `.spec/ongoing/` and `.spec/archive/`. Don't read old phases in full. For the most recent archived phase, read only its walkthrough's "Current state" and "Deviations from spec".
- Rambles: read the frontmatter of each `.spec/rambles/*.md`. For each one with `status: closed` and a `closed` date after the newest `.spec/archive/` directory date (all of them if there is no archive), read its Learnings. Use at most the 5 most recent. If given a ramble notes path (a promote), read that file in full.
- If `CLAUDE.md` (or `AGENTS.md`) has no Donatello block, the project isn't onboarded. Suggest `/donatello:onboard` and stop unless the user wants to continue anyway.
- If a phase is already in `.spec/ongoing/`, say so and ask whether to start another one in parallel or finish that one first.

## 2. Number and name

- **N** = the highest number in any `phase<N>-` directory name across `ongoing/` and `archive/`, plus 1; or 1 if there are none. Numbers never reset.
- **Slug**: kebab-case, at most 4 words, from the scope (e.g. `auth-and-onboarding`).
- If the directory already exists, stop and ask. Never overwrite.

## 3. Settle the scope

- If the scope argument already says what the phase delivers, restate it in 2–3 lines and continue. Don't interview for the sake of it.
- Otherwise ask briefly, one or two questions at a time: what the phase must deliver, what's explicitly out, anything already decided.

## 4. Write the files

Copy each template from `templates/` (relative to this skill's directory), fill it in, and **remove every HTML comment**; the comments are guidance for you, not content. Don't create `walkthrough.md`; it's created when the first task is finished.

- **overview.md**: goal, deliverables, out of scope, success criteria. Intent only. If rambles were read in step 1 (their Learnings, or the full notes on a promote), name each ramble in the goal or out of scope.
- **spec.md**: requirements state behavior, not implementation steps, and are numbered R1, R2, ... Every requirement has a **Check** a reviewer can run or look at.
- **design.md**: only decisions actually made so far, as Decision / Why / Tradeoff. Record what the read Learnings settled as decisions, naming the ramble. If none yet, leave a single line: `No decisions yet.`
- **tasks.md**: follow the rules below.

### Writing tasks

Tasks are carried out by a Haiku implementer agent working alone, then checked by a Sonnet reviewer, so each one must stand on its own:

- **Header:** `**Serves:**` requirement IDs, `**Files:**` exact file paths, each marked `(new)` or `(edit)`, `**Pattern:**` one existing file the implementer should copy the structure of, `**Skill:**` only when a specific skill you can see applies. Delete the Pattern or Skill line when there's nothing to name; never write "none".
- **Files and Pattern save the most.** The implementer starts from them instead of exploring the repo, which is where most of its tokens go. Look up real paths (list directories, `grep` for the class or module) rather than guessing; one quick lookup here saves a long search in every run.
- **Size:** one task is one implementer run: a few files, about 3–6 steps. Split anything bigger.
- **Concrete steps:** name the function, class, or composable and what it must return, emit, or show. "Add `farewell(name)` returning `"Bye, <name>"`", not "add farewell support".
- **Tests are explicit steps.** Any task that adds behavior gets a step saying what to test and what result to assert, written the way the preferences doc asks (e.g. assert results, not implementation). Never leave the implementer to infer that tests are wanted.
- **UI follows the preferences**, e.g. a preview step using the project's preview wrappers and grouped annotations.
- **Check:** the exact command to run, or the preview or screen to look at.
- **Commit:** a conventional commit line (`feat:`, `fix:`, `test:`, `chore:`, `docs:`).
- **Order** by dependency and by what the preferences require first (e.g. `:core:ui` preview wrappers and annotations before any feature UI). Draw the dependency sketch at the top.

## 5. Review

Show the user a short summary: the phase directory, the goal in one line, requirement titles, and the numbered task list. Ask for changes and apply them. Then offer to commit with `docs: plan phase <N> (<slug>)`; commit only if they agree.
