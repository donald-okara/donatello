# Phase 1: Tasks

Each task ends with a commit. Verification means running the skill, not unit tests (see design.md).

```
1. Manifests + skeleton
   ├─> 2. Preferences (real content)
   └─> 3. Templates
          └─> 4. phase-new
                 ├─> 5. onboard (reuses phase-new)
                 └─> 6. phase-archive
                        └─> 7. Real-project trial + README
```

---

### Task 1: Plugin manifests and skeleton
- [ ] Create `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`
- [ ] Create stub `skills/{onboard,phase-new,phase-archive}/SKILL.md` with frontmatter (`name`, `description`, `disable-model-invocation: true`, `argument-hint`)
- [ ] Add `preferences/.gitkeep`, `templates/.gitkeep`
- [ ] `claude plugin validate .` passes
- [ ] `claude --plugin-dir .` shows all three `/donatello:*` commands
- [ ] Settle the open question: a stub skill reads `../../preferences/` under both `--plugin-dir` and an installed copy
- [ ] Commit: `feat: add plugin manifests and skill stubs`

### Task 2: Preferences
- [ ] Write `preferences/testing.md` from your actual practice
- [ ] Write `preferences/naming.md`
- [ ] Write one stack-specific file (e.g. `preferences/android-ui.md`, `applies-to: [android]`)
- [ ] Each has `category`, `summary`, optional `applies-to` frontmatter
- [ ] Commit: `feat: add initial preferences`

### Task 3: Templates
- [ ] `skills/phase-new/templates/overview.md`: Goal / Delivers / Out of scope / Success criteria / Outcome
- [ ] `skills/phase-new/templates/design.md`: Decision / Why / Tradeoff entries; Handoff notes
- [ ] `skills/phase-new/templates/spec.md`: R1… with scope and a check each; NFRs
- [ ] `skills/phase-new/templates/tasks.md`: dependency sketch + `- [ ]` checklist, commit line per task
- [ ] `templates/claude-block.md`: marked block per ARCHITECTURE.md; confirm < 300 tokens
- [ ] Commit: `feat: add phase and CLAUDE.md templates`

### Task 4: `phase-new` skill
- [ ] Write SKILL.md: read context → compute N and slug → clarify scope → fill templates → show for review
- [ ] Task shaping rules: follow `docs/preferences.md`; name skills in tasks where relevant
- [ ] Verify in a fixture with `phase1-*` archived → creates `phase2-<slug>/`
- [ ] Verify in an empty fixture → creates `phase1-<slug>/`
- [ ] Commit: `feat: implement phase-new skill`

### Task 5: `onboard` skill
- [ ] Write SKILL.md: inspect → interview (skip on `--defaults`) → propose → write on approval
- [ ] Preference filtering by `applies-to`; write `docs/preferences.md`
- [ ] `docs/architecture.md` from code (existing) or interview (new)
- [ ] Phase 1 by following `../phase-new/SKILL.md` and its templates
- [ ] CLAUDE.md block: append, or replace between markers with a diff
- [ ] Cover every edge-case row in ARCHITECTURE.md
- [ ] Verify: empty fixture with `--defaults`
- [ ] Verify: existing fixture with its own README + CLAUDE.md; original content intact outside markers
- [ ] Verify: re-run on an onboarded fixture → resumes, doesn't duplicate
- [ ] Verify: fresh session in the fixture answers "what are we working on?" correctly
- [ ] Commit: `feat: implement onboard skill`

### Task 6: `phase-archive` skill
- [ ] Write SKILL.md: pick phase → handle unchecked tasks → write Outcome → promote decisions → `git mv` to dated archive dir
- [ ] Verify: full loop phase-new → phase-archive → phase-new in a fixture
- [ ] Verify: unchecked tasks prompt the three choices
- [ ] Commit: `feat: implement phase-archive skill`

### Task 7: Real-project trial and README
- [ ] Push to GitHub; install via `/plugin marketplace add donald-okara/donatello` and `/plugin install donatello@donatello`
- [ ] Onboard one real Android project and one real web project; log issues in design.md handoff notes
- [ ] Fix skill text for each issue found
- [ ] Update README: what it is, install, the three commands, the files it creates
- [ ] Commit: `docs: README and fixes from real-project trial`
- [ ] Archive this phase with `/donatello:phase-archive`
