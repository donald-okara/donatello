# Phase 2: Skill Catalog

## Goal

Let you keep one catalog of the skills you use often, from the Android CLI, Claude plugin marketplaces, git repos, and Donatello itself, and import all or some of them into a project during onboard. That way a new project starts with the right skills and no manual installs.

## What This Phase Delivers

- `catalog/skills.yaml` in the Donatello repo: one entry per skill, with its source, install ref, `applies-to`, and summary
- `catalog/skills/<id>/`: your own skills, stored outside `skills/` so they don't load in every project
- `/donatello:skills` with `add`, `remove`, `list`, and `import`
- An onboard step that lists the matching catalog skills and asks all / pick / none, then committed / just me, and imports them
- Resume mode in onboard offers catalog skills the project doesn't have yet
- Updated `docs/SPECIFICATION.md`, `docs/ARCHITECTURE.md`, and README

## Out of Scope

- Updating already-imported git or Donatello skills to newer versions (a possible later `import --update`)
- Picking individual skills out of a plugin; plugins install whole
- Disabling Donatello's own workflow skills per project
- Per-stack Donatello plugins (`donatello-android`, …)

## Success Criteria

- `/donatello:skills add android:adaptive`, `add <plugin>@<marketplace>`, `add <GitHub skill URL>`, and `add <local folder>` each produce a correct catalog entry in your Donatello clone and offer to commit and push
- Onboarding a fresh Android project lists the Android-tagged and untagged catalog skills; picking a subset with "committed" installs exactly that subset, and a new session in the project sees those skills
- The same with "just me" leaves `git status` showing no skill files or shared settings changes
- A failing source (e.g. `android` not installed) doesn't stop onboarding; the summary lists the command to run later
- `/donatello:skills import` in an onboarded project offers only skills it doesn't have yet

## Related Docs

`docs/SPECIFICATION.md` (behavior) · `docs/ARCHITECTURE.md` (layout, components, edge cases) · Phase 1's `onboard` skill (`skills/onboard/SKILL.md`), which this phase extends
