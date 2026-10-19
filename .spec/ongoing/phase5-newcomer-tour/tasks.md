# Phase 5: Tasks

Each task ends with a commit. As in phases 3 and 4, checks in Tasks 1–5 are reviewer walk-throughs of the skill text; live runs are in Task 6, run interactively. Independent of phase 4's remaining Task 5.

```
1. tour: load and stops
   └─> 2. tour: area focus, deeper, read-only
          └─> 3. onboard: newcomer question, onboarded branch
                 └─> 4. onboard: newcomer mode
                        └─> 5. Integration: block, scout
                               ├─> 7. Docs and README
                               └─> 6. Fixture verification (needs Donald)
```

---

### Task 1: tour — load and stops
**Serves:** R1, R2
**Files:** `skills/tour/SKILL.md`

- [x] Frontmatter: `name: tour`, a one-line `description` that names the triggers ("new to this repo", "show me around"), `argument-hint: "[area]"`, no `disable-model-invocation`; a short intro stating the tour is read-only and chat only
- [x] Section "1. Load": onboarded detection via the Donatello block in `CLAUDE.md`/`AGENTS.md`; the exact files read when onboarded (per R1); scout dispatch when not; no source reads
- [x] Section "2. Stops": the six stops in order with their content per R2, each at most ~10 lines, each ending "questions, deeper, or next?"; stop 5 skipped when not onboarded; stop 6's next step for each case
- [x] One rule line: facts come from the docs, the scout report, or files read for "deeper"; otherwise say "unknown"
- [x] Check: the reviewer walks an onboarded and a non-onboarded repo through "Load" and "Stops" against R1 and R2
- [x] Commit: `feat: add /donatello:tour`

### Task 2: tour — area focus, deeper, read-only
**Serves:** R3, NF2
**Files:** `skills/tour/SKILL.md`

- [x] Area handling: with `$ARGUMENTS`, limit every stop to that area; if the loaded docs don't cover it, dispatch the scout with a brief naming the focus path
- [x] Navigation: "next", "skip to <stop>", "deeper" (may read specific files to answer; the only code reads)
- [x] Guardrail: never write, edit, or commit; end of tour states nothing was changed
- [x] Check: the reviewer walks `/donatello:tour auth` where docs cover auth, and where they don't (scout dispatched with focus), against R3
- [x] Commit: `feat: add area focus to /donatello:tour`

### Task 3: onboard — newcomer question, onboarded branch
**Serves:** R4
**Files:** `skills/onboard/SKILL.md`

- [ ] Step 1: after the existing checks, unless `--defaults`, ask "Are you new to this repo?" with the git-authorship default from R4 (yes when no git, 0 commits, or under 5% of all commits by `git config user.email`)
- [ ] "Already onboarded" bullet: if new, say the repo is set up and offer `/donatello:tour` (recommended) or resume mode; before resume mode, warn that preferences refresh and block update change shared, committed files
- [ ] State that "not new" and `--defaults` keep today's behavior
- [ ] Check: the reviewer walks onboarded + new, onboarded + not new, and `--defaults` against R4
- [ ] Commit: `feat: ask newcomers in /donatello:onboard`

### Task 4: onboard — newcomer mode
**Serves:** R5, NF2
**Files:** `skills/onboard/SKILL.md`

- [ ] Step 4: in newcomer mode, replace "what's next" with "Do you know the next piece of work? 'No' is fine"; present scout open questions as findings to confirm, correct, or mark "don't know"; conflicts keep the project's rule without asking
- [ ] Step 5: list kept conflicts; offer "docs only" or "docs + phase 1"
- [ ] Step 6c: "don't know" answers go to `## Open questions` (at most 5 lines) at the end of the architecture doc, or as a diff to an adopted one; step 6e skipped for "docs only"
- [ ] Step 7: next step `/donatello:phase-run` or `/donatello:phase-new`; offer `/donatello:tour`; commit offer notes shared files and suggests a PR
- [ ] Check: the reviewer walks a non-onboarded repo with "new", one "don't know", and "docs only" against R5
- [ ] Commit: `feat: add newcomer mode to /donatello:onboard`

### Task 5: Integration
**Serves:** R6
**Files:** `templates/claude-block.md`, `agents/scout.md`

- [ ] `templates/claude-block.md`: add `/donatello:tour` to Commands
- [ ] `agents/scout.md`: description names `/donatello:tour` as a user; one line saying a brief may name a focus path, which narrows what is read but not the report format
- [ ] Check: the block measures under 300 tokens (`sed -n '/donatello:start/,/donatello:end/p' templates/claude-block.md | wc -c` / 4); `claude plugin validate .` passes
- [ ] Commit: `feat: connect tour to the block and scout`

### Task 6: Fixture verification
**Serves:** R1–R6

- [ ] Onboarded fixture: full `/donatello:tour` shows six stops, reads no source files, `git status` unchanged
- [ ] Non-onboarded fixture: tour runs from the scout, skips stop 5, ends at `/donatello:onboard`
- [ ] `/donatello:tour <area>` stays within the area; an area the docs don't cover dispatches the scout with focus
- [ ] Onboard, onboarded fixture, "new": offers the tour; "not new": resume mode as before
- [ ] Onboard, non-onboarded fixture, "new" + one "don't know" + "docs only": Open questions written, no phase 1, wrap-up names phase-new and the tour
- [ ] Onboard with `--defaults`: no newcomer question, output as before
- [ ] Log anything unexpected in this phase's `walkthrough.md` and fix the skill text
- [ ] Commit: `fix: adjust tour and onboard from fixture trial` (only if fixes were needed)

### Task 7: Docs and README
**Serves:** R7
**Files:** `docs/SPECIFICATION.md`, `docs/ARCHITECTURE.md`, `README.md`

- [ ] SPECIFICATION: workflow 8 `/donatello:tour [area]` after workflow 7, without renumbering others; workflow 1 notes the newcomer question; `skills/tour/` in the directory tree
- [ ] ARCHITECTURE: a `tour` component (frontmatter, reads, writes nothing, guardrails); onboard entry updated with the newcomer branches; this phase's lasting decisions; the edge cases from R7; block copy synced with `templates/claude-block.md`
- [ ] README: the command in Commands, a short "New to a repo?" section, `skills/tour/` in the layout, block copy synced
- [ ] Check: `grep -n "tour" README.md docs/*.md` covers the command, read-only, and onboard's newcomer question; block copies match the template
- [ ] Commit: `docs: document /donatello:tour`
