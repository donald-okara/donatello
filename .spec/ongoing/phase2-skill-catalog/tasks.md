# Phase 2: Tasks

Each task ends with a commit. Verification means running the skill against fixtures in a scratch directory, not unit tests (same approach as phase 1).

```
1. Catalog file
   └─> 2. /donatello:skills — list, add, remove
          └─> 3. /donatello:skills — import
                 └─> 4. Onboard integration
                        ├─> 5. Fixture verification (needs Donald)
                        └─> 6. Docs and README
```

---

### Task 1: Catalog file
**Serves:** R1
**Files:** `catalog/skills.yaml`, `catalog/skills/`

- [ ] Create `catalog/skills.yaml` with a header comment listing the fields (`id`, `source`, `ref`, `summary`, optional `applies-to`, `marketplace` for plugins) and the four `source` values
- [ ] Add entries: `adaptive`, `navigation-3`, `styles` (`source: android`, `ref` = the id, `applies-to: [android]`, summary from `android skills list`), and `superpowers` (`source: plugin`, `ref: superpowers@claude-plugins-official`, `marketplace: anthropics/claude-plugins-official`, no `applies-to`)
- [ ] Create `catalog/skills/.gitkeep`
- [ ] Check: `python3 -c "import yaml; d=yaml.safe_load(open('catalog/skills.yaml')); assert all({'id','source','ref','summary'} <= set(e) for e in d)"` passes; `claude plugin validate .` still passes
- [ ] Commit: `feat: add skill catalog`

### Task 2: `/donatello:skills` — list, add, remove
**Serves:** R2, R3
**Files:** `skills/skills/SKILL.md`

- [ ] Frontmatter: `name: skills`, a one-line `description`, `argument-hint: "[add <ref> | remove <id> | list | import]"`, `disable-model-invocation: true`
- [ ] Section "Find the catalog": clone = current directory if its `.claude-plugin/plugin.json` has `"name": "donatello"`, else `clone` in `~/.claude/donatello.json`, else ask for the path and save it there; read from the clone if found, else from `../../catalog/skills.yaml` (installed copy); `add`/`remove` require the clone
- [ ] Section "list": entries grouped by source, one line each: `id` · `applies-to` (or "all") · summary
- [ ] Section "add": detect the source from the ref per R2's table; fill `id`/`summary` (Android: `android skills list`; plugin: the marketplace's `.claude-plugin/marketplace.json` under `~/.claude/plugins/marketplaces/<name>/`, taking `marketplace` from `~/.claude/plugins/known_marketplaces.json`; git: fetch `SKILL.md` frontmatter and resolve the branch to a SHA with `git ls-remote`; local: read `SKILL.md` and copy the folder to `catalog/skills/<id>/`); suggest `applies-to`; refuse a duplicate `id`; show the entry and write it on approval, keeping the file's comments and order
- [ ] Section "remove": confirm, delete the entry and, for `donatello` entries, `catalog/skills/<id>/`
- [ ] After `add` or `remove`: offer to commit in the clone (`feat: add <id> to skill catalog` / `chore: remove <id> from skill catalog`) and push
- [ ] Check: from a scratch project (not the Donatello repo), `/donatello:skills add android:adaptive` refuses as a duplicate, `add` of a local fixture skill folder creates an entry and copies the folder into the clone, `list` shows it, `remove` deletes both; the scratch project is untouched
- [ ] Commit: `feat: add /donatello:skills with list, add, remove`

### Task 3: `/donatello:skills` — import
**Serves:** R4, NF2
**Files:** `skills/skills/SKILL.md`

- [ ] Section "import", written so onboard can follow it from its own step: work out the stack (from the caller's scout report, or ask), build candidates per R4 (matching `applies-to` or none; minus `.claude/skills/<id>/` present or plugin already in either settings file)
- [ ] Show candidates grouped by source with "(installs the whole plugin)" on plugin entries; ask all / pick / none, then committed / just me
- [ ] Install per R4's table; merge settings JSON keeping existing keys; never replace an existing `.claude/skills/<id>/`; for "just me" append `.claude/skills/<id>/` lines to `.gitignore` (creating it if needed, no duplicates)
- [ ] On a failure, record the entry and the exact command to run later, and continue
- [ ] Summary: imported (by scope), already present, failed with commands, and "start a new session to load newly enabled plugins" when any plugin was enabled
- [ ] Check: in a scratch git repo with a catalog of one entry per source, import all as "committed" → files per R4's table; reset, import all as "just me" → `git status --porcelain` shows only `.gitignore`; run import again → every entry reported as already present; with `android` removed from `PATH`, the android entry fails with its command and the rest succeed
- [ ] Commit: `feat: add catalog import to /donatello:skills`

### Task 4: Onboard integration
**Serves:** R5, NF1
**Files:** `skills/onboard/SKILL.md`

- [ ] Step 3: after deduplicating preferences, build the import candidates by following "import" in `../skills/SKILL.md` up to (not including) the questions
- [ ] Step 4: add the all / pick / none and committed / just me questions (skipped with `--defaults`)
- [ ] Step 5: list the chosen skills and their scope in the proposal
- [ ] Step 6: new sub-step "h. Skills" before plugin pruning: run the import's install step; 6g never proposes a plugin imported in 6h
- [ ] `--defaults`: skip skills and name `/donatello:skills import` in the wrap-up; resume mode lists catalog candidates among the missing pieces
- [ ] Step 7: include the import summary; leave "just me" files out of the onboard commit
- [ ] Check: onboard's `SKILL.md` grew by ≤ 25 lines; the import flow isn't repeated in it
- [ ] Commit: `feat: import catalog skills during onboard`

### Task 5: Fixture verification
**Serves:** R4, R5

- [ ] Fresh Android fixture, catalog with an android entry, a `applies-to: [web]` entry, and an untagged entry: onboard offers the android and untagged ones only; pick one, "committed"; a new session in the fixture lists that skill
- [ ] Same fixture, `/donatello:onboard` again (resume mode) offers the remaining candidate
- [ ] `/donatello:onboard --defaults` on another fixture imports nothing and mentions `/donatello:skills import`
- [ ] Log anything unexpected in this phase's `walkthrough.md` and fix the skill text
- [ ] Commit: `fix: adjust skill import from fixture trial` (only if fixes were needed)

### Task 6: Docs and README
**Serves:** R6
**Files:** `docs/SPECIFICATION.md`, `docs/ARCHITECTURE.md`, `README.md`

- [ ] SPECIFICATION: add the catalog and `/donatello:skills` workflow; replace "Domain skills … ship in the same plugin" with the catalog approach; add the onboard skill step; add `catalog/` to the repo tree
- [ ] ARCHITECTURE: a `skills` skill entry in Components; this phase's decisions; edge cases (missing CLI, unknown marketplace, skill already present, no clone configured)
- [ ] README: a "Skill catalog" section (format, sources table, `add` examples for each source, import scopes), the command in Commands and Quick start, the repo layout
- [ ] Check: `grep -n "catalog" README.md docs/*.md` covers every subcommand and source; nothing says domain skills live in `skills/`
- [ ] Commit: `docs: document skill catalog`
