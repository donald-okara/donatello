# Phase 2: Design

### The catalog stores references, not copies
**Decision:** `catalog/skills.yaml` records each skill's source and install ref; only your own skills are stored as files (`catalog/skills/<id>/`).
**Why:** Android CLI skills and plugins have their own install and update channels; copying them into Donatello would fork them and go stale.
**Tradeoff:** Importing needs the network and the relevant tools (`android`, `git`). Failures are skipped and listed with the command to run later.

### Own skills live outside `skills/`
**Decision:** Donatello's domain skills go in `catalog/skills/`, and onboard copies chosen ones into the project's `.claude/skills/`.
**Why:** Everything in the plugin's `skills/` directory loads in every project. Keeping domain skills in the catalog means a project pays for only the skills it imported, and it matches how preferences are copied rather than linked.
**Tradeoff:** Copies drift from the catalog version. Accepted for now; a later `import --update` can refresh them.

### Edits go to your clone, not the installed plugin
**Decision:** `add` and `remove` edit the Donatello clone (the current directory if it's the repo, else the path saved in `~/.claude/donatello.json`) and offer to commit and push.
**Why:** The installed plugin is a cached copy that a marketplace update overwrites. The clone is the source of truth and syncs across machines through git.
**Tradeoff:** One-time setup of the clone path. Other machines see new entries only after a push and a `/plugin marketplace update`; reading from the clone when one is configured hides that lag on the machine where you added them.

### Scope is asked per import
**Decision:** Each import asks "committed" (shared through git and `.claude/settings.json`) or "just me" (`.claude/settings.local.json` and `.gitignore`).
**Why:** Your choice: team projects want shared skills, and personal experiments in shared repos shouldn't leak into teammates' setups.
**Tradeoff:** One more question per onboard.

### Git skills are pinned to a commit
**Decision:** `add` resolves a GitHub URL to a commit SHA and stores that.
**Why:** Imports are reproducible, and an upstream change can't silently alter a project's instructions.
**Tradeoff:** No automatic updates; re-adding refreshes the pin.

### Plugins install whole
**Decision:** A `plugin` entry enables the whole plugin; the import list says so.
**Why:** Claude Code enables plugins, not individual skills inside them.
**Tradeoff:** A plugin can bring more skills than wanted; plugin pruning and the summary make that visible.

### The import flow lives in the `skills` skill
**Decision:** `skills/skills/SKILL.md` defines the import flow; onboard follows it, the same way onboard reuses phase-new's steps.
**Why:** One definition of the flow for onboard and for later `/donatello:skills import` runs; keeps onboard's always-read text short.
**Tradeoff:** onboard depends on a sibling skill's path (`../skills/`), the same pattern as `../phase-new/`, which phase 1 already verified.
