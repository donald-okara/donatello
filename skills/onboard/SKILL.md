---
name: onboard
description: Onboard this project to Donatello: survey the repo, interview, propose phases, then write docs/, .spec/, and a CLAUDE.md block.
argument-hint: "[--defaults]"
disable-model-invocation: true
---

# Onboard a project

Set up the Donatello structure in the current project: `docs/` (preferences, architecture, requirements), `.claude/rules/`, phase 1 in `.spec/ongoing/`, and the Donatello block in `CLAUDE.md`. Arguments: `$ARGUMENTS`

**`--defaults`**: skip the interview and the approval step and write new files straight away from what the survey found. Even then: never change existing content, never replace an existing Donatello block, and skip plugin pruning (mention the command for later instead).

**Never overwrite.** Existing files are left alone or changed only after showing a diff and getting a yes. The single exception is appending the Donatello block to an existing `CLAUDE.md`, which adds without changing anything.

Paths below that start with `../` are relative to this skill's directory.

## 1. Check what's already here

Look for `CLAUDE.md` (and whether it has `<!-- donatello:start -->`), `AGENTS.md`, `docs/preferences.md`, `docs/architecture.md`, `.claude/rules/`, and `.spec/`.

- **Already onboarded** (block or `.spec/` present): switch to resume mode. Report what exists and what's missing, then offer only the missing pieces, a preferences refresh (shown as a diff against the current Donatello preferences), or a block update. Don't redo anything that exists.
- **Only `AGENTS.md`, no `CLAUDE.md`**: ask whether the block goes into `AGENTS.md` or a new `CLAUDE.md`.

## 2. Survey with the scout

Unless the repo is empty, dispatch the `donatello:scout` agent and work from its report. **Don't read the codebase yourself**; that's what the scout is for. Read individual files only to settle a specific question its report leaves open.

If the report shows several apps or stacks (a monorepo), ask which part to onboard before going further.

## 3. Pick preferences

Read the frontmatter of every file in `../../preferences/`. Select the files whose `applies-to` includes this project's stack, plus files with no `applies-to`. Sort them:
- **general** (no `paths`) → `docs/preferences.md`
- **path-scoped** (has `paths`) → `.claude/rules/<category>.md`

Read the bodies of the selected files too; you need them to plan phase 1.

## 4. Interview (skip with `--defaults`)

Keep it short: about five questions in total, one or two at a time, and never ask what the scout already answered. Cover:
- what's being built and where it's going (new projects: the core idea; existing: what's next)
- anything the scout's "Open questions" flagged, especially architecture specifics the tasks will depend on (DI, state management, navigation)
- the preference selection: list each file's `summary` and ask if any should be dropped or changed for this project

## 5. Propose (skip approval with `--defaults`)

Show one compact proposal and wait for a yes or changes:
- the architecture outline you'll write (a few bullets)
- preferences: which files go to `docs/preferences.md` and which to `.claude/rules/`
- phase breakdown: phase 1 with its goal and rough tasks; later phases as one-liners
- plugins to disable for this project (see step 6g)
- every file you'll create, and every existing file you'll touch and how

## 6. Write

**a. `docs/preferences.md`.** Start with `<!-- Copied from Donatello preferences on <YYYY-MM-DD>; edit freely for this project. -->`, then the body of each general file (frontmatter removed) as its own section. Keep the total at 100 lines or less; if it's over, point out the longest sections and suggest moving file-specific ones into `.claude/rules/`. If the file already exists, show a diff and merge only on approval.

**b. `.claude/rules/<category>.md`.** For each path-scoped preference: the body, with frontmatter reduced to the `paths` key only (that's the key Claude Code reads for rules).

**c. `docs/architecture.md`.** Under 80 lines, from the scout report and the interview: what the project is, modules, stack, key patterns (state, DI, navigation, testing), **exact build and test commands** (the agents rely on these), and decisions already made. If it exists, propose additions as a diff instead.

**d. `docs/requirements.md`.** Existing projects only, and only when current behavior is clear from the scout report: plain requirement statements grouped by area. Otherwise don't create it at all, not even as a placeholder; the first `phase-archive` creates it.

**e. Phase 1.** Follow `../phase-new/SKILL.md` from its step 2, using its templates. You already have the context, so don't repeat its step 1. In resume mode with archived phases, numbering continues from them.

**f. CLAUDE.md block.** Read `../../templates/claude-block.md`.
- No `CLAUDE.md`: create it with `# <Project name>` and the block.
- `CLAUDE.md` without the markers: append a blank line and the block at the end. Change nothing else.
- `CLAUDE.md` with the markers: show a diff of the region between them and replace it only on approval. Never touch content outside the markers.

**g. Plugin pruning** (skip with `--defaults`). Run `claude plugin list --json` and take the enabled plugins. Propose disabling the ones clearly irrelevant to this project's stack (e.g. web hosting, SEO, or marketing plugins in an Android app). Never propose `donatello` itself. For each one the user confirms, run `claude plugin disable <id> --scope local`; this writes `.claude/settings.local.json`, so it affects only this project and only this user. Make sure `.claude/settings.local.json` is listed in `.gitignore`.

## 7. Wrap up

Summarize what was created and changed in a few lines, name the next step (`/donatello:phase-run` to start phase 1's first task), and offer to commit with `docs: onboard project to Donatello`. Leave `.claude/settings.local.json` out of the commit. Commit only if the user agrees.
