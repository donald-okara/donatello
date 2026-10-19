---
name: tour
description: Guided, read-only walk through a repo for someone new to it: what it is, the map, how to run it, how code is written, where work stands, and where to start. Use for "I'm new to this repo", "show me around", or "walk me through this project".
argument-hint: "[area]"
---

# Tour a repo

A read-only, chat-only orientation in six stops. The tour writes nothing: no files, no notes, no commits. Arguments: `$ARGUMENTS`

## 1. Load

- **Onboarded:** the project has a Donatello block (`<!-- donatello:start -->`) in `CLAUDE.md` or `AGENTS.md`. Read only these, and each only if it exists:
  - the architecture, requirements, and preferences docs at the paths the block names
  - the file names and `paths` frontmatter of `.claude/rules/*.md`
  - each ongoing phase's `overview.md` and the "Current state" section of its `walkthrough.md`
  - the directory names in `.spec/archive/`
  - the frontmatter of `.spec/rambles/*.md`
- **Not onboarded:** dispatch the `donatello:scout` agent and work from its report. Say that the repo isn't onboarded yet.
- **No source files** are read while loading, in either case.

## 2. Stops

Give one stop at a time. Each is at most about 10 lines and ends with "questions, deeper, or next?". Then wait for the user.

1. **What this is:** purpose, users, and stage (new or existing).
2. **Map:** modules, stack, and how they connect.
3. **Run it:** the exact build and test commands.
4. **How code is written here:** a short summary of the preferences, and which `.claude/rules` apply to which paths. Not onboarded: the scout's conventions.
5. **Where work stands:** the ongoing phase and its Current state, the names of archived phases, and open rambles. Skip this stop when not onboarded.
6. **Where to start:** 3 to 5 files to read first, then the next step.
   - Onboarded: the next step from the Current state, then `/donatello:phase-run`, which takes the next unticked task.
   - Not onboarded: `/donatello:onboard`.

Facts come only from the docs, the scout report, or files read for "deeper". Anything else is "unknown", not a guess.
