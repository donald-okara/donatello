---
name: skills
description: Manage the Donatello skill catalog: list entries, add or remove one (in your Donatello clone), or import catalog skills into this project.
argument-hint: "[add <ref> | remove <id> | list | import]"
disable-model-invocation: true
---

# Skill catalog

Manage `catalog/skills.yaml`, the list of skills that onboard and `import` can offer. Arguments: `$ARGUMENTS` (`add <ref>`, `remove <id>`, `list`, or `import`; with no argument, show `list` and ask which to run).

Field meanings and sources are in the header of `catalog/skills.yaml`.

## Find the catalog

1. **Clone:** the current directory, if its `.claude-plugin/plugin.json` has `"name": "donatello"`. Otherwise the `clone` path in `~/.claude/donatello.json`. Otherwise, for `add` and `remove` only, ask for the path once and save it there as `{"clone": "<path>"}`.
2. **Read** `catalog/skills.yaml` from the clone if there is one. Otherwise read `../../catalog/skills.yaml` (the installed copy). `list` and `import` never ask for a clone; they use the installed copy without prompting.
3. **`add` and `remove` need the clone.** If the user declines, or the path has no `catalog/skills.yaml`, stop and say so. Never edit the installed plugin copy.

## list

1. Group the entries by `source` (`android`, `plugin`, `git`, `donatello`).
2. Print one line per entry: `id` · `applies-to` (or "all") · `summary`.

## add

`add <ref>` accepts these forms:

| Ref form | Source |
|---|---|
| `android:<id>` | `android` (`ref` = the id) |
| `<plugin>@<marketplace>` | `plugin` |
| GitHub URL of a folder containing `SKILL.md` | `git` |
| local folder containing `SKILL.md` | `donatello` |

1. **Detect** the source from the ref. Anything else: stop and list the accepted forms.
2. **Fill `id` and `summary`:**
   - `android`: run `android skills list` and find the id. Take its name and description, condensed to one line. If the id is not listed, stop and say "not found".
   - `plugin`: find the plugin in `.claude-plugin/marketplace.json` under `~/.claude/plugins/marketplaces/<marketplace>/`. Take its description, condensed to one line. Take the marketplace's source from `~/.claude/plugins/known_marketplaces.json`: the `<marketplace>` entry's `source.repo`, when its `source` is `github`. If the marketplace is missing from that file, or the plugin is missing from `marketplace.json`, stop and say "not found".
   - `git`: parse the URL as `https://github.com/<owner>/<repo>/tree/<ref>/<path>`. `id` is the last segment of `<path>`. Resolve `<ref>` to a commit SHA:
     - A 40-character hex string is already a SHA. Use it as is.
     - Otherwise, a branch name may contain `/`. Try the longest prefix of the segments that `git ls-remote https://github.com/<owner>/<repo>.git refs/heads/<prefix>` returns, and give the rest to `<path>`. If no prefix or more than one matches, ask which one is meant.
     - A URL with no `tree/` segment means `HEAD`: `git ls-remote https://github.com/<owner>/<repo>.git HEAD`.
     Fetch `<path>/SKILL.md` at that SHA with `https://raw.githubusercontent.com/<owner>/<repo>/<sha>/<path>/SKILL.md`, and take `name` and `description` from its frontmatter, condensed to one line. If the fetch fails, stop and say so.
     The stored `ref` is `https://github.com/<owner>/<repo>/tree/<sha>/<path>`, pinned to the SHA.
   - `donatello`: read the local `SKILL.md` frontmatter. `id` is the folder name. The `ref` is `catalog/skills/<id>`.
3. **Refuse** if `id` already exists in the catalog.
4. **Suggest `applies-to`** from the skill's content (e.g. `[android]` for Android UI skills). Leave it out when the skill fits any stack.
5. **Show** the entry. For a `donatello` entry, also show the folder that will be copied.
6. **Write on approval only.** Add the entry at the end of the list, keeping the header comments and the existing entries as they are. For a `donatello` entry, copy the folder into `catalog/skills/<id>/`.

## remove

1. **Confirm** the `id` and say what will be deleted.
2. **Delete** the entry. For a `donatello` entry, also delete `catalog/skills/<id>/`.
3. Keep the rest of the file unchanged.

## After add or remove

Offer to commit in the clone, then push. Don't commit or push without a yes.

- add: `feat: add <id> to skill catalog`
- remove: `chore: remove <id> from skill catalog`

## import

Installs catalog skills into the current project. The four parts below can run at different times, so a caller can use some of them alone:
- **Candidates** ends with a list and asks nothing.
- **Ask** turns the user's answers into chosen ids and a scope.
- **Install** takes those ids and that scope as input.
- **Summary** reports the results.

Standalone `import` runs all four in order. Onboard runs Candidates and Ask as its own questions (it can show the Candidates list in its proposal instead of showing it again), then runs Install and Summary later.

Run it from the project root. The stack comes from the caller (onboard passes the stack it found); standalone, ask once: "Which stacks does this project use?" (for example `android`, `web`, or none).

### Candidates

1. **Read** the catalog (see "Find the catalog"; `import` never asks for a clone).
2. **Keep** entries whose `applies-to` includes the project's stack, or that have no `applies-to`.
3. **Split** the kept entries into:
   - **present**: `.claude/skills/<id>/` exists, or the plugin is a key of `enabledPlugins` in `.claude/settings.json` or `.claude/settings.local.json`. Don't offer these; they go in the summary as already present.
   - **candidates**: the rest.
4. **Show** the candidates grouped by source, with "(installs the whole plugin)" on each `plugin` entry. If there are no candidates, end the import and go to Summary (which still lists the present entries).

### Ask

1. Ask **all / pick / none**. "All" selects every candidate. "Pick" takes ids from the candidate list; an id that isn't in the list is rejected and the question repeats. "None" ends the import and goes to Summary.
2. Ask **committed / just me** once for the whole import:
   - **committed**: shared through git. Files go in `.claude/` and are meant to be committed.
   - **just me**: stays on this machine. Skill folders and the plugin settings file are listed in `.gitignore`.

Output: the chosen ids (from the candidates only) and the scope.

### Install

Input: the chosen ids and the scope. Never replace an existing `.claude/skills/<id>/` folder or an existing settings value. Before each entry, check that `.claude/skills/<id>/` doesn't exist (folder sources only). If it has appeared since Candidates, skip the entry, list it as already present, and don't add it to `.gitignore`.

| Source | Committed | Just me |
|---|---|---|
| `android` | `android skills add <id> --agent=claude-code --project=.` | same command, then gitignore the folder |
| `plugin` | `.claude/settings.json`: add the marketplace to `extraKnownMarketplaces` (if missing) and `"<plugin>@<marketplace>": true` to `enabledPlugins` (if missing) | `.claude/settings.local.json`: the same two entries; the marketplace goes in only if `~/.claude/plugins/known_marketplaces.json` doesn't already have it; then gitignore the settings file |
| `git` | fetch the pinned folder (see below) into `.claude/skills/<id>/` | same, then gitignore the folder |
| `donatello` | copy `catalog/skills/<id>/` from the same location the catalog was read from (clone or installed copy) into `.claude/skills/<id>/` | same, then gitignore the folder |

- **Marketplace entry:** `extraKnownMarketplaces` is keyed by the marketplace name (the part after `@` in the ref), with value `{"source": {"source": "github", "repo": "<owner/repo>"}}`, where `<owner/repo>` is the entry's `marketplace` field.
- **Settings files:** read the file (create it as `{}` if missing) and add only the missing keys. Keep every other key and value. If the file isn't valid JSON, don't touch it and fail the entry.
- **Git folder:** run every git command with `GIT_TERMINAL_PROMPT=0`, so a private or missing repo fails instead of prompting. In a new scratch directory made with `mktemp -d` (outside the project): `git init -q`, `git remote add origin https://github.com/<owner>/<repo>.git`, `git fetch -q --depth 1 origin <sha>`, `git checkout -q FETCH_HEAD -- <path>`. Check that `<path>/SKILL.md` exists in the scratch directory; if not, fail the entry. Only then copy `<path>` into `.claude/skills/<id>/`. Delete the scratch directory whether or not the install succeeded. If the copy fails, remove any partial `.claude/skills/<id>/`.
- **Gitignore (just me):** create `.gitignore` if missing, then append each just-me folder as `.claude/skills/<id>/` if that line isn't already there. For a just-me plugin, also make sure `.claude/settings.local.json` is ignored: run `git check-ignore -q .claude/settings.local.json`; exit 1 means it isn't ignored, so append that line. Exit 128 means the project isn't a git repo; skip the check. Add a newline before the first appended line if the file doesn't end with one.
- **Failures** (missing CLI, no network, unknown marketplace, bad ref, unreadable settings file, missing `SKILL.md`): skip that entry and carry on. Record the entry and its fallback command:
  - `android`: `android skills add <id> --agent=claude-code --project=.`
  - `plugin`: `/plugin marketplace add <owner/repo>` (only if the marketplace is unknown), then `/plugin install <plugin>@<marketplace>`. If the cause was invalid JSON, also name the file (`.claude/settings.json` for committed, `.claude/settings.local.json` for just me) and the entries to add: `"extraKnownMarketplaces": {"<name>": {"source": {"source": "github", "repo": "<owner/repo>"}}}` and `"enabledPlugins": {"<plugin>@<marketplace>": true}`.
  - `git`: fix the cause, then re-run `/donatello:skills import`. Manual alternative: fetch `<path>` at `<sha>` from `https://github.com/<owner>/<repo>` (the commands above) and copy it to `.claude/skills/<id>/`.
  - `donatello`: fix the cause (usually `catalog/skills/<id>/SKILL.md` is missing from the catalog), then re-run `/donatello:skills import`. Manual alternative: `cp -R <catalog location>/catalog/skills/<id> .claude/skills/<id>`.

### Summary

Print these, even when the import stopped early:
- **Imported**: each entry with its scope (committed or just me).
- **Already present**: the present entries, plus any entry skipped at Install because its folder appeared.
- **Failed**: each entry with its fallback command.
- **Committed files**: the paths written for committed entries (for onboard's commit).
- If any plugin was enabled: "Start a new session to load newly enabled plugins."
