# Phase 2: Specification

## Requirements

### R1. Catalog file
- `catalog/skills.yaml` is a YAML list. Each entry has:
  - `id`: unique, kebab-case
  - `source`: one of `android`, `plugin`, `git`, `donatello`
  - `ref`: the install reference for that source (see R2)
  - `summary`: one line, shown during onboard
  - optional `applies-to`: stack list, same meaning as in preferences; omitted means every stack
  - `plugin` entries only: `marketplace`, the marketplace's source (e.g. a GitHub `owner/repo`), so a project's settings can declare it
- `donatello` entries keep their skill folder in `catalog/skills/<id>/`, never in the plugin's `skills/` directory
- The file ships with a short header comment explaining the fields, plus the skills currently in use: `adaptive`, `navigation-3`, `styles` (android, `applies-to: [android]`), and `superpowers` (plugin `superpowers@claude-plugins-official`, marketplace `anthropics/claude-plugins-official`)
- **Check:** the file parses as YAML; every entry has the required fields; every `donatello` entry's folder exists and has a `SKILL.md`

### R2. Sources and refs

| `source` | `ref` | `add` accepts |
|---|---|---|
| `android` | Android CLI skill id | `android:<id>` |
| `plugin` | `<plugin>@<marketplace>` | `<plugin>@<marketplace>` |
| `git` | GitHub URL of the skill folder, pinned to a commit SHA | a GitHub URL of a folder containing `SKILL.md` (branch or SHA) |
| `donatello` | `catalog/skills/<id>` | a local folder containing `SKILL.md` |

- **Check:** each "accepts" form in the table is recognized by `add` and stored as the `ref` form

### R3. `/donatello:skills` command
- Explicit-only skill at `skills/skills/SKILL.md`; argument hint `[add <ref> | remove <id> | list | import]`
- **Clone location:** `add` and `remove` edit the catalog in your Donatello clone, never the installed plugin copy. The clone is the current directory if it is the Donatello repo; otherwise the path in `~/.claude/donatello.json` (`{"clone": "<path>"}`); otherwise ask once and save it there.
- **Reading the catalog** (`list`, `import`, onboard): from the clone when one is configured, else from the installed plugin copy
- **`add <ref>`:** detects the source (R2), fills `id` and `summary` from the skill's own name and description (Android: `android skills find`/`list`; plugin: the marketplace's plugin description; git and local: the `SKILL.md` frontmatter), suggests `applies-to`, shows the entry, and writes it after approval. Git refs are pinned to the current commit SHA. Local folders are copied into `catalog/skills/<id>/`. Refuses a duplicate `id`. Then offers to commit and push the clone.
- **`remove <id>`:** deletes the entry (and the `catalog/skills/<id>/` folder for `donatello` entries) after confirmation, then offers to commit and push
- **`list`:** entries grouped by source: id, `applies-to`, summary
- **`import`:** runs the import flow (R4) in the current project
- **Check:** each subcommand run from a project other than the Donatello repo changes only the clone (and, for `import`, the current project)

### R4. Import flow
- **Candidates:** catalog entries whose `applies-to` matches the project's stack or is omitted, minus ones the project already has. "Already has" means: `.claude/skills/<id>/` exists, or the plugin is in `enabledPlugins` of `.claude/settings.json` or `.claude/settings.local.json`.
- **Show** the candidates grouped by source, noting that a `plugin` entry installs every skill in that plugin. Ask **all / pick / none**, then **committed / just me** (asked once per import).
- **Install:**

  | Source | Committed | Just me |
  |---|---|---|
  | `android` | `android skills add <id> --agent=claude-code --project=.` (installs to `.claude/skills/<id>/`) | same, then `.claude/skills/<id>/` goes in `.gitignore` |
  | `plugin` | `.claude/settings.json`: add the marketplace to `extraKnownMarketplaces` and the plugin to `enabledPlugins` | `.claude/settings.local.json`: add the plugin to `enabledPlugins` (and the marketplace if it's unknown on this machine) |
  | `git` | fetch the pinned folder into `.claude/skills/<id>/` | same, then `.claude/skills/<id>/` goes in `.gitignore` |
  | `donatello` | copy `catalog/skills/<id>/` into `.claude/skills/<id>/` | same, then `.claude/skills/<id>/` goes in `.gitignore` |

- Settings files are merged: existing keys are kept, and only the needed entries are added
- **Failures** (missing CLI, no network, unknown marketplace, bad ref) skip that entry and carry on; the summary lists each failed entry with the command to run later
- **Summary:** imported (by scope), skipped as already present, failed with commands; a note that newly enabled plugins need a new session
- **Check:** importing one entry of each source, once with "committed" and once with "just me", produces the files in the table; with "just me", `git status` shows only `.gitignore` and nothing under `.claude/skills/` or in `.claude/settings.json`

### R5. Onboard integration
- After picking preferences (step 3), onboard computes the import candidates (R4) for the stack the scout found
- The proposal (step 5) lists them; the interview asks all / pick / none and the scope
- Writing (step 6) runs the import flow's install step for the chosen entries
- Plugin pruning (6g) never proposes disabling a plugin that was just imported
- `--defaults` skips importing skills and mentions `/donatello:skills import` in the wrap-up
- Resume mode offers the candidates as one of the "missing pieces"
- The wrap-up summary includes the import summary; skills committed to the project are included in the offered onboard commit, "just me" ones are not
- **Check:** onboarding a fresh Android fixture with a catalog containing an android, a web-only, and an untagged entry offers the android and untagged ones only, and the chosen ones are installed

### R6. Documentation
- `docs/SPECIFICATION.md`: the catalog, the `/donatello:skills` command, and the onboard step; the line about domain skills shipping in the plugin is replaced by the catalog approach
- `docs/ARCHITECTURE.md`: the catalog and the new skill in Components, decisions from this phase's `design.md`, and edge cases (missing CLI, unknown marketplace, already-present skill, no clone configured)
- `README.md`: a "Skill catalog" section with examples, the command in the Commands section, and the repository layout updated
- **Check:** the docs mention every subcommand and every source, and nothing still says domain skills live in `skills/`

## Non-Functional Requirements

- **NF1. Token cost:** `/donatello:skills` is explicit-only, so always-on cost rises by no more than ~50 tokens; onboard's `SKILL.md` grows by at most ~25 lines and points to `skills/skills/SKILL.md` for the import flow rather than repeating it
- **NF2. No overwrites:** importing never replaces an existing `.claude/skills/<id>/` folder or existing settings values
