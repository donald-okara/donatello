---
name: phase-discuss
description: Revise an ongoing phase's plan (requirements, scope, tasks) through a conversation, then apply one approved diff across the phase files. Ticked tasks are never changed.
argument-hint: "[topic]"
disable-model-invocation: true
---

# Discuss a phase plan

Rethink the ongoing phase's plan with the user, then apply the result as one diff they approve. Topic: `$ARGUMENTS`

Ticked tasks are frozen: their text never changes, and rework becomes a new task. Nothing is written until the user approves the diff.

## 1. Load

- **Phase:** the single directory in `.spec/ongoing/`. If there are several, ask which one. If there are none, suggest `/donatello:phase-new` and stop. Archived phases are never revised.
- **Ramble branch:** if the current branch is `ramble/*`, stop with "this is a ramble branch; end it with `/donatello:ramble end` or switch to main".
- **Read:** `overview.md`, `spec.md`, `design.md`, `tasks.md`, and the Current state section of `walkthrough.md` if it exists. Don't read source code. If a fact about the code matters to the discussion, dispatch the `donatello:scout` agent and work from its report.
- **Note:** which tasks are ticked, and which requirements each task serves (its `**Serves:**` line). Keep this in mind for the rest of the session.

## 2. Discuss

- Start from the topic if one was given; otherwise ask what the user wants to change. Ask one question at a time.
- If a requested change contradicts the overview's goal, say so and point to the goal.
- If the request is really new work rather than a revision (it's unrelated to the phase's goal), say that it belongs in a new phase, and offer to record it under `overview.md` "Out of Scope" as a `Deferred:` line rather than adding it to `tasks.md`.
- Keep the discussion in chat. Nothing goes into the phase files until section 3.
