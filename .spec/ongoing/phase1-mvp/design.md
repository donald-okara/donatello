# Phase 1: Design Notes

## Architecture Decisions

### MVP Scope: Why No Interview Yet?

**Decision:** Start with defaults-only mode (skip interactive interview).

**Why:** Simplifies implementation; proves scaffolding works before adding conversational intelligence.

**Tradeoff:** Users can't customize phases during onboard in MVP; they'll edit `.spec/` files manually if needed. Phase 2 adds interview mode.

### Preferences: Copy vs Link

**Decision:** Copy preferences from Donatello repo to `docs/preferences.md` in new project.

**Why:** Projects are self-contained; no runtime resolution needed.

**Tradeoff:** Slight duplication, but cleaner for agents to consume.

### Skills: Local Only in MVP

**Decision:** MVP supports only local skills from `donatello/skills/`.

**Why:** Reduces complexity; external skill resolution can wait for Phase 2.

**Tradeoff:** Can't reference external skills in Phase 1; workaround: user manually adds external skill links to `CLAUDE.md`.

### Task Suggestions: Static Templates

**Decision:** Use predefined task templates per phase scope, not dynamic generation.

**Why:** Faster to implement; covers common cases.

**Tradeoff:** Less personalization; Phase 2 adds preference-aware task suggestion.

## Implementation Approach

### Phase 1 Components

1. **ProjectDetector** — detect type, stage, language (read markers)
2. **PreferenceLoader** — read YAML/Markdown preferences from `donatello/preferences/`
3. **SkillsManager** — list local skills from `donatello/skills/`
4. **Scaffolder** — create `.spec/`, `docs/`, template files
5. **CLAUDEGenerator** — create `CLAUDE.md` with skills + preferences summary
6. **CLI** — `onboard` and `skills list` commands

### File Organization

```
src/donatello/
├── cli/
│   └── main.py                    # Entry point, command routing
├── detectors/
│   └── project.py                 # ProjectDetector
├── loaders/
│   └── preferences.py             # PreferenceLoader
├── managers/
│   └── skills.py                  # SkillsManager
├── generators/
│   ├── scaffolder.py              # Scaffolder
│   └── claude.py                  # CLAUDEGenerator
├── models/
│   ├── project.py                 # ProjectType, ProjectStage enums
│   ├── preference.py              # Preference model
│   └── skill.py                   # Skill model
└── utils/
    ├── paths.py                   # Path helpers
    └── io.py                       # File I/O helpers
```

### Key Interfaces (High-Level)

- `ProjectDetector.detect(path) -> ProjectInfo` — returns type, stage, language
- `PreferenceLoader.load() -> dict[Preference]` — loads all preferences
- `SkillsManager.list_skills() -> list[Skill]` — lists local skills
- `Scaffolder.scaffold(path, project_info) -> None` — creates structure
- `CLAUDEGenerator.generate(path, skills, preferences) -> None` — creates CLAUDE.md
- `CLI.onboard(path, no_interview=True)` — main entry point

## Testing Strategy

- Unit tests: detectors, loaders, generators (mock file I/O)
- Integration test: full `donatello onboard` workflow on sample project
- Manual test: run on real project, inspect output

## Handoff to Next Phase

After MVP is complete:
- Phase 2 adds interview mode (conversational phase planning)
- Phase 3 adds GitHub integration
- Phase 4 adds task auto-suggestion
- Phase 5 adds external skill resolution
