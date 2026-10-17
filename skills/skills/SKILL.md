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

Defined in Task 3 of this phase. Not written yet.
