# Donatello MVP Implementation Plan

> **For agentic workers:** RECOMMENDED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a developer toolkit that intelligently bootstraps new projects and onboards developers to existing projects by interviewing users, checking documentation, integrating with GitHub/Figma, and matching coding style against stored preferences.

**Architecture:** Donatello is a Python CLI tool with two main workflows. The **new project workflow** detects if a project is brand new, checks for existing documentation, and if missing, interviews the user about design/GitHub/preferences to shape the initial setup. The **onboarding workflow** analyzes a developer's current codebase, compares their patterns against the user's stored preferences (preview workflows, dependency injection, naming conventions, etc.), and reports alignment gaps and likely conflicts. A preference engine stores patterns across projects; a GitHub integrator fetches issues and repo shape; a code analyzer patterns-match against the stored style.

**Tech Stack:** Python 3.10+, Click (CLI), Typer (structured commands), Pydantic (config), OpenAI API (or local Claude via MCP) for intelligent interviews, gitpython (repo analysis), requests (GitHub/Figma APIs).

**Spec:** `/Users/donaldisoe/AI-Mployees/donatello/README.md` (high-level vision; this plan operationalizes the two workflows).

---

## Global Constraints

- Python 3.10+ required
- CLI entry point: `donatello init` and `donatello onboard` commands
- Preference store: JSON file at `~/.donatello/preferences.json`
- Project detection: infer type (Android, Web, Python, etc.) from file structure and config
- Non-destructive: do not modify user code; only analyze and report
- Interview questions must be guided by existing docs; if docs exist, skip interview

---

## File Structure

```
donatello/
├── src/
│   └── donatello/
│       ├── __init__.py
│       ├── cli/
│       │   ├── __init__.py
│       │   └── main.py                 # Entry point, init & onboard commands
│       ├── workflows/
│       │   ├── __init__.py
│       │   ├── new_project.py          # New project bootstrap flow
│       │   └── onboard.py              # Developer onboarding flow
│       ├── config/
│       │   ├── __init__.py
│       │   ├── preferences.py          # User preference model & storage
│       │   └── project_config.py       # Project config schema
│       ├── detectors/
│       │   ├── __init__.py
│       │   ├── project.py              # Detect project type & stage
│       │   └── docs.py                 # Find & parse existing docs
│       ├── integrations/
│       │   ├── __init__.py
│       │   ├── github.py               # GitHub API client
│       │   ├── figma.py                # Figma link extraction
│       │   └── claude.py               # Claude/LLM interview engine
│       ├── analyzer/
│       │   ├── __init__.py
│       │   └── codebase.py             # Code pattern analyzer
│       └── interviews/
│           ├── __init__.py
│           └── questionnaire.py        # Interview flow & questions
├── tests/
│   ├── __init__.py
│   ├── test_detectors.py
│   ├── test_analyzer.py
│   └── test_workflows.py
├── setup.py
├── pyproject.toml
├── README.md
└── .gitignore
```

---

## Review Focus

1. **User abort during interview** — If user cancels interview midway, save partial answers and exit gracefully; next run should offer resume option.
2. **Project type ambiguity** — A repo might match multiple project types (e.g., monorepo with Android + Web); detection must ask or warn, not guess.
3. **Preference match false positives** — A single matching pattern (e.g., "uses dependency injection") should not claim high alignment; scoring must weight multiple patterns.
4. **GitHub rate limits** — API calls to GitHub should cache results and warn user if approaching rate limits.
5. **Figma link extraction** — User may paste link in interview; parser must handle short links, branch links, and file vs. node IDs correctly.

---

## Task 1: Project Setup & CLI Foundation

**Files:**
- Create: `setup.py`
- Create: `pyproject.toml`
- Create: `src/donatello/__init__.py`
- Create: `src/donatello/cli/main.py`
- Create: `src/donatello/cli/__init__.py`
- Create: `tests/__init__.py`
- Create: `.gitignore`

**Interfaces:**
- Produces: CLI entry points `donatello init` and `donatello onboard` (commands registered but not yet implemented)

**Steps:**

- [ ] **Step 1: Write pyproject.toml and setup.py**

Define project metadata, dependencies (Click, Typer, Pydantic, gitpython, requests), and entry points. Package name: `donatello`, version: `0.1.0`.

```toml
[project]
name = "donatello"
version = "0.1.0"
description = "Developer toolkit for project initialization and onboarding"
requires-python = ">=3.10"
dependencies = [
    "click>=8.1",
    "typer>=0.9",
    "pydantic>=2.0",
    "gitpython>=3.1",
    "requests>=2.31",
]

[project.scripts]
donatello = "donatello.cli.main:app"
```

- [ ] **Step 2: Create `src/donatello/__init__.py`**

Single line: `__version__ = "0.1.0"`

- [ ] **Step 3: Create `src/donatello/cli/main.py`**

Define Typer app with two commands (`init` and `onboard`). Each command should be a stub that prints "TODO: implement" for now.

```python
import typer

app = typer.Typer()

@app.command()
def init(path: str = typer.Argument(".")):
    """Bootstrap a new project."""
    typer.echo(f"TODO: initialize project at {path}")

@app.command()
def onboard(path: str = typer.Argument(".")):
    """Onboard to an existing project."""
    typer.echo(f"TODO: onboard to project at {path}")

if __name__ == "__main__":
    app()
```

- [ ] **Step 4: Create `.gitignore`**

Standard Python .gitignore: `.env`, `*.pyc`, `__pycache__`, `*.egg-info`, `dist/`, `build/`, `.pytest_cache/`, `.venv/`.

- [ ] **Step 5: Install package in development mode**

Run: `pip install -e .`
Expected: Package installs without errors; `donatello --help` shows the two commands.

- [ ] **Step 6: Commit**

```bash
git add setup.py pyproject.toml src/ tests/ .gitignore
git commit -m "feat: init project structure and CLI foundation"
```

---

## Task 2: Preference System

**Files:**
- Create: `src/donatello/config/preferences.py`
- Create: `src/donatello/config/__init__.py`
- Create: `src/donatello/config/project_config.py`
- Create: `tests/test_preferences.py`

**Interfaces:**
- Produces: 
  - `PreferenceStore` class with methods: `load()`, `save()`, `get_preference(key: str)`, `set_preference(key: str, value: Any)`
  - `Preference` Pydantic model with fields: `key`, `value`, `description`, `tags` (e.g., "preview", "dependency_injection", "testing")

**Steps:**

- [ ] **Step 1: Define Preference and PreferenceStore models**

Create `src/donatello/config/preferences.py`. `Preference` is a Pydantic model with fields: `key: str`, `value: Any`, `description: str`, `tags: list[str]`. `PreferenceStore` loads/saves from `~/.donatello/preferences.json`.

```python
from pathlib import Path
from pydantic import BaseModel
from typing import Any

class Preference(BaseModel):
    key: str
    value: Any
    description: str
    tags: list[str] = []

class PreferenceStore:
    def __init__(self, path: Path = None):
        self.path = path or Path.home() / ".donatello" / "preferences.json"
    
    def load(self) -> dict[str, Preference]:
        # Load from JSON; return empty dict if file doesn't exist
        pass
    
    def save(self, preferences: dict[str, Preference]) -> None:
        # Save to JSON
        pass
    
    def get_preference(self, key: str) -> Preference | None:
        # Retrieve a single preference
        pass
    
    def set_preference(self, key: str, pref: Preference) -> None:
        # Save a single preference
        pass
```

- [ ] **Step 2: Write tests for PreferenceStore**

Test loading empty store, saving preferences, retrieving by key, overwriting preferences.

```python
def test_load_empty_store(tmp_path):
    store = PreferenceStore(tmp_path / "prefs.json")
    prefs = store.load()
    assert prefs == {}

def test_save_and_load(tmp_path):
    store = PreferenceStore(tmp_path / "prefs.json")
    pref = Preference(key="style", value="snake_case", description="Naming style", tags=["naming"])
    store.set_preference("style", pref)
    loaded = store.get_preference("style")
    assert loaded.value == "snake_case"
```

- [ ] **Step 3: Implement PreferenceStore methods**

Implement load/save using JSON serialization. Ensure parent directory is created if it doesn't exist.

- [ ] **Step 4: Run tests**

Run: `pytest tests/test_preferences.py -v`
Expected: All tests pass.

- [ ] **Step 5: Commit**

```bash
git add src/donatello/config/ tests/test_preferences.py
git commit -m "feat: add preference storage system"
```

---

## Task 3: Project Detector

**Files:**
- Create: `src/donatello/detectors/project.py`
- Create: `src/donatello/detectors/docs.py`
- Create: `src/donatello/detectors/__init__.py`
- Create: `tests/test_detectors.py`

**Interfaces:**
- Produces:
  - `ProjectType` enum with values: ANDROID, WEB_REACT, WEB_NEXT, PYTHON, GO, RUST, UNKNOWN
  - `ProjectStage` enum with values: NEW, ONGOING
  - `detect_project_type(path: Path) -> ProjectType`
  - `detect_project_stage(path: Path) -> ProjectStage`
  - `find_documentation(path: Path) -> list[Path]` (returns paths to README, docs/, etc.)

**Steps:**

- [ ] **Step 1: Define project type and stage enums**

Create `src/donatello/detectors/project.py`.

```python
from enum import Enum

class ProjectType(Enum):
    ANDROID = "android"
    WEB_REACT = "web_react"
    WEB_NEXT = "web_next"
    PYTHON = "python"
    GO = "go"
    RUST = "rust"
    UNKNOWN = "unknown"

class ProjectStage(Enum):
    NEW = "new"
    ONGOING = "ongoing"
```

- [ ] **Step 2: Implement detect_project_type**

Check for marker files/directories: `build.gradle` or `AndroidManifest.xml` → ANDROID, `package.json` + `next.config.js` → WEB_NEXT, `package.json` → WEB_REACT, `pyproject.toml` or `setup.py` → PYTHON, `go.mod` → GO, `Cargo.toml` → RUST, else UNKNOWN.

- [ ] **Step 3: Implement detect_project_stage**

If project has non-trivial git history (>1 commit) or src files, mark as ONGOING; otherwise NEW.

- [ ] **Step 4: Implement find_documentation**

Recursively search for README.md, docs/, DESIGN.md, ADR/, etc. Return list of paths found.

- [ ] **Step 5: Write tests**

Test detection of each project type, NEW vs ONGOING, and doc finding.

- [ ] **Step 6: Commit**

```bash
git add src/donatello/detectors/ tests/test_detectors.py
git commit -m "feat: add project type and stage detection"
```

---

## Task 4: GitHub Integration

**Files:**
- Create: `src/donatello/integrations/github.py`
- Create: `src/donatello/integrations/__init__.py`
- Create: `tests/test_github.py`

**Interfaces:**
- Produces:
  - `GitHubClient` class with methods: `fetch_repo_info(url: str) -> dict`, `list_issues(url: str) -> list[dict]`, `get_repo_topics(url: str) -> list[str]`
  - Caches responses in `~/.donatello/cache/github.json`

**Steps:**

- [ ] **Step 1: Create GitHubClient class**

Parse GitHub URL to extract owner/repo. Use GitHub API (no auth for public repos). Implement rate-limit awareness.

```python
import requests

class GitHubClient:
    BASE_URL = "https://api.github.com"
    
    def fetch_repo_info(self, url: str) -> dict:
        # Parse URL, fetch repo info from API
        pass
    
    def list_issues(self, url: str, limit: int = 10) -> list[dict]:
        # Fetch open issues
        pass
    
    def get_repo_topics(self, url: str) -> list[str]:
        # Fetch repo topics
        pass
```

- [ ] **Step 2: Implement caching**

Cache responses for 1 hour in `~/.donatello/cache/github.json`.

- [ ] **Step 3: Write tests**

Mock GitHub API responses. Test URL parsing, issue fetching, topic extraction.

- [ ] **Step 4: Commit**

```bash
git add src/donatello/integrations/github.py tests/test_github.py
git commit -m "feat: add GitHub integration"
```

---

## Task 5: Documentation Analyzer

**Files:**
- Create: `src/donatello/detectors/docs_parser.py`
- Create: `tests/test_docs_parser.py`

**Interfaces:**
- Produces:
  - `parse_readme(path: Path) -> dict` with keys: `title`, `description`, `sections`, `setup_instructions`
  - `extract_metadata(paths: list[Path]) -> dict` with keys: `has_design_doc`, `has_architecture_doc`, `has_setup_guide`

**Steps:**

- [ ] **Step 1: Create documentation parser**

Parse README and other docs to extract structure, identify sections (Setup, Architecture, Contributing, etc.).

- [ ] **Step 2: Implement metadata extraction**

Determine if docs already cover design, architecture, and setup. This informs whether to run the interview.

- [ ] **Step 3: Write tests**

Test parsing various README formats.

- [ ] **Step 4: Commit**

```bash
git add src/donatello/detectors/docs_parser.py tests/test_docs_parser.py
git commit -m "feat: add documentation parsing"
```

---

## Task 6: Code Analyzer

**Files:**
- Create: `src/donatello/analyzer/codebase.py`
- Create: `tests/test_analyzer.py`

**Interfaces:**
- Produces:
  - `CodeAnalyzer` class with methods: `detect_patterns(path: Path) -> dict[str, list[str]]`
  - Returns patterns like: `{"dependency_injection": [...], "testing": [...], "naming": [...], "preview": [...]}`

**Steps:**

- [ ] **Step 1: Define pattern detectors**

Create methods to scan code and detect:
- Dependency injection: grep for constructor injection patterns, factory patterns
- Testing: test file structure, testing framework used
- Naming: analyze naming conventions in codebase
- Preview: detect preview/demo setup (Storybook, etc.)

- [ ] **Step 2: Implement CodeAnalyzer**

Traverse codebase, apply pattern detectors, aggregate results.

- [ ] **Step 3: Write tests**

Test analysis of sample codebases.

- [ ] **Step 4: Commit**

```bash
git add src/donatello/analyzer/codebase.py tests/test_analyzer.py
git commit -m "feat: add codebase pattern analyzer"
```

---

## Task 7: New Project Workflow

**Files:**
- Create: `src/donatello/workflows/new_project.py`
- Create: `src/donatello/interviews/questionnaire.py`
- Modify: `src/donatello/cli/main.py` (implement init command)
- Create: `tests/test_new_project_workflow.py`

**Interfaces:**
- Consumes: `ProjectStage`, `detect_project_stage()`, `find_documentation()`, `PreferenceStore`, `GitHubClient`
- Produces: Initialized project config saved to `.donatello/project.json`

**Steps:**

- [ ] **Step 1: Create interview questionnaire**

Design questions for new projects:
- "Is this a project from scratch or do you have existing design docs?"
- "Do you have a Figma design? (link: ...)"
- "Do you have a GitHub repo? (URL: ...)"
- "What's the primary tech stack?"
- Ask follow-up questions based on detected project type.

```python
QUESTIONS = [
    {
        "key": "from_scratch",
        "prompt": "Is this project from scratch? (yes/no)",
        "type": "boolean"
    },
    {
        "key": "figma_link",
        "prompt": "Do you have a Figma design? (link or skip)",
        "type": "string",
        "required": False
    },
    # ... more questions
]
```

- [ ] **Step 2: Implement interview flow**

Create function `run_interview(project_type: ProjectType) -> dict` that:
1. Displays intro message
2. Asks questions in order
3. Allows user to skip/abort
4. Returns answers dict

- [ ] **Step 3: Create new project workflow**

Function `initialize_new_project(path: Path) -> None`:
1. Detect project stage; if ONGOING, warn and exit
2. Find documentation
3. If docs exist, parse and skip interview
4. If no docs, run interview
5. Save answers to `.donatello/project.json`
6. Print summary and next steps

- [ ] **Step 4: Integrate into CLI**

Update `src/donatello/cli/main.py` `init` command to call `initialize_new_project()`.

- [ ] **Step 5: Write tests**

Mock user input, test interview flow, config save.

- [ ] **Step 6: Commit**

```bash
git add src/donatello/workflows/new_project.py src/donatello/interviews/ src/donatello/cli/main.py tests/test_new_project_workflow.py
git commit -m "feat: implement new project initialization workflow"
```

---

## Task 8: Onboarding Workflow

**Files:**
- Create: `src/donatello/workflows/onboard.py`
- Modify: `src/donatello/cli/main.py` (implement onboard command)
- Create: `tests/test_onboard_workflow.py`

**Interfaces:**
- Consumes: `detect_project_stage()`, `CodeAnalyzer`, `PreferenceStore`, `ProjectType`
- Produces: Report printed to stdout with alignment analysis and conflict warnings

**Steps:**

- [ ] **Step 1: Create alignment scorer**

Function `score_alignment(detected_patterns: dict, user_preferences: dict) -> dict` returns:
```python
{
    "score": 0.0-1.0,
    "matches": [...],
    "deviations": [...],
    "conflicts": [...]
}
```

Score is weighted: multiple pattern matches → higher confidence. Single match → lower confidence.

- [ ] **Step 2: Implement onboarding workflow**

Function `onboard_to_project(path: Path) -> None`:
1. Detect project stage; if NEW, warn user
2. Load user preferences from PreferenceStore
3. Analyze codebase patterns
4. Score alignment
5. Print report: "Your style score: X/10. Matches: [...]. Deviations: [...]. Likely conflicts: [...]"

- [ ] **Step 3: Create conflict detector**

Identify high-impact misalignments (e.g., preference for DI but codebase uses static factories → flag as conflict).

- [ ] **Step 4: Integrate into CLI**

Update `src/donatello/cli/main.py` `onboard` command.

- [ ] **Step 5: Write tests**

Test alignment scoring, conflict detection.

- [ ] **Step 6: Commit**

```bash
git add src/donatello/workflows/onboard.py src/donatello/cli/main.py tests/test_onboard_workflow.py
git commit -m "feat: implement developer onboarding workflow"
```

---

## Task 9: LLM Interview Engine (Claude Integration)

**Files:**
- Create: `src/donatello/integrations/claude.py`
- Modify: `src/donatello/interviews/questionnaire.py` (support LLM-guided interviews)
- Create: `tests/test_claude_integration.py`

**Interfaces:**
- Produces:
  - `ClaudeInterviewEngine` class with method `conduct_interview(questions: list[dict], context: dict) -> dict`
  - Uses Claude to ask follow-up questions, clarify answers, and guide conversation naturally

**Steps:**

- [ ] **Step 1: Set up Claude API or MCP client**

Create `ClaudeInterviewEngine` that communicates with Claude via API (or local MCP). Initialize with API key or connection details.

- [ ] **Step 2: Implement guided interview**

Use Claude to:
1. Present question to user
2. Listen to answer
3. Ask follow-ups based on answer (e.g., if user mentions "monorepo", ask about multiple languages)
4. Validate and clarify unclear answers

- [ ] **Step 3: Integrate into questionnaire**

Update `run_interview()` to use `ClaudeInterviewEngine` when available (fall back to simple CLI prompts if not).

- [ ] **Step 4: Write tests**

Mock Claude responses, test interview flow.

- [ ] **Step 5: Commit**

```bash
git add src/donatello/integrations/claude.py tests/test_claude_integration.py
git commit -m "feat: add Claude-powered interview engine"
```

---

## Task 10: Figma Integration

**Files:**
- Create: `src/donatello/integrations/figma.py`
- Create: `tests/test_figma_integration.py`

**Interfaces:**
- Produces:
  - `parse_figma_link(link: str) -> dict` with keys: `file_id`, `node_id`, `is_valid`
  - Validates Figma URL formats (design, make, slides, branch links)

**Steps:**

- [ ] **Step 1: Parse Figma URLs**

Handle formats:
- `figma.com/design/:fileKey/:fileName?node-id=:nodeId`
- `figma.com/design/:fileKey/branch/:branchKey/:fileName`
- `figma.com/make/:makeFileKey/:makeFileName`
- Short links: `figma.app.com/...`

```python
def parse_figma_link(link: str) -> dict:
    # Parse and validate; return {"file_id": "...", "node_id": "...", "is_valid": True/False}
    pass
```

- [ ] **Step 2: Validate links**

Test URLs; return is_valid=False for malformed links.

- [ ] **Step 3: Write tests**

Test various URL formats, short links, invalid URLs.

- [ ] **Step 4: Commit**

```bash
git add src/donatello/integrations/figma.py tests/test_figma_integration.py
git commit -m "feat: add Figma link parsing"
```

---

## Task 11: Integration & Polish

**Files:**
- Modify: `README.md`
- Create: `docs/USAGE.md`
- Create: `docs/ARCHITECTURE.md`

**Interfaces:**
- No new code interfaces; polish and document existing ones.

**Steps:**

- [ ] **Step 1: Write usage documentation**

Document `donatello init` and `donatello onboard` commands, examples, and configuration.

- [ ] **Step 2: Write architecture documentation**

Explain workflows, preference system, GitHub/Figma integration.

- [ ] **Step 3: Update README**

Add quick-start, feature overview, and links to docs.

- [ ] **Step 4: Test end-to-end**

Run `donatello init` on a sample project, verify interview flow and config creation. Run `donatello onboard` on another sample project, verify analysis and report.

- [ ] **Step 5: Final cleanup**

Ensure all code is formatted, tests pass, no warnings.

- [ ] **Step 6: Commit**

```bash
git add README.md docs/ 
git commit -m "docs: add usage and architecture documentation"
```

---

## Self-Review

**Spec Coverage:**
- ✅ New project workflow with docs-first approach
- ✅ Interview flow for design/GitHub
- ✅ Ongoing project onboarding
- ✅ Preference system for coding style
- ✅ GitHub/Figma integration
- ✅ Code analyzer for pattern matching
- ✅ CLI entry points

**Step Scan:**
- All steps are atomic and testable.
- Signatures and test names are unambiguous.

**Type Consistency:**
- `ProjectType` and `ProjectStage` enums consistent across tasks.
- `PreferenceStore` interface stable.
- `CodeAnalyzer.detect_patterns()` returns consistent dict structure.

**Review Focus Coverage:**
- ✅ Task 7: user abort during interview → save/resume
- ✅ Task 3: project type ambiguity → ask/warn
- ✅ Task 8: preference false positives → weighted scoring
- ✅ Task 4: GitHub rate limits → caching + warning
- ✅ Task 10: Figma link edge cases → multiple format support

**Proportion:**
- 11 tasks, ~60 steps total. Plan is detailed without being a transcript.
- Each task has clear boundaries and deliverables.

---

## Summary

This plan breaks Donatello into 11 focused tasks:
1. CLI & project setup
2. Preference storage
3. Project detection
4. GitHub integration
5. Doc parsing
6. Code analysis
7. New project workflow
8. Onboarding workflow
9. LLM interview engine
10. Figma integration
11. Docs & polish

Tasks are ordered by dependency: foundation first (1–2), then detectors (3–6), then workflows (7–8), then enhancements (9–10), finally docs (11). Each task is independently testable and adds concrete value.
