# Phase 1: Tasks

Each task ends with a commit. Verification means running the skill, not unit tests (see design.md).

```
1. Manifests + skeleton
   ├─> 2. Preferences (real content, incl. model-usage)
   ├─> 3. Templates
   └─> 4. Agents
          └─> 5. phase-new
                 ├─> 6. onboard (reuses phase-new, uses scout)
                 └─> 7. phase-run (uses implementer + reviewer)
                        └─> 8. phase-archive
                               └─> 9. Real-project trial + README
```

---

### Task 1: Plugin manifests and skeleton
- [x] Create `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`
- [x] Create stub `skills/{onboard,phase-new,phase-run,phase-archive}/SKILL.md` with frontmatter (`name`, `description`, `disable-model-invocation: true`, `argument-hint`)
- [x] Create stub `agents/{implementer,reviewer,scout}.md` with frontmatter (`name`, `description`, `model`, `tools`, `maxTurns`)
- [x] Add `preferences/.gitkeep`, `templates/.gitkeep`
- [x] `claude plugin validate .` passes (marketplace and plugin manifests)
- [x] Plugin loads: `claude plugin details` lists 4 skills and 3 agents
- [x] Settle the open question: a stub skill reads `../../preferences/` under both `--plugin-dir` and an installed copy (see design.md)
- [x] Commit: `feat: add plugin manifests, skill and agent stubs`

### Task 2: Preferences
- [x] Write `preferences/model-usage.md`: Opus / Sonnet / Haiku roles and the escalation rule (no `paths`; always loaded)
- [x] Write `preferences/testing.md` from your actual practice
- [x] Write `preferences/naming.md`
- [x] Write one stack-specific file with `paths` (e.g. `preferences/android-ui.md`, `applies-to: [android]`, `paths: ["**/*.kt"]`)
- [x] Each has `category`, `summary`, optional `applies-to` / `paths` frontmatter
- [x] The general files (no `paths`) total ≤ 100 lines
- [x] Commit: `feat: add initial preferences`

### Task 3: Templates
- [x] `skills/phase-new/templates/overview.md`: Goal / Delivers / Out of scope / Success criteria
- [x] `skills/phase-new/templates/design.md`: Decision / Why / Tradeoff entries only
- [x] `skills/phase-new/templates/spec.md`: R1… with scope and a check each; NFRs
- [x] `skills/phase-new/templates/tasks.md`: dependency sketch + `- [x]` checklist, commit line per task
- [x] `skills/phase-new/templates/walkthrough.md`: Current state / How to verify / Changes by task / Deviations from spec, with one-line guidance each
- [x] `templates/claude-block.md`: marked block per ARCHITECTURE.md, including model routing and walkthrough upkeep; confirm < 300 tokens (~208)
- [x] Commit: `feat: add phase and CLAUDE.md templates`

### Task 4: Agents
- [x] `agents/implementer.md`: `model: haiku`, all tools except Agent, `maxTurns: 40`; one task only; quiet builds/tests; ≤ 15-line report; no ticking, `.spec/` edits, or commits
- [x] `agents/reviewer.md`: `model: sonnet`, `effort: medium`, Read/Grep/Glob/Bash, `maxTurns: 20`; `PASS` or ≤ 10 `file:line` issues against task, spec, and preferences
- [x] `agents/scout.md`: `model: haiku`, Read/Grep/Glob/Bash (read-only commands), `maxTurns: 25`; ≤ 60-line repo summary
- [x] Descriptions short (each loads into every session's agent listing)
- [x] Verify: each agent runs on its intended model (check the model shown in the agent's transcript)
- [x] Verify: implementer on a failing Gradle build reports only the errors, not the full log
- [x] Commit: `feat: add implementer, reviewer, and scout agents`

### Task 5: `phase-new` skill
- [ ] Write SKILL.md: read context → compute N and slug → clarify scope → fill templates → show for review
- [ ] Task shaping: follow `docs/preferences.md`; tasks precise enough for Haiku (files, expected behavior, how to check); name skills where relevant
- [ ] Verify in a fixture with `phase1-*` archived → creates `phase2-<slug>/`
- [ ] Verify in an empty fixture → creates `phase1-<slug>/`
- [ ] Commit: `feat: implement phase-new skill`

### Task 6: `onboard` skill
- [ ] Write SKILL.md: scout → interview (skip on `--defaults`) → propose → write on approval
- [ ] Preferences split: general → `docs/preferences.md`; `paths` files → `.claude/rules/`; flag if general total exceeds 100 lines
- [ ] Plugin pruning: list enabled plugins, propose ones to disable for this stack, write chosen ones to `.claude/settings.local.json` (merge, don't replace)
- [ ] `docs/architecture.md` from the scout report (existing) or the interview (new)
- [ ] Phase 1 by following `../phase-new/SKILL.md` and its templates
- [ ] CLAUDE.md block: append, or replace between markers with a diff
- [ ] Cover every edge-case row in ARCHITECTURE.md
- [ ] Verify: empty fixture with `--defaults`
- [ ] Verify: existing fixture with its own README + CLAUDE.md; original content intact outside markers
- [ ] Verify: the main session's transcript shows the scout report, not raw repo file reads
- [ ] Verify: re-run on an onboarded fixture → resumes, doesn't duplicate
- [ ] Verify: fresh session in the fixture answers "what are we working on?" correctly
- [ ] Verify: a Kotlin-file edit loads the path-scoped rule; a README edit doesn't
- [ ] Commit: `feat: implement onboard skill`

### Task 7: `phase-run` skill
- [ ] Write SKILL.md: pick task → implementer → reviewer → one fix round → escalate to Sonnet → stop and ask; on PASS commit, tick, update walkthrough
- [ ] Pass paths and requirement IDs to agents, not file contents
- [ ] `--all` continues until done or a step needs the user
- [ ] Verify: a simple task completes end-to-end; `walkthrough.md` created with all four sections
- [ ] Verify: a deliberately underspecified task triggers the fix round, then escalation
- [ ] Verify: main-session context after one task holds agent reports only (check `/context`)
- [ ] Verify: working without phase-run, the CLAUDE.md block alone still gets the agent to tick tasks and update the walkthrough
- [ ] Commit: `feat: implement phase-run skill`

### Task 8: `phase-archive` skill
- [ ] Write SKILL.md: pick phase → handle unchecked tasks → finalize walkthrough → promote decisions and requirements (diff + approval) → `git mv` to dated archive dir
- [ ] Verify: full loop phase-new → phase-run → phase-archive → phase-new in a fixture; `docs/requirements.md` created on first archive and updated, not duplicated, on the second
- [ ] Verify: unchecked tasks prompt the three choices
- [ ] Verify: archiving a phase with no walkthrough builds one from tasks and git history
- [ ] Commit: `feat: implement phase-archive skill`

### Task 9: Real-project trial and README
- [ ] Push to GitHub; install via `/plugin marketplace add donald-okara/donatello` and `/plugin install donatello@donatello`
- [ ] Onboard one real Android project and one real web project; run at least two tasks in each with `/donatello:phase-run`
- [ ] Log issues in this phase's `walkthrough.md`, including where Haiku needed escalation
- [ ] Fix skill and agent text for each issue found
- [ ] Update README: what it is, install, the four commands, the agents, the files it creates
- [ ] Commit: `docs: README and fixes from real-project trial`
- [ ] Archive this phase with `/donatello:phase-archive`
