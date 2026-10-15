# Phase 1: MVP — Core Scaffolding & CLI

## Goal

Build the minimum viable product: a working `donatello onboard` command that detects projects, loads your preferences, scaffolds `.spec/` structure, and generates `CLAUDE.md` context.

## What This Phase Delivers

- **CLI interface** for `donatello onboard` and `donatello skills list` commands
- **Project detection** (type, stage, language)
- **Preference system** (load from Donatello repo, copy to project)
- **Basic scaffolding** (create `.spec/ongoing/phase1/` + `docs/` + templates)
- **CLAUDE.md generation** (link to skills, preferences summary, project pointers)
- **Simple interview** (skip for now; default mode only in MVP)

## Out of Scope for MVP

- GitHub integration (issue inspection, project description fetching)
- Advanced interview with customization per project type
- Task auto-suggestion (MVP uses static task templates)
- External skill resolution (MVP supports local skills only)
- Phase archiving and validation commands

## Success Criteria

- `donatello onboard .` runs without errors
- `.spec/`, `docs/`, and `CLAUDE.md` are created at project root
- `docs/preferences.md` contains copy of user's preferences
- `CLAUDE.md` lists local skills available
- CLI is accessible: `donatello --help` shows commands

## Related Docs

See `docs/SPECIFICATION.md` for full feature set and `docs/ARCHITECTURE.md` for component design.
