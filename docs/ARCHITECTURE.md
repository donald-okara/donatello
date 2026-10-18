# Donatello Architecture

## Overview

Donatello is a Claude Code plugin with no runtime code. It has four parts:

1. **Workflow skills**: instructions Claude follows (`onboard`, `phase-new`, `phase-run`, `phase-archive`, `skills`)
2. **Agents**: subagents with fixed models for legwork and review (`implementer`, `reviewer`, `scout`)
3. **Templates**: the shape of the files the skills write
4. **Data**: your preferences, the skill catalog (`catalog/skills.yaml`), and your own skill folders (`catalog/skills/`)

Claude is the engine: it reads the target project, runs the interview, fills the templates, and writes the code. The skills set the steps, guardrails, and output format; the agents decide which model does which work.

---

## Plugin Layout

```
.claude-plugin/
  plugin.json           # name: "donatello", version, author
  marketplace.json      # one plugin, source: "./"
skills/
  onboard/SKILL.md
  phase-new/SKILL.md
  phase-new/templates/{overview,design,spec,tasks,walkthrough}.md
  phase-run/SKILL.md
  phase-archive/SKILL.md
  skills/SKILL.md
agents/
  implementer.md
  reviewer.md
  scout.md
preferences/*.md
catalog/
  skills.yaml           # the skill catalog: one entry per offerable skill
  skills/<id>/SKILL.md  # your own skills (only folders with a donatello source)
templates/claude-block.md
```

Skills and agents are namespaced by the plugin: `skills/onboard/` is invoked as `/donatello:onboard`, `skills/skills/` as `/donatello:skills`, and `agents/implementer.md` is the `donatello:implementer` agent.

### Referencing files across the plugin

Claude Code gives a skill its base directory, so skills reference supporting files with relative paths:
- `phase-new` → `templates/overview.md`
- `onboard` → `../../preferences/`, `../../templates/claude-block.md`, `../phase-new/` for phase 1, and `../skills/SKILL.md` for the import flow
- `skills` → `../../catalog/skills.yaml` (the installed copy, when no clone is found) and `../../catalog/skills/<id>/`

Templates live in `phase-new` because that is their only writer; `onboard` delegates to it, so there is one source of truth for phase files. The same holds for the import flow: it is defined once in `skills/skills/SKILL.md`, and onboard follows it.

---

## Components

### `onboard` skill
- **Frontmatter:** `disable-model-invocation: true`, `argument-hint: "[--defaults]"`
- **Reads:** the `donatello:scout` report on the target repo (not the repo directly), `preferences/`, the skill catalog (via the import's Candidates step), the list of enabled plugins
- **Writes:** `docs/preferences.md` (general preferences), `.claude/rules/<category>.md` (preferences with `paths`), `docs/architecture.md`, `docs/requirements.md` (existing projects, only when behavior is clear from the code), phase 1 (via phase-new steps), the CLAUDE.md block, `.claude/skills/<id>/` and the plugin entries for chosen catalog skills (skills step, 6g), `.gitignore` lines for "just me" imports, and `.claude/settings.local.json` with the plugins you chose to disable
- **Guardrails:** propose before writing; never overwrite; never duplicate (adopt existing docs for the architecture, requirements, and preferences roles, and drop preferences the project already states); resume if `.spec/` already exists; ask rather than guess when the project type is ambiguous (monorepos); skills step runs before plugin pruning so pruning never proposes an imported plugin; `--defaults` skips skill import

### `skills` skill
- **Frontmatter:** `disable-model-invocation: true`, `argument-hint: "[add <ref> | remove <id> | list | import]"`
- **Reads:** `catalog/skills.yaml` from the clone when one is found (cwd or `~/.claude/donatello.json`), else the installed copy; `list` and `import` never ask for a clone; the Android CLI's `skills list` and the marketplace files under `~/.claude/plugins/` (to fill `add` entries); `.claude/skills/`, `.claude/settings*.json` (to find what is already present)
- **Writes:** on approval only: a catalog entry (and `catalog/skills/<id>/` for your own skills) in the clone; for `import`, `.claude/skills/<id>/`, plugin entries in `.claude/settings.json` or `.claude/settings.local.json`, and `.gitignore` lines for "just me"
- **Clone path:** `~/.claude/donatello.json` (`{"clone": "<path>"}`), asked once for add and remove only
- **Import shape:** four separable parts (Candidates, Ask, Install, Summary) so onboard can run Candidates and Ask as its own questions and Install and Summary later
- **Guardrails:** refuses duplicate ids; pins git refs to a commit SHA; fetches git folders with `GIT_TERMINAL_PROMPT=0` into a `mktemp` directory that is always removed; never replaces an existing `.claude/skills/<id>/` folder or settings value; never edits the installed plugin copy; never commits or pushes without a yes

### `phase-new` skill
- **Frontmatter:** `disable-model-invocation: true`, `argument-hint: "[scope]"`
- **Reads:** `docs/`, all `.claude/rules/*.md` (path-scoped rules don't load during planning, since no matching file is touched), `.spec/ongoing/`, `.spec/archive/` (to find the next N and prior context)
- **Writes:** `.spec/ongoing/phase<N>-<slug>/` from the four templates
- **Task shaping:** follows `docs/preferences.md` (test-first ordering, preview tasks, etc.); a task names a skill when one applies

### `phase-run` skill
- **Frontmatter:** `disable-model-invocation: true`, `argument-hint: "[task | --all]"`; no `model` (it runs on the session model, Opus)
- **Reads:** the phase's `tasks.md` and the spec requirements the task references; agent reports. Never full diffs or build logs.
- **Loop per task:** implementer → reviewer → (one fix round) → escalate the implementer to Sonnet via the Agent tool's per-call `model` → stop and ask
- **Writes:** commit per task, tick in `tasks.md`, walkthrough "Changes by task" and "Current state"
- **Handoff to agents:** passes paths, not content (phase directory, `docs/preferences.md`, requirement IDs), so each agent reads only what it needs

### `phase-archive` skill
- **Frontmatter:** `disable-model-invocation: true`, `argument-hint: "[phase]"`
- **Reads:** the phase's `tasks.md`, `design.md`, `spec.md`, `walkthrough.md`
- **Writes:** final "Current state" summary in `walkthrough.md`; promoted decisions in `docs/architecture.md`; promoted requirements (adjusted for deviations) in `docs/requirements.md`; then moves the directory to `.spec/archive/<date>-phase<N>-<slug>/`

### Walkthrough upkeep (no skill)
- Driven by the CLAUDE.md block, not a command, so it happens during normal work
- On the first finished task the agent creates `walkthrough.md`. The CLAUDE.md block can't reference plugin paths, so it names the four sections inline; `skills/phase-new/templates/walkthrough.md` is the fuller version that `phase-archive` uses when it builds a missing walkthrough.

### Agents

| Agent | Model | Tools | maxTurns | Returns |
|---|---|---|---|---|
| `implementer` | `haiku` | All except Agent | 40 | ≤ 15 lines: files changed, build/test result, blockers |
| `reviewer` | `sonnet`, `effort: medium` | Read, Grep, Glob, Bash (git, tests) | 20 | `PASS`, or ≤ 10 issues as `file:line: problem` |
| `scout` | `haiku` | Read, Grep, Glob, Bash (read-only commands) | 25 | ≤ 60 lines: stack, modules, build/test commands, observed conventions, existing docs |

Shared rules in each agent's prompt:
- Follow `docs/preferences.md` and any `.claude/rules/` that load
- Run builds and tests in quiet mode (e.g. `./gradlew -q`, `--console=plain`) and report only failures, never full logs
- Implementer: do only the given task; don't tick tasks, edit `.spec/` files, or commit (phase-run does all three after review)
- Reviewer: judge against the task, spec requirements, and preferences; no style nitpicks beyond the preferences

### Templates
| File | Content |
|---|---|
| `overview.md` | Goal, what the phase delivers, out of scope, success criteria (intent only, not updated after planning) |
| `design.md` | Decisions as *Decision / Why / Tradeoff*, and nothing else |
| `spec.md` | Numbered requirements (R1…), each with scope and a verifiable acceptance check |
| `tasks.md` | Plain `- [ ]` checklist grouped by task; no status emoji, since the checkbox *is* the status |
| `walkthrough.md` | Current state (≤ 10 lines) / How to verify / Changes by task / Deviations from spec |
| `claude-block.md` | The marked CLAUDE.md block (see below) |

### CLAUDE.md block (shape)
```markdown
<!-- donatello:start -->
## Spec workflow (Donatello plugin)
- Preferences: @docs/preferences.md (file-specific rules live in .claude/rules/)
- Models: plan and discuss in the main session; implement via donatello:implementer, review via donatello:reviewer, survey via donatello:scout
- Orientation: architecture in docs/architecture.md, requirements in docs/requirements.md (read when needed)
- Current work: .spec/ongoing/. Read the active phase's overview, spec, tasks, and the "Current state" section of walkthrough.md
- After each task: tick it in tasks.md and update the phase's walkthrough.md (create if missing, with sections: Current state, How to verify, Changes by task, Deviations from spec)
- End of session: rewrite walkthrough.md "Current state"
- Commands: /donatello:phase-new · /donatello:phase-run · /donatello:phase-archive
<!-- donatello:end -->
```

### Preferences
- One file per category with frontmatter:
  ```markdown
  ---
  category: testing
  summary: Test-first; unit tests next to code; fakes over mocks
  applies-to: [android, web, python]   # optional; omit = all
  paths: ["**/*.kt"]                   # optional; present = path-scoped rule
  ---
  ```
- `onboard` picks files whose `applies-to` matches the project:
  - no `paths` → joined into `docs/preferences.md` in sections (always loaded; keep the total ≤ 100 lines)
  - with `paths` → written to `.claude/rules/<category>.md` with the `paths` frontmatter, so it loads only when Claude touches matching files
- `model-usage.md` ships by default: Opus for brainstorming/planning in the main session, Sonnet for review and fallback implementation, Haiku for implementation and legwork, plus the escalation rule. It's the human-readable version of what the agents' frontmatter enforces.

---

## Design Decisions

### Plugin instead of CLI
- **Why:** interviewing, phase breakdown, and task suggestion are reasoning tasks. A rule-based CLI would imitate them badly; Claude already does them well.
- **Tradeoff:** output varies more between runs than a templating CLI's would. Mitigated by strict templates and guardrails in each skill.

### Preferences are copied, not linked
- **Why:** projects stay self-contained and can diverge; no lookup at runtime; works for collaborators who don't have Donatello installed.
- **Tradeoff:** drift. Re-running onboard offers a diff against the current preferences.

### Walkthrough as a separate record
- **Why:** intent (overview, spec), reasoning (design), and what actually happened are different kinds of content; mixing them made `design.md` and `overview.md` change after planning. A walkthrough also gives reviewers a "how to verify" path. This follows Antigravity's artifacts (task list / implementation plan / walkthrough).
- **Tradeoff:** one more file. Mitigated by creating it lazily and reading only "Current state" on resume.

### Living requirements in `docs/requirements.md`
- **Why:** without it, current behavior can only be pieced together from archived specs. This borrows OpenSpec's always-current `specs/` without its ADDED/MODIFIED/REMOVED delta format; the agent merges in prose at archive time.
- **Tradeoff:** merges are judgment calls, so they're shown as a diff for approval.

### Model routing through agents, not skill frontmatter
- **Why:** a skill's `model:` lasts only for the turn it's invoked in, so a multi-turn interview or task loop would drift back to the session model. An agent's `model:` holds for its whole run, and verbose work stays in the agent's context.
- **Tradeoff:** Haiku can miss on underspecified tasks. The reviewer step and the escalation to Sonnet are always on to cover this; the precise tasks that phase-new writes help too.

### Path-scoped rules for stack preferences
- **Why:** `@imports` load at launch, so everything in `docs/preferences.md` costs tokens every session. `.claude/rules/` files with `paths` load only when matching files are touched.
- **Tradeoff:** preferences are split across two places; onboard writes both, and the CLAUDE.md block points to the rules directory.

### Disabling irrelevant plugins per project
- **Why:** every enabled plugin's skill descriptions are listed on every turn. On a machine with many plugins, most are noise in any one project.
- **Where:** `.claude/settings.local.json` (personal, not committed), so teammates' setups aren't affected.

### Marked block in CLAUDE.md
- **Why:** many projects already have a CLAUDE.md; the markers let Donatello update its part idempotently.

### Adopt existing docs; the block holds the paths
- **Why:** a fact written in two files drifts, and duplicated rules in always-loaded files cost tokens every session.
- **How:** onboard maps each role (architecture, requirements, preferences) to an existing file when one covers it and writes the paths into the block. Skills and agents read them from there, with `docs/*.md` as defaults.

### Catalog stores references, not copies
- **Why:** Android CLI skills and plugins have their own install and update channels; copying them into Donatello would fork them and go stale.
- **Tradeoff:** importing needs the network and the relevant tools (`android`, `git`). Failures are skipped and listed with the command to run later.

### Own skills live outside `skills/`
- **Why:** everything in the plugin's `skills/` directory loads in every project. Keeping domain skills in `catalog/skills/` means a project pays only for the skills it imported, and matches how preferences are copied rather than linked.
- **Tradeoff:** copies drift from the catalog version. Accepted for now; a later `import --update` could refresh them.

### Edits go to your clone, not the installed plugin
- **Why:** the installed plugin is a cached copy that a marketplace update overwrites. The clone is the source of truth and syncs across machines through git.
- **Tradeoff:** one-time setup of the clone path. Other machines see new entries only after a push and a `/plugin marketplace update`; reading from the clone when one is configured hides that lag on the machine where you added them.

### Scope is asked per import
- **Why:** team projects want shared skills, and personal experiments in shared repos shouldn't leak into teammates' setups.
- **Where:** "committed" writes `.claude/skills/` and `.claude/settings.json` (meant to be committed); "just me" writes the same folders and `.claude/settings.local.json`, then lists them in `.gitignore`.
- **Tradeoff:** one more question per onboard.

### Git skills are pinned to a commit
- **Why:** imports are reproducible, and an upstream change can't silently alter a project's instructions.
- **Tradeoff:** no automatic updates. To move the pin, `remove` the entry and `add` the URL again, since `add` refuses an existing id.

### Plugins install whole
- **Why:** Claude Code enables plugins, not individual skills inside them.
- **Tradeoff:** a plugin can bring more skills than wanted; plugin pruning and the import summary make that visible.

### The import flow lives in the `skills` skill
- **Why:** one definition of the flow for onboard and for later `/donatello:skills import` runs; keeps onboard's always-read text short.
- **Tradeoff:** onboard depends on a sibling skill's path (`../skills/`), the same pattern as `../phase-new/`.

### Repo is both plugin and marketplace
- **Why:** one repo and one install command, with no separate marketplace repo to maintain.
- **Tradeoff:** splitting into several plugins later means restructuring into subdirectories.

### Phase templates live in `phase-new`
- **Why:** one writer and one source of truth for phase files. Onboard reuses phase-new's steps for phase 1.
- **Tradeoff:** onboard depends on a sibling skill's path (`../phase-new/`).

### One plugin to start
- **Why:** simplest to install and iterate on.
- **Later:** split domain skills into per-stack plugins (`donatello-android`, `donatello-web`) in the same marketplace so onboard can enable only the relevant ones via the project's `.claude/settings.json`.

---

## Edge Cases

| Case | Behavior |
|---|---|
| CLAUDE.md exists, no block | Append the block; leave other content alone |
| CLAUDE.md has a block | Replace only between the markers, after showing a diff |
| `.spec/` exists | Resume: report what's there, offer to add missing pieces |
| `docs/preferences.md` exists | Show a diff against current preferences; merge only with approval |
| Existing doc covers a role under another name (`docs/DESIGN.md`, ADRs, `CONTRIBUTING.md`) | Adopt it: no new file; the block names its path; additions go in as a diff |
| Architecture or requirements only described inside CLAUDE.md / AGENTS.md | Create the default file anyway (those files load every session), without repeating what they say |
| CLAUDE.md / rules already state a preference | Drop that rule; on conflict the project wins unless the user switches |
| File name differs only in case (`ARCHITECTURE.md`) | Treated as the same file |
| Monorepo / ambiguous stack | Ask which part(s) to onboard |
| Archiving with unchecked tasks | List them; archive anyway, carry over, or stop |
| Archiving with no `walkthrough.md` | Build one from `tasks.md` and git history since the phase started; ask about "How to verify" |
| No `docs/requirements.md` yet | Create it on this archive |
| Not a git repo | Use plain moves; mention that history won't be tracked. `phase-run` skips commits, and the reviewer gets a list of changed files instead of `git diff` |
| Implementer blocked or review fails twice | Re-dispatch the implementer on Sonnet; if it still fails, stop and present the issues in the main session |
| `docs/preferences.md` over 100 lines after merge | Onboard points out the longest sections and proposes moving file-specific ones into `.claude/rules/` |
| Project already has `.claude/settings.local.json` | Merge the plugin entries only; leave other keys alone |
| Project already has `.claude/settings.json` (committed import) | Merge the plugin entries only; leave other keys alone |
| Settings file is not valid JSON | Don't touch it; fail that import entry and print the entries to add by hand |
| `android` CLI missing or failing | Skip that entry; list it under Failed with `android skills add <id> --agent=claude-code --project=.` |
| Plugin marketplace unknown to this machine | `add` stops with "not found" if it's not in `known_marketplaces.json`; `import` fails the entry with `/plugin marketplace add <owner/repo>` then `/plugin install` |
| Plugin missing from the marketplace's `marketplace.json` | `add` stops with "not found"; nothing is written |
| Android skill id not listed by `android skills list` | `add` stops with "not found" |
| Git ref whose branch name contains `/` | Resolve by trying the longest prefix with `git ls-remote`; if none or several match, ask which is meant |
| Git ref that is a tag, or a URL with no `tree/` segment | Tags aren't resolved (only `refs/heads/`) and fall back to asking; no `tree/` means `HEAD` |
| Git repo private, missing, or offline | `GIT_TERMINAL_PROMPT=0` makes it fail instead of prompting; entry fails with the manual fetch command |
| Fetched folder has no `SKILL.md` | Fail the entry; nothing is copied; scratch directory removed |
| Copy fails partway | Remove the partial `.claude/skills/<id>/` and fail the entry |
| `add` with an id that already exists in the catalog | Refuse |
| `add` with a local folder that has no `SKILL.md` | Stop; nothing is copied into `catalog/skills/` |
| Skill folder already in `.claude/skills/<id>/` | Skip it at Install, list as already present, and don't add it to `.gitignore` (even if it appeared after Candidates) |
| No clone configured, `add` or `remove` | Ask once for the clone path and save it to `~/.claude/donatello.json`; decline stops the command |
| No clone found, `list` or `import` | Read the installed copy; never prompt. A clone found in the cwd or `~/.claude/donatello.json` is read instead |
| `import` in a project that isn't a git repo | The `git check-ignore` step is skipped (exit 128); `.gitignore` lines are still written for "just me" |
| Onboard with `--defaults` | Skill import skipped; the wrap-up points to `/donatello:skills import` |
