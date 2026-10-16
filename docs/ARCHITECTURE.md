# Donatello Architecture

## Overview

Donatello is a Claude Code plugin with no runtime code. It has three parts:

1. **Workflow skills**: instructions Claude follows (`onboard`, `phase-new`, `phase-archive`)
2. **Templates**: the shape of the files those skills write
3. **Data**: your preferences and domain skills

Claude is the engine: it reads the target project, runs the interview, and fills the templates. The skills set the steps, the guardrails, and the output format.

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
  phase-archive/SKILL.md
preferences/*.md
templates/claude-block.md
```

Skills are namespaced by the plugin: `skills/onboard/` is invoked as `/donatello:onboard`.

### Referencing files across the plugin

Claude Code gives a skill its base directory, so skills reference supporting files with relative paths:
- `phase-new` → `templates/overview.md`
- `onboard` → `../../preferences/`, `../../templates/claude-block.md`, and `../phase-new/` for phase 1

Templates live in `phase-new` because that is their only writer; `onboard` delegates to it, so there is one source of truth for phase files.

---

## Components

### `onboard` skill
- **Frontmatter:** `disable-model-invocation: true`, `argument-hint: "[--defaults]"`
- **Reads:** target repo (README, manifests, CLAUDE.md, docs/, .spec/, `git log`), `preferences/`
- **Writes:** `docs/preferences.md`, `docs/architecture.md`, `docs/requirements.md` (existing projects, only when behavior is clear from the code), phase 1 (via phase-new steps), the CLAUDE.md block
- **Guardrails:** propose before writing; never overwrite; resume if `.spec/` already exists; ask rather than guess when the project type is ambiguous (monorepos)

### `phase-new` skill
- **Frontmatter:** `disable-model-invocation: true`, `argument-hint: "[scope]"`
- **Reads:** `docs/`, `.spec/ongoing/`, `.spec/archive/` (to find the next N and prior context)
- **Writes:** `.spec/ongoing/phase<N>-<slug>/` from the four templates
- **Task shaping:** follows `docs/preferences.md` (test-first ordering, preview tasks, etc.); a task names a skill when one applies

### `phase-archive` skill
- **Frontmatter:** `disable-model-invocation: true`, `argument-hint: "[phase]"`
- **Reads:** the phase's `tasks.md`, `design.md`, `spec.md`, `walkthrough.md`
- **Writes:** final "Current state" summary in `walkthrough.md`; promoted decisions in `docs/architecture.md`; promoted requirements (adjusted for deviations) in `docs/requirements.md`; then moves the directory to `.spec/archive/<date>-phase<N>-<slug>/`

### Walkthrough upkeep (no skill)
- Driven by the CLAUDE.md block, not a command, so it happens during normal work
- On the first finished task the agent creates `walkthrough.md`. The CLAUDE.md block can't reference plugin paths, so it names the four sections inline; `skills/phase-new/templates/walkthrough.md` is the fuller version that `phase-archive` uses when it builds a missing walkthrough.

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
## Project workflow (Donatello)
- Preferences: @docs/preferences.md
- Orientation: docs/architecture.md, docs/requirements.md (read when needed)
- Current work: .spec/ongoing/. Read the active phase's overview, spec, tasks, and the "Current state" section of walkthrough.md
- After each task: tick it in tasks.md and update the phase's walkthrough.md (create if missing, with sections: Current state, How to verify, Changes by task, Deviations from spec)
- End of session: rewrite walkthrough.md "Current state"
- New phase: /donatello:phase-new · Finish a phase: /donatello:phase-archive
<!-- donatello:end -->
```

### Preferences
- One file per category with frontmatter:
  ```markdown
  ---
  category: testing
  summary: Test-first; unit tests next to code; fakes over mocks
  applies-to: [android, web, python]   # optional; omit = all
  ---
  ```
- `onboard` picks files whose `applies-to` matches the project and joins them into `docs/preferences.md`, in sections, so the user can edit per project

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

### Marked block in CLAUDE.md
- **Why:** many projects already have a CLAUDE.md; the markers let Donatello update its part idempotently.

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
| Monorepo / ambiguous stack | Ask which part(s) to onboard |
| Archiving with unchecked tasks | List them; archive anyway, carry over, or stop |
| Archiving with no `walkthrough.md` | Build one from `tasks.md` and git history since the phase started; ask about "How to verify" |
| No `docs/requirements.md` yet | Create it on this archive |
| Not a git repo | Use plain moves; mention that history won't be tracked |
