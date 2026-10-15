# Donatello Specification

## Vision

Donatello is a developer toolkit that scaffolds and manages spec-driven project structure. It provides a template for organizing projects into phases, integrates with your personal skills and preferences, and generates project onboarding context so Claude (or another AI agent) can work with you efficiently without redundant context replication across sessions.

Donatello is **not an AI tool**—it's a preference and structure manager. It holds your coding conventions, references your skills, and helps organize project phases in a way that Claude can consume with minimal token overhead.

---

## Core Concepts

### Two Components

**1. Donatello Repo** (this project)
- Central hub for your skills and preferences
- Command-line tool that scaffolds projects
- Personal toolkit; can be published to marketplace later if desired

**2. Per-Project `.spec/` Directory**
- Created when you run `donatello onboard` on a new or existing project
- Holds project phases (ongoing, archive)
- High-level documentation lives in `docs/` (architecture, design, preferences)
- Links to your skills (doesn't copy them)

### Directory Structure

#### Donatello Repo

```
donatello/
├── README.md
├── docs/
│   ├── SPECIFICATION.md          # This file
│   ├── ARCHITECTURE.md
│   └── DESIGN.md
├── src/donatello/
│   └── cli/main.py               # CLI commands
├── skills/                       # Skills you write
│   ├── android/
│   ├── web/
│   └── general/
├── preferences/                  # Your coding preferences (by category)
│   ├── testing-preferences.md
│   ├── preview-preferences.md
│   ├── naming-preferences.md
│   ├── dependency-injection-preferences.md
│   └── ...
└── .spec/                        # Donatello's own phases (dogfooding)
    ├── ongoing/
    │   └── phase1-cli-foundation/
    │       ├── overview.md
    │       ├── design.md
    │       ├── spec.md
    │       └── tasks.md
    └── archive/
```

#### Per-Project After `donatello onboard`

```
project/
├── README.md
├── docs/
│   ├── architecture.md           # High-level system overview
│   ├── design.md                 # Decisions, handoff notes
│   ├── preferences.md            # Project-specific rules, conventions
│   └── ...
├── CLAUDE.md or AGENT.md         # Agent context (skills list, preferences summary)
└── .spec/
    ├── ongoing/
    │   ├── phase1/
    │   │   ├── overview.md       # What phase 1 accomplishes
    │   │   ├── design.md         # Phase-specific decisions
    │   │   ├── spec.md           # Phase-specific requirements
    │   │   └── tasks.md          # Task checklist (auto-suggested, customizable)
    │   ├── phase2/
    │   │   └── ...
    │   └── phaseN/
    │       └── ...
    └── archive/
        ├── 2026-10-01-phase1/
        ├── 2026-10-15-phase2/
        └── ...
```

---

## Workflows

### Workflow 1: New Project Setup

**Command:** `donatello onboard <path>`

**Steps:**
1. Detect if project is new or existing
2. If new, ask if user wants an interview or defaults
3. If interview: ask customizable questions based on project type (Android, Web, iOS, etc.)
   - Inspect README and GitHub project description to tailor questions
   - Understand what's being built and how to break into phases
4. Create `.spec/` directory with phase structure
5. Copy user's preferences from Donatello repo to `docs/preferences.md` (user can customize during interview)
6. Generate `CLAUDE.md` or `AGENT.md` with:
   - Links to skills (local + external)
   - Summary of preferences
   - Pointer to `.spec/` and `docs/`
7. Suggest first phase (phase1) with auto-generated tasks based on:
   - Phase scope
   - User's preferences (e.g., always include test-first tasks if TDD is preferred)

**User Flow:**
- Runs `donatello onboard`
- Answers interview questions (or skips with defaults)
- Reviews and customizes `docs/preferences.md` if needed
- Donatello scaffolds `.spec/` with phase1 ready to go
- User can now hand the project to Claude with context already structured

### Workflow 2: Existing Project Onboarding

**Command:** `donatello onboard <path>` (on existing project)

**Steps:**
1. Detect project is existing
2. If no `.spec/`, run interview to understand the current state and future direction
3. Help user decide how to organize existing work and future work into phases
4. Create `.spec/` structure
5. Generate `CLAUDE.md` or `AGENT.md`

**User Flow:**
- Brings an existing project
- Runs `donatello onboard`
- Has a conversation with Donatello (preloaded with user's preferences) about how to organize work
- Gets a `.spec/` structure and can start adding phases

### Workflow 3: Skills Management

**Command:** `donatello skills list`

**Output:**
- Lists available skills (local from Donatello repo + external references)
- Shows which skills are active for current project (if in a project directory)
- Plain text with indentation and bullets
- Includes description for each skill

**Per-Project Skill Reference:**
- Skills are linked in `CLAUDE.md` or `AGENT.md` or optional `skills.md` at project root
- Skills are stored wherever the agent prefers (e.g., `.claude/skills/`)
- Task-specific references: within `.spec/phaseX/tasks.md`, mention if a task uses a specific skill
- Example: "Task: Create core.ui module — Reference Jetpack Compose skill for component patterns"

---

## Key Behaviors

### Preferences
- Stored in Donatello repo under `preferences/` (split by category)
- Examples: testing-preferences.md, preview-preferences.md, naming-preferences.md
- Content: How you prefer to code (TDD? Previews? DI patterns? Naming conventions?)
- Copied to project's `docs/preferences.md` during onboard; user can customize per project

### Skills
- Stored in Donatello repo under `skills/` (by category: android, web, general)
- Can be local (written by you) or external (linked from others)
- Can have overrides: use external skill + your custom preferences on top
- Skills are NOT stored in `.spec/` or project; they're referenced
- Donatello does NOT publish/manage skills; user decides marketplace publishing separately

### Tasks Auto-Generation
- When creating a phase, Donatello suggests tasks based on:
  - Phase scope/name (e.g., "implement auth" suggests login, password reset, etc.)
  - User's preferences (e.g., if TDD is preferred, suggest test-first tasks)
  - Project type (Android, Web, iOS have different task patterns)
- Tasks are suggestions; user edits and customizes
- Tasks are stored in `.spec/phaseX/tasks.md` as a checklist

### Documentation Separation
- `docs/` = high-level orientation (architecture, design, preferences)
  - No code details, no step-by-step
  - Guides context; answers "why" and "what"
- `.spec/` = phase-specific and actionable (overview, design, spec, tasks)
  - Answers "what to do" for each phase
  - Phases can reference `docs/` for orientation

### Interview Customization
- When running `donatello onboard`, interview questions adapt based on:
  - Project type detected (Android, Web, Python, etc.)
  - README and GitHub project description
  - User's stored preferences
- User can skip interview and use defaults
- Interview is a conversation, not a questionnaire; Donatello suggests, user guides

---

## CLI Commands

```bash
# Initialize Donatello in a new project (or onboard to existing)
donatello onboard [--path .] [--no-interview]

# List available skills (local + external)
donatello skills list [--active]

# Create a new phase (scaffold phase template)
donatello phase create <name> [--scope "description"]

# Archive a completed phase
donatello phase archive <phase-name>

# Validate .spec/ structure
donatello validate [--path .]
```

---

## Non-Goals

- **Donatello is not a code generator.** It generates task suggestions and structure, not code.
- **Donatello is not a context manager.** It doesn't keep state across sessions or provide a stateful API.
- **Donatello is not a skill publisher.** It manages references; users publish to marketplaces independently.
- **Donatello is not an AI.** It's rule-based suggestions and templates.

---

## Success Criteria

✅ User can run `donatello onboard` on a new project and get a structured `.spec/` with phases  
✅ User can customize preferences during onboard  
✅ User receives a generated `CLAUDE.md` that links to their skills  
✅ Phase structure is clean: `.spec/ongoing/phaseX/` with overview, design, spec, tasks  
✅ `donatello skills list` shows available skills (local + external, filtered to active)  
✅ No context duplication: high-level docs in `docs/`, phase-specific in `.spec/`  
✅ User can hand a project to Claude with all context pre-structured  
✅ Token overhead is minimal: Claude reads `.spec/phaseX/` + `docs/preferences.md`, not everything  
