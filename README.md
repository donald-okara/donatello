# donatello

A Claude Code plugin with my favorite AI workflows: onboarding an agent to a new or existing project, organizing work into spec-driven phases, and carrying my coding preferences into every project.

## Commands

- `/donatello:onboard`: interview, propose phases, scaffold `docs/`, `.spec/`, and a CLAUDE.md block
- `/donatello:phase-new [scope]`: start the next phase from templates
- `/donatello:phase-run [task | --all]`: run tasks with Haiku implementing and Sonnet reviewing, while Opus coordinates
- `/donatello:phase-archive [phase]`: close out a phase and move it to the archive

## Install

```
/plugin marketplace add donald-okara/donatello
/plugin install donatello@donatello
```

See `docs/SPECIFICATION.md` and `docs/ARCHITECTURE.md` for details. Status: phase 1 in progress (`.spec/ongoing/phase1-plugin-mvp/`).
