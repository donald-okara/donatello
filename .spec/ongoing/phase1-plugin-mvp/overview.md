# Phase 1: Plugin MVP

## Goal

Ship an installable `donatello` Claude Code plugin whose three workflow skills (`onboard`, `phase-new`, `phase-archive`) take a project from nothing to a working spec-driven structure and keep it consistent across phases.

## What This Phase Delivers

- Plugin and marketplace manifests; installable from GitHub
- `/donatello:onboard`: inspect, interview (skippable), propose, write `docs/`, phase 1, and the CLAUDE.md block
- `/donatello:phase-new`: next-numbered phase from templates, with tasks shaped by preferences
- `/donatello:phase-archive`: close out, record the outcome, move to the archive
- Phase templates and the CLAUDE.md block template
- A starter set of real preferences (at least testing and naming, plus one stack-specific file)

## Out of Scope

- Domain skills (Compose patterns, etc.): added incrementally after the workflow is proven
- Splitting into per-stack plugins
- A validation script or hook
- Automated skill evals (`claude plugin eval`); revisit once the skills settle

## Success Criteria

- `claude plugin validate .` passes, and the plugin installs via `/plugin marketplace add` + `/plugin install`
- Onboarding a real **new** project and a real **existing** project (one Android, one web) produces correct files with nothing overwritten
- A fresh session in an onboarded project states the active phase and key preferences when asked "what are we working on?"
- Running phase-new → phase-archive → phase-new yields `phase2-*` in ongoing and a dated `phase1-*` in the archive
- The CLAUDE.md block is under 300 tokens

## Related Docs

`docs/SPECIFICATION.md` (behavior) · `docs/ARCHITECTURE.md` (layout, components, edge cases)

## Outcome

_Filled in on archive._
