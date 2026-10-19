# Phase 5: Newcomer Tour

## Goal

Help someone who is new to a repo get oriented, whether or not the repo is already onboarded to Donatello. Add a read-only `/donatello:tour`, and make `/donatello:onboard` stop asking newcomers questions only the project's authors can answer.

## What This Phase Delivers

- `/donatello:tour [area]`: a read-only, chat-only guided walk through the project in six stops (what this is, map, run it, how code is written here, where work stands, where to start), pausing after each for questions
- Works from the Donatello docs when the repo is onboarded, and from a `donatello:scout` report when it isn't; `[area]` narrows every stop to one part of the repo
- `/donatello:onboard` asks "Are you new to this repo?" (default guessed from git authorship):
  - already onboarded: offers the tour instead of resume mode, and warns that resume mode edits shared files
  - not onboarded: newcomer mode, with "don't know" answers going to an Open questions list, an optional phase 1 ("docs only"), and a tour offer at wrap-up
- `/donatello:tour` in the CLAUDE.md block's Commands
- Updated `docs/SPECIFICATION.md`, `docs/ARCHITECTURE.md`, and README

## Out of Scope

- Writing any tour output to a file (notes or a shared onboarding doc); the tour is chat only
- Changing onboard's behavior for people who aren't new, or with `--defaults`
- Changing the scout's report format
- Tours of archived phases in detail (only their names are shown)

## Success Criteria

- On an onboarded fixture, `/donatello:tour` walks all six stops from the docs without reading source files, and writes or commits nothing
- On a non-onboarded fixture, the tour runs from a scout report, skips "Where work stands", and ends by suggesting `/donatello:onboard`
- `/donatello:tour <area>` limits every stop to that area
- Onboard on an onboarded fixture, answering "new", offers the tour; onboard on a non-onboarded fixture, answering "new" and "don't know" to open questions, writes those to the architecture doc's Open questions and, with "docs only", creates no phase
- Onboard with `--defaults`, or answering "not new", behaves as before

## Related Docs

`skills/onboard/SKILL.md` · `agents/scout.md` · `templates/claude-block.md` · `docs/SPECIFICATION.md` · `docs/ARCHITECTURE.md`
