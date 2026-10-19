# Phase 4: Walkthrough

## Current state

Task 1 is done: `skills/phase-discuss/SKILL.md` has frontmatter, intro, "1. Load", and "2. Discuss".
Next: Task 2, proposing changes, checks, and apply. Its writes (including `Deferred:` lines offered in Discuss) must go through the approved diff.

## How to verify

- `claude plugin validate .` passes
- Task 1: walk one / several / no ongoing phases, the `ramble/*` refusal, the read limits, and the new-phase response through "Load" and "Discuss" against R1 and R2

## Changes by task

### Task 1: phase-discuss — load and discuss
- Added `skills/phase-discuss/SKILL.md`: frontmatter (`[topic]`, explicit-only), intro (ticked tasks frozen, nothing written without approval), "1. Load", "2. Discuss"
- The `ramble/*` refusal reuses phase-run's wording
- Reviewer NOTE: Load doesn't handle a topic that names a phase (optional)

## Deviations from spec

None
