# Donatello Architecture

## Overview

Donatello is organized into three functional areas:

1. **CLI Layer** — Commands users run (`donatello onboard`, `donatello skills list`, etc.)
2. **Core Toolkit** — Detectors, scaffolders, and processors that power the commands
3. **Data Layer** — Preferences, skills, and templates stored in the Donatello repo

---

## Component Design

### CLI Layer

**Entry Point:** `donatello/cli/main.py`

Commands:
- `donatello onboard [--path] [--no-interview]` — Main workflow entry
- `donatello skills list [--active]` — List available skills
- `donatello phase create <name> [--scope]` — Create phase template
- `donatello phase archive <name>` — Archive completed phase
- `donatello validate [--path]` — Validate `.spec/` structure

Each command is a thin wrapper that delegates to core toolkit components.

### Core Toolkit

**Project Detector**
- Reads `.git/`, `README.md`, `package.json`, `build.gradle`, etc.
- Detects: project type (Android, Web, Python, iOS), stage (NEW vs EXISTING), language
- Used by: onboard workflow to tailor questions

**GitHub Inspector** (optional)
- Fetches project description from GitHub URL if available
- Reads GitHub issue labels to understand project scope
- Used by: interview to ask better questions

**Interview Engine**
- Runs interactive Q&A session
- Questions are customized based on:
  - Project type (Android asks about Kotlin/Compose, Web asks about frameworks)
  - Project description (tailors scope questions)
  - User's stored preferences (understands their style)
- Returns: phase breakdown plan, project overview

**Preference Loader**
- Reads from `donatello/preferences/<category>-preferences.md`
- Returns preferences as structured data
- Used by: onboard (to copy to project), skills manager (to tag relevant skills), task suggester (to tune suggestions)

**Skills Manager**
- Reads from `donatello/skills/<category>/`
- Resolves external skill links (e.g., GitHub URLs, shared skill repos)
- Filters: returns only active skills for current project
- Used by: CLAUDE.md generator, `skills list` command

**Scaffolder**
- Creates directory structure:
  - `.spec/ongoing/phaseX/` directories
  - Template files: `overview.md`, `design.md`, `spec.md`, `tasks.md`
- Populates templates with:
  - Phase scope (from interview or user input)
  - Auto-suggested tasks (from task suggester)
  - Links to relevant preferences and skills
- Used by: onboard workflow

**Task Suggester**
- Suggests tasks for a phase based on:
  - Phase name/scope (e.g., "implement auth" → login, logout, password reset, etc.)
  - User's preferences (e.g., TDD preference adds test tasks first)
  - Project type (Android, Web patterns differ)
- Returns: ordered task list (test-first if TDD preferred)
- Used by: scaffolder to populate `tasks.md`

**CLAUDE.md Generator**
- Creates agent context file at project root
- Includes:
  - Brief project overview
  - Skills section: links to local + external skills
  - Preferences summary: key rules from `docs/preferences.md`
  - Pointer to `.spec/` structure
- Goal: minimize context overhead; Claude reads `.spec/phaseX/` + `docs/preferences.md` per phase

### Data Layer

**Preferences** (`donatello/preferences/`)
- Organized by category: testing, preview, naming, dependency-injection, etc.
- Each file describes: how you prefer to code
- Copied to project's `docs/preferences.md` during onboard
- Example sections: "TDD-first development", "Preview workflows", "Naming conventions"

**Skills** (`donatello/skills/`)
- Organized by category: android, web, general
- Each skill describes: conventions, patterns, examples, links to external docs
- Can be standalone or reference external skill with overrides
- Example: "Jetpack Compose patterns (with root component navigation preference)"

**Templates** (`donatello/templates/`)
- Markdown templates for phase files:
  - `overview.md` — scaffold with phase scope + links to docs/
  - `design.md` — scaffold with decision structure (What we chose, Why, Tradeoffs)
  - `spec.md` — scaffold with phase-specific requirements sections
  - `tasks.md` — scaffold with task checklist + auto-suggested tasks

---

## Data Flow

### Onboarding a New Project

```
User runs: donatello onboard

1. Project Detector
   └─> Detects project type, stage, language

2. GitHub Inspector (if GitHub URL found)
   └─> Fetches description, issues, topics

3. Interview Engine
   ├─> Customizes questions based on project type + GitHub info
   ├─> Asks user to describe scope, tech stack, phase breakdown
   └─> Returns: project plan, phase structure

4. Preference Loader
   └─> Loads user's preferences from donatello/preferences/

5. Scaffolder
   ├─> Creates .spec/ongoing/phaseX/ for each phase
   ├─> Task Suggester: generates task list per phase
   ├─> Populates templates (overview, design, spec, tasks)
   ├─> Injects phase scope + suggested tasks
   └─> Creates docs/ directory with preferences.md

6. CLAUDE.md Generator
   ├─> Links to donatello/skills/ (local) + external skills
   ├─> Includes preferences summary
   ├─> Points to .spec/ structure
   └─> Creates CLAUDE.md or AGENT.md at project root

7. Output
   └─> User has: .spec/, docs/, CLAUDE.md, ready to hand to Claude
```

### Skills Management

```
User runs: donatello skills list

1. Skills Manager
   ├─> Reads donatello/skills/ (local)
   ├─> Resolves external skill links (if in a project, filters to active)
   └─> Returns: organized list with descriptions

2. Output
   └─> Plain text, indented, with brief descriptions
```

---

## Design Decisions

### Why Preferences are Copied, Not Linked

- **Copied:** User can customize per-project preferences during onboard
- **Alternative (linked):** Would require runtime resolution of external preferences; complicates agent integration
- **Tradeoff:** Slight duplication, but cleaner project structure and easier for agents to consume

### Why Skills Aren't Stored in `.spec/`

- **Current:** Skills stored in Donatello repo; `.spec/` links to them
- **Alternative:** Copy skills into each project
- **Tradeoff:** Avoids duplication and marketplace dependency complexity; users manage skill publishing independently

### Why Templates are Markdown, Not Code

- **Current:** Phase structure is plain markdown files
- **Alternative:** Generate code scaffolding, generated phase configs, etc.
- **Tradeoff:** Markdown is human-readable, agent-readable, version-controllable; no special parsing needed

### Why Tasks are Suggested, Not Mandated

- **Current:** Donatello suggests tasks based on patterns; user edits
- **Alternative:** Auto-generate full task list
- **Tradeoff:** Gives control to user; Donatello is advisory, not directive

---

## Component Interfaces

### ProjectDetector
```
detect_type(path: Path) -> ProjectType
  Returns: ANDROID | WEB_REACT | WEB_NEXT | PYTHON | GO | RUST | UNKNOWN

detect_stage(path: Path) -> ProjectStage
  Returns: NEW | EXISTING

detect_language(path: Path) -> Language
  Returns: KOTLIN | TYPESCRIPT | PYTHON | etc.
```

### InterviewEngine
```
run_interview(project_type: ProjectType, github_info: dict) -> InterviewResult
  Returns: {
    "project_overview": str,
    "phases": [{"name": str, "scope": str, "order": int}],
    "tech_stack": [str],
    "preferences_agreed": bool
  }
```

### PreferenceLoader
```
load_preferences(category: str = None) -> dict[str, Preference]
  Returns: preferences by category or all if category is None

get_preference(key: str) -> Preference or None
```

### SkillsManager
```
list_skills(active_only: bool = False) -> list[Skill]
  Returns: skills with descriptions, sources (local/external)

get_skill(name: str) -> Skill or None
```

### Scaffolder
```
scaffold_project(path: Path, interview_result: InterviewResult) -> None
  Creates: .spec/, docs/, CLAUDE.md structure

scaffold_phase(path: Path, phase_name: str, scope: str) -> None
  Creates: .spec/ongoing/phaseX/ with templates + suggested tasks
```

### TaskSuggester
```
suggest_tasks(phase_name: str, phase_scope: str, project_type: ProjectType, preferences: dict) -> list[Task]
  Returns: ordered task list (respects TDD preference, etc.)
```

---

## Integration Points

### With Claude / Agent
- Agent reads: `CLAUDE.md` (entry point) → `docs/preferences.md` (rules) → `.spec/ongoing/phaseX/` (current work)
- Agent updates: `.spec/phaseX/tasks.md` (marks tasks complete), `.spec/phaseX/design.md` (adds handoff notes)
- Agent archives: runs `donatello phase archive` when phase is complete

### With GitHub
- Optional: fetch project description to tailor interview questions
- Optional: read GitHub issues to understand current work priorities

### With External Skills
- Skills can reference external GitHub repos, published marketplace entries, etc.
- Donatello does NOT fetch or manage external skills; it just links

---

## Error Handling & Edge Cases

### Project Type Ambiguity
- If project matches multiple types (e.g., monorepo): ask user to clarify, don't guess

### Missing/Incomplete GitHub Info
- Gracefully fall back to generic questions if GitHub is unreachable

### Preference Conflicts
- If user disagrees with preferences during onboard: show diffs, let user customize

### Phase Naming Collisions
- If phaseX already exists: warn, ask to rename or overwrite

### Skills Link Rot
- If external skill link is broken: note in output, suggest user fix or remove
