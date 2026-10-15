# Phase 1: Tasks

## Overview

This checklist breaks Phase 1 into testable, committable tasks. Each task is test-first: write failing test, implement, verify, commit.

## Task Dependencies

```
1. Setup (no deps)
  ├─> 2. ProjectDetector tests + impl
  ├─> 3. PreferenceLoader tests + impl
  └─> 4. SkillsManager tests + impl
       └─> 5. Scaffolder tests + impl
       └─> 6. CLAUDEGenerator tests + impl
             └─> 7. CLI tests + impl
                  └─> 8. Integration test
                       └─> 9. Manual test
```

---

## Tasks

### ✅ Task 1: Project Setup & Package Structure

**Scope:** Ensure Donatello package is installable and testable.

- [ ] Verify `setup.py` and `pyproject.toml` have correct metadata
- [ ] Install package in editable mode: `pip install -e .`
- [ ] Verify `donatello --help` shows (stub) commands
- [ ] Verify pytest is configured and runs: `pytest tests/ -v` (empty tests OK for now)
- [ ] Commit: `git add setup.py pyproject.toml src/ tests/ && git commit -m "chore: verify project setup"`

---

### ✅ Task 2: ProjectDetector — Detect Type & Stage

**Scope:** Implement project type/stage detection.

**Files:**
- `src/donatello/models/project.py` — Enums: ProjectType, ProjectStage, Language
- `src/donatello/detectors/project.py` — ProjectDetector class
- `tests/test_project_detector.py` — Tests

**Tests to Write:**
- `test_detect_android_project()` — given `build.gradle` + `AndroidManifest.xml`, returns ANDROID
- `test_detect_web_react_project()` — given `package.json` + React config, returns WEB_REACT
- `test_detect_web_next_project()` — given `package.json` + `next.config.js`, returns WEB_NEXT
- `test_detect_python_project()` — given `pyproject.toml` or `setup.py`, returns PYTHON
- `test_detect_new_project()` — given empty dir or 1 commit, returns NEW
- `test_detect_existing_project()` — given multiple commits + src files, returns EXISTING
- `test_detect_language()` — given Kotlin files, returns KOTLIN; TypeScript files → TYPESCRIPT

**Commit:** `git add src/donatello/models/ src/donatello/detectors/ tests/ && git commit -m "feat: add project type and stage detection"`

---

### ✅ Task 3: PreferenceLoader — Load & Expose Preferences

**Scope:** Load preferences from Donatello repo; structure for copying.

**Files:**
- `src/donatello/models/preference.py` — Preference model
- `src/donatello/loaders/preferences.py` — PreferenceLoader class
- `tests/test_preference_loader.py` — Tests

**Tests to Write:**
- `test_load_preferences()` — load from `donatello/preferences/`, return dict by category
- `test_load_category()` — load single category (e.g., testing-preferences.md)
- `test_parse_markdown_preferences()` — parse headings + bullets from .md files
- `test_copy_to_project()` — copy preferences to `docs/preferences.md` in new project
- `test_empty_preferences()` — handle missing preferences dir gracefully

**Commit:** `git add src/donatello/models/ src/donatello/loaders/ tests/ && git commit -m "feat: add preference loading"`

---

### ✅ Task 4: SkillsManager — List & Describe Skills

**Scope:** List local skills with descriptions.

**Files:**
- `src/donatello/models/skill.py` — Skill model
- `src/donatello/managers/skills.py` — SkillsManager class
- `tests/test_skills_manager.py` — Tests

**Tests to Write:**
- `test_list_local_skills()` — scan `donatello/skills/`, return all skills
- `test_skills_by_category()` — group skills by category (android, web, general)
- `test_extract_skill_description()` — parse description from skill .md file header
- `test_format_skill_list()` — format as plain text with indentation
- `test_empty_skills_dir()` — handle missing skills directory

**Commit:** `git add src/donatello/models/ src/donatello/managers/ tests/ && git commit -m "feat: add skills management"`

---

### ✅ Task 5: Scaffolder — Create `.spec/` & `docs/`

**Scope:** Create directory structure and template files.

**Files:**
- `src/donatello/generators/scaffolder.py` — Scaffolder class
- `src/donatello/templates/` — Template files (.md)
- `tests/test_scaffolder.py` — Tests

**Templates to Create:**
- `src/donatello/templates/phase_overview.md`
- `src/donatello/templates/phase_design.md`
- `src/donatello/templates/phase_spec.md`
- `src/donatello/templates/phase_tasks.md`

**Tests to Write:**
- `test_scaffold_new_project()` — create `.spec/ongoing/phase1/` + `docs/` + templates
- `test_scaffold_creates_directories()` — verify directories exist after scaffolding
- `test_scaffold_with_preferences()` — copy `docs/preferences.md` to project
- `test_scaffold_already_exists()` — handle case where `.spec/` already exists
- `test_populate_templates()` — verify templates have project info injected

**Commit:** `git add src/donatello/generators/scaffolder.py src/donatello/templates/ tests/ && git commit -m "feat: add project scaffolding"`

---

### ✅ Task 6: CLAUDEGenerator — Create Agent Context File

**Scope:** Generate `CLAUDE.md` with skills + preferences summary.

**Files:**
- `src/donatello/generators/claude.py` — CLAUDEGenerator class
- `tests/test_claude_generator.py` — Tests

**Tests to Write:**
- `test_generate_claude_md()` — create `CLAUDE.md` at project root
- `test_include_skills_section()` — CLAUDE.md has skills list with links
- `test_include_preferences_summary()` — CLAUDE.md summarizes key preferences
- `test_include_structure_pointer()` — CLAUDE.md points to `.spec/` and `docs/`
- `test_claude_md_token_count()` — verify < 1000 tokens (rough check)
- `test_format_skill_links()` — verify links point to correct paths

**Commit:** `git add src/donatello/generators/claude.py tests/ && git commit -m "feat: add CLAUDE.md generation"`

---

### ✅ Task 7: CLI — Implement `onboard` & `skills list` Commands

**Scope:** Wire up CLI to call toolkit components.

**Files:**
- `src/donatello/cli/main.py` — Update with real implementations
- `tests/test_cli.py` — Tests

**Tests to Write:**
- `test_onboard_command()` — `donatello onboard .` creates `.spec/` + `docs/` + `CLAUDE.md`
- `test_onboard_with_path()` — `donatello onboard /some/project` works
- `test_skills_list_command()` — `donatello skills list` prints skills
- `test_help_text()` — `donatello --help` shows all commands
- `test_error_project_not_found()` — clear error if path doesn't exist
- `test_error_already_scaffolded()` — warn if `.spec/` already exists

**Commit:** `git add src/donatello/cli/ tests/ && git commit -m "feat: implement CLI commands"`

---

### ✅ Task 8: Integration Test — Full Workflow

**Scope:** Test complete `donatello onboard` workflow on sample project.

**Files:**
- `tests/fixtures/sample_android_project/` — minimal Android project
- `tests/fixtures/sample_web_project/` — minimal Web project
- `tests/test_integration.py` — Integration tests

**Tests to Write:**
- `test_onboard_android_project()` — scaffold sample Android project, verify all files created
- `test_onboard_web_project()` — scaffold sample Web project, verify all files created
- `test_claude_md_readable()` — generated CLAUDE.md is valid markdown, readable
- `test_preferences_copied()` — preferences are in generated project
- `test_skills_linked()` — skills are correctly linked in CLAUDE.md

**Commit:** `git add tests/fixtures/ tests/test_integration.py && git commit -m "test: add integration tests"`

---

### ✅ Task 9: Manual Test & Polish

**Scope:** Test on real projects, fix rough edges.

**Checklist:**
- [ ] Run `donatello onboard` on a real Android project; inspect output
- [ ] Run `donatello onboard` on a real Web project; inspect output
- [ ] Run `donatello skills list`; verify formatting
- [ ] Test on macOS and Linux (if available)
- [ ] Verify CLAUDE.md is readable for Claude
- [ ] Check all tests pass: `pytest tests/ -v`
- [ ] Ensure no warnings or lint issues: `pylint src/`
- [ ] Verify token count of generated files

**Commit:** `git add . && git commit -m "polish: manual testing and minor fixes"`

---

## Completion Criteria

✅ All tests pass
✅ `donatello onboard` command works end-to-end
✅ Generated `.spec/`, `docs/`, `CLAUDE.md` are correct
✅ CLI is usable and has help text
✅ No external dependencies during scaffolding
✅ Token efficiency target met (CLAUDE.md < 1000 tokens)

## Next Phase

After MVP complete, Phase 2 adds:
- Interactive interview mode
- Customizable phase planning
- GitHub integration
