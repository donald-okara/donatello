---
name: onboard
description: Onboard this project to Donatello: survey the repo, interview, propose phases, then write docs/, .spec/, and a CLAUDE.md block.
argument-hint: "[--defaults]"
disable-model-invocation: true
---

# Onboard a project

Set up the Donatello structure in the current project: `docs/` (preferences, architecture, requirements), `.claude/rules/`, phase 1 in `.spec/ongoing/`, and the Donatello block in `CLAUDE.md`. Arguments: `$ARGUMENTS`

**`--defaults`**: skip the interview and the approval step and write new files straight away from what the survey found. Even then: never change existing content, never replace an existing Donatello block, and skip plugin pruning and skill import (mention `/donatello:skills import` for later instead).

**Never overwrite.** Existing files are left alone or changed only after showing a diff and getting a yes. The single exception is appending the Donatello block to an existing `CLAUDE.md`, which adds without changing anything.

**Never duplicate.** If the project already documents something (architecture, current behavior, conventions), point to it instead of writing it again. A rule or fact should live in one file.

Paths below that start with `../` are relative to this skill's directory.

## 1. Check what's already here

Look for `CLAUDE.md` (and whether it has `<!-- donatello:start -->`), `AGENTS.md`, `docs/preferences.md`, `docs/architecture.md`, `docs/requirements.md`, `.claude/rules/`, and `.spec/`. Match names case-insensitively: `docs/ARCHITECTURE.md` counts as `docs/architecture.md`.

**Ask whether they're new.** Unless `--defaults`, ask "Are you new to this repo?" once, before the branches below. Default to yes when there's no git repo or no commits, when `git config user.email` is empty (an empty `--author` matches every commit), or when `git log --author="$(git config user.email)" --oneline | wc -l` is under 5% of `git rev-list --count HEAD`; otherwise default to no. Ask even with a default, so a wrong guess costs one answer. A yes carries forward as newcomer mode in steps 4–7 (non-onboarded repos only; an onboarded repo never enters newcomer mode, even when the user is new).

- **Already onboarded** (block or `.spec/` present): if the user is new, say the repo is already set up and offer `/donatello:tour` (recommended) or resume mode. Before resume mode, warn that the preferences refresh and block update change shared, committed files. Otherwise switch to resume mode: report what exists and what's missing, then offer only the missing pieces (including skill candidates from step 3), a preferences refresh (shown as a diff against the current Donatello preferences, deduplicated as in step 3), or a block update. Don't redo anything that exists. The doc paths are the ones the existing block names. With `--defaults`, skip the question and go straight to resume mode.
- **Only `AGENTS.md`, no `CLAUDE.md`**: ask whether the block goes into `AGENTS.md` or a new `CLAUDE.md`.

## 2. Survey with the scout

Unless the repo is empty, dispatch the `donatello:scout` agent and work from its report. **Don't read the codebase yourself**; that's what the scout is for. Read individual files only to settle a specific question its report leaves open.

If the report shows several apps or stacks (a monorepo), ask which part to onboard before going further.

**Map existing docs to roles.** Donatello uses three docs: **architecture** (what the project is, modules, patterns, build and test commands, decisions), **requirements** (current behavior), and **preferences** (coding conventions). Use the scout's "Existing docs & agent files" to decide where each one lives:
- **Covered by an existing file**, whatever its name or location (`docs/DESIGN.md`, `CONTRIBUTING.md`, `docs/adr/`, …): adopt that file. Don't create Donatello's version. The block points to the adopted file, later skills write to it, and anything it's missing is proposed as a diff to it. If several files share a role, adopt the closest match and ask when it's unclear. Don't consolidate them unless the user asks.
- **Covered only inside `CLAUDE.md` or `AGENTS.md`**: adopt it for preferences, but not for architecture or requirements, because those files load every session and `phase-archive` keeps adding to these docs. Create the default file instead, and leave out anything the agent file already says (see 6c).
- **Not covered**: use the default path, `docs/architecture.md`, `docs/requirements.md`, or `docs/preferences.md`.

Then read `CLAUDE.md`, `AGENTS.md`, `.claude/rules/`, and every adopted file yourself. They are docs rather than code, and you need their exact wording to avoid repeating them.

## 3. Pick preferences

Read the frontmatter of every file in `../../preferences/`. Select the files whose `applies-to` includes this project's stack, plus files with no `applies-to`. Sort them:
- **general** (no `paths`) → the preferences doc (see 6a)
- **path-scoped** (has `paths`) → `.claude/rules/<category>.md`

Read the bodies of the selected files too; you need them to plan phase 1.

**Deduplicate against the project.** Compare each selected preference, rule by rule, with the conventions already written in `CLAUDE.md`, `AGENTS.md`, `.claude/rules/`, and the adopted preferences file:
- **Already stated** (the same rule in any wording): drop it.
- **Conflicts** (the project says otherwise): the project's rule wins by default. Note the conflict for the interview.
- **New**: keep it.

Drop individual rules, not whole files; the rest of a file's section stays. The same applies to path-scoped preferences: if `.claude/rules/<category>.md` already exists, only the rules it lacks are candidates for merging in.

**Skill candidates** (skip with `--defaults`). Follow the import's Candidates section in `../skills/SKILL.md` for the stack the scout found, stopping before its display step. Step 4 shows the list.

## 4. Interview (skip with `--defaults`)

Keep it short: about five questions in total, one or two at a time, and never ask what the scout already answered. Cover:
- what's being built and where it's going (new projects: the core idea; existing: what's next; newcomer mode: "Do you know the next piece of work? 'No' is fine")
- anything the scout's "Open questions" flagged, especially architecture specifics the tasks will depend on (DI, state management, navigation). Newcomer mode: show each as a finding to confirm, correct, or mark "don't know"
- the preference selection: list each file's `summary` and ask if any should be dropped or changed for this project
- each conflict from step 3: keep the project's rule (the default) or switch to Donatello's. Switching means editing the project's rule where it already lives, not adding a second one. Newcomer mode: keep the project's rule without asking
- skills (skip with `--defaults`, or when there are no candidates): show the import's Candidates list, grouped by source with "(installs the whole plugin)" on plugin entries, then ask the Ask questions in `../skills/SKILL.md`: all / pick / none, then committed / just me. Pick ids come only from that list.

## 5. Propose (skip approval with `--defaults`)

Show one compact proposal and wait for a yes or changes:
- where each doc lives: an adopted existing file or a new default file
- the architecture outline you'll write (a few bullets), or the additions to the adopted file
- preferences: what goes to the preferences doc and what to `.claude/rules/`, plus how many rules were dropped as already stated (with a couple of examples)
- phase breakdown: phase 1 with its goal and rough tasks; later phases as one-liners (unless docs only)
- newcomer mode: the conflicts kept from step 3, listed, and a choice of "docs only" or "docs + phase 1"
- skill candidates, and the ones chosen with their scope (omit when none; see step 6g)
- plugins to disable for this project (see step 6h)
- every file you'll create, and every existing file you'll touch and how

## 6. Write

**a. Preferences doc.** Write only the general preferences that survived step 3. If none survived, don't create the file at all. If the preferences doc is a new `docs/preferences.md`, start it with `<!-- Copied from Donatello preferences on <YYYY-MM-DD>; edit freely for this project. -->`, then add the kept rules from each general file (frontmatter removed) as their own sections. If the preferences doc is an adopted file other than `CLAUDE.md`/`AGENTS.md`, show the kept rules as a diff to that file and merge only on approval. If it was adopted from `CLAUDE.md`/`AGENTS.md`, put the kept rules in a new `docs/preferences.md` rather than growing the agent file. Keep Donatello's part at 100 lines or less; if it's over, point out the longest sections and suggest moving file-specific ones into `.claude/rules/`.

**b. `.claude/rules/<category>.md`.** For each path-scoped preference with rules left after step 3: the body, with frontmatter reduced to the `paths` key only (that's the key Claude Code reads for rules). If the file already exists, show the new rules as a diff.

**c. Architecture doc.** If it's a new file: under 80 lines, from the scout report and the interview, covering what the project is, modules, stack, key patterns (state, DI, navigation, testing), **exact build and test commands** (the agents rely on these), and decisions already made. Leave out anything `CLAUDE.md`/`AGENTS.md` already states. Where that's a section the agents need (e.g. build commands), write a one-line pointer such as `Build and test commands: see CLAUDE.md`. If it's adopted: propose only what's missing, as a diff. Build and test commands are the most important gap to fill. Newcomer mode: put "don't know" answers and unresolved scout questions under a final `## Open questions` section (at most 5 lines), as a diff if adopted.

**d. Requirements doc.** If it's adopted, leave it alone. Otherwise create `docs/requirements.md` only for existing projects, and only when current behavior is clear from the scout report: plain requirement statements grouped by area. If not, don't create it at all, not even as a placeholder; the first `phase-archive` creates it.

**e. Phase 1.** Follow `../phase-new/SKILL.md` from its step 2, using its templates. Newcomer mode with "docs only": skip this step. You already have the context, so don't repeat its step 1. In resume mode with archived phases, numbering continues from them.

**f. CLAUDE.md block.** Read `../../templates/claude-block.md`. Replace its default doc paths with the ones from step 2; the other skills and agents find the docs through these lines. Preferences line: import with `@` only a preferences file of 100 lines or less. Name a longer one without `@` ("read before coding"). If there's no separate preferences file (none survived, or they live in `CLAUDE.md` itself), keep only the pointer to `.claude/rules/`. Keep the requirements path even if the file doesn't exist yet; `phase-archive` creates it there.
- No `CLAUDE.md`: create it with `# <Project name>` and the block.
- `CLAUDE.md` without the markers: append a blank line and the block at the end. Change nothing else.
- `CLAUDE.md` with the markers: show a diff of the region between them and replace it only on approval. Never touch content outside the markers.

**g. Skills** (skip with `--defaults`; runs before h, so pruning never proposes a plugin imported here). If the user chose none, skip this. Otherwise follow the import's Install section in `../skills/SKILL.md` for the chosen ids and scope, in this project. Its committed-file paths go into step 7.

**h. Plugin pruning** (skip with `--defaults`; after g). Run `claude plugin list --json` and take the enabled plugins. Never propose a plugin imported in step 6g, or any catalog plugin the Skills step listed as already present. Propose disabling the ones clearly irrelevant to this project's stack (e.g. web hosting, SEO, or marketing plugins in an Android app). Never propose `donatello` itself. For each one the user confirms, run `claude plugin disable <id> --scope local`; this writes `.claude/settings.local.json`, so it affects only this project and only this user. Make sure `.claude/settings.local.json` is listed in `.gitignore`.

## 7. Wrap up

Summarize what was created and changed in a few lines, add the import summary from `../skills/SKILL.md` (imported by scope, already present, failed with commands; with `--defaults`, just the `/donatello:skills import` pointer), name the next step (`/donatello:phase-run` to start phase 1's first task, or `/donatello:phase-new` when no phase 1 was written), in newcomer mode offer `/donatello:tour`, and offer to commit with `docs: onboard project to Donatello`. In newcomer mode, note that `CLAUDE.md` and `docs/` are shared, so a PR may suit better than a direct commit. Include the committed skill files from step 6g in that commit, plus the `.gitignore` lines added for "just me" imports (so the ignores are shared). Leave out the ignored "just me" skill folders and `.claude/settings.local.json`. Commit only if the user agrees.
