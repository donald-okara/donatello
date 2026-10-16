# Phase 1: Design Notes

## Decisions

### Plugin instead of CLI
**Decision:** Build Donatello as a Claude Code plugin with no runtime code.
**Why:** The core features (interview, phase breakdown, task suggestion) are reasoning tasks Claude already handles; a rule-based CLI would imitate them badly. It also makes skills installable and discoverable, which relative markdown links in CLAUDE.md would not.
**Tradeoff:** Output varies more between runs than a templating CLI's would. Mitigated with strict templates, explicit steps, and propose-before-write.

### Repo is both plugin and marketplace
**Decision:** `marketplace.json` with `source: "./"`.
**Why:** One repo, one install command, no separate marketplace repo to maintain.
**Tradeoff:** Splitting into several plugins later means restructuring into subdirectories. Acceptable.

### Templates live in `phase-new`
**Decision:** Phase templates sit under `skills/phase-new/templates/`; `onboard` reuses phase-new's steps for phase 1.
**Why:** One writer, one source of truth for phase files.
**Tradeoff:** `onboard` depends on a sibling skill's path (`../phase-new/`); verified in Task 1.

### Workflow skills are explicit-only
**Decision:** `disable-model-invocation: true` on onboard, phase-new, phase-archive.
**Why:** They write many files; they should run only when asked.

### Preferences: copied, filtered by `applies-to`
**Decision:** Copy matching preference files into `docs/preferences.md` and import it from the CLAUDE.md block.
**Why:** Projects stay self-contained; collaborators without Donatello still get the context.
**Tradeoff:** Drift; re-onboard shows a diff.

### Write real preferences before the skills
**Decision:** Task 2 (preferences) comes before the skills.
**Why:** The skills' main job is to carry your preferences into projects; without real content there's nothing to judge the output against.

### Walkthrough replaces Outcome and handoff notes
**Decision:** Add `walkthrough.md` (Current state / How to verify / Changes by task / Deviations from spec), created after the first finished task.
**Why:** It keeps intent, reasoning, and the record of what happened separate, gives reviewers a verification path, and gives the agent a cheap resume point. Modeled on Antigravity's walkthrough artifact.
**Tradeoff:** One more file; read only "Current state" on resume to keep cost down.

### Living requirements in `docs/requirements.md`
**Decision:** On archive, merge lasting requirements into `docs/requirements.md` as prose, with a diff for approval.
**Why:** Current behavior shouldn't require reading the archive. Borrowed from OpenSpec's `specs/`, without its delta syntax.
**Tradeoff:** Merges are judgment calls; the approval diff catches bad ones.

## Verification Approach

There is no code to unit-test. Each skill is verified by running it:
- Load locally with `claude --plugin-dir /Users/donaldisoe/AI-Mployees/donatello` in throwaway fixture projects (empty dir; small existing repo with its own CLAUDE.md)
- Then on one real Android and one real web project
- Record issues found in this phase's `walkthrough.md` and fix them in the skill text

## Open Questions

- Does a skill's relative path to `../../preferences/` resolve in the installed plugin cache as it does with `--plugin-dir`? Verify in Task 1; if not, move preferences under `skills/onboard/`.

## Later Phases (one-liners)

- **Phase 2:** First domain skills (Android/Compose), written from real use
- **Phase 3:** Split into per-stack plugins; onboard enables the relevant ones in the project's `.claude/settings.json`
- **Phase 4:** Skill evals with `claude plugin eval` for onboard/phase-new output shape
