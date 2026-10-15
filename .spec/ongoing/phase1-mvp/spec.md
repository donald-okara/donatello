# Phase 1: Specification

## Requirements

### R1. Project Detection

**Requirement:** Donatello must detect project type and stage.

**Scope:**
- Types: Android (Kotlin + Gradle), Web (React/Next + Node), Python, Go, Rust, Unknown
- Stage: NEW (empty/minimal git history) or EXISTING (established history)
- Language: Kotlin, TypeScript/JavaScript, Python, etc.

**Validation:** Given a directory with `build.gradle` and `AndroidManifest.xml`, must detect as Android.

### R2. Preference Loading

**Requirement:** Load user preferences from Donatello repo and structure them for copying.

**Scope:**
- Read from `donatello/preferences/<category>-preferences.md`
- Support categories: testing, preview, naming, dependency-injection, etc.
- Parse as structured data (heading + bullet points or YAML frontmatter)
- Return as dict keyed by category

**Validation:** Load testing-preferences.md, confirm structure is preserved.

### R3. Skills Listing

**Requirement:** List available local skills with descriptions.

**Scope:**
- Scan `donatello/skills/` recursively
- Group by category (android, web, general)
- Extract skill name and description (from file header or metadata)
- Support simple metadata format (Markdown heading + first paragraph)

**Validation:** `donatello skills list` outputs organized skill list with descriptions.

### R4. Project Scaffolding

**Requirement:** Create `.spec/`, `docs/`, and template files.

**Scope:**
- Create `.spec/ongoing/phase1/` with four template files:
  - `overview.md` — phase scope (auto-populated with project name)
  - `design.md` — decision template
  - `spec.md` — requirement template
  - `tasks.md` — empty task checklist
- Create `docs/` directory
- Create `docs/preferences.md` with copy of user's preferences
- Create `.spec/` structure directories if they don't exist

**Validation:** After scaffolding, `.spec/ongoing/phase1/` and `docs/` exist with correct files.

### R5. CLAUDE.md Generation

**Requirement:** Generate agent context file at project root.

**Scope:**
- Create `CLAUDE.md` or `AGENT.md` (configurable, default CLAUDE.md)
- Include sections:
  - **Project Overview** — brief description (from README or user input)
  - **Preferences** — summary of key preferences (e.g., "Test-first development, Jetpack Compose patterns")
  - **Skills** — list of available skills with categories (link format: `- [Skill Name](../../donatello/skills/category/skill.md)`)
  - **Structure** — pointer to `.spec/` and `docs/`
- Minimize context; no full documentation inline

**Validation:** `CLAUDE.md` is created, contains skill links and preferences summary.

### R6. CLI Commands

**Requirement:** Implement minimum CLI interface.

**Scope:**
- Command: `donatello onboard [--path .] [--no-interview]` (MVP defaults to `--no-interview`)
- Command: `donatello skills list [--active]`
- Help text: `donatello --help`
- Error handling: graceful messages for missing files, permission issues, etc.

**Validation:** CLI is callable; `--help` shows available commands.

### R7. Error Handling

**Requirement:** Handle common error cases gracefully.

**Scope:**
- Project not found → clear message: "Project not found at <path>"
- Already has `.spec/` → ask: "Project already scaffolded. Overwrite? (y/n)"
- GitHub repo detection fails → warn but continue
- Missing preferences directory → use defaults

**Validation:** Each error case produces a clear, actionable message.

## Non-Functional Requirements

### NF1. Token Efficiency

Generated CLAUDE.md should be < 1000 tokens (excluding linked files).

### NF2. Performance

`donatello onboard` should complete in < 5 seconds on typical project.

### NF3. Portability

Must work on macOS, Linux, Windows (via WSL or native Python).

### NF4. No External Dependencies During Scaffolding

Scaffolding must not require network calls (GitHub API, etc.) in MVP.
