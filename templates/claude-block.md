<!-- donatello:start -->
## Spec workflow (Donatello plugin)
- Preferences: @docs/preferences.md (file-specific rules live in .claude/rules/)
- Models: plan and discuss in the main session; implement via donatello:implementer, review via donatello:reviewer, survey via donatello:scout
- Orientation: architecture in docs/architecture.md, requirements in docs/requirements.md (read when needed)
- Current work: .spec/ongoing/. Read the active phase's overview, spec, tasks, and the "Current state" section of walkthrough.md
- After each task: tick it in tasks.md and update the phase's walkthrough.md (create if missing, with sections: Current state, How to verify, Changes by task, Deviations from spec)
- End of session: rewrite walkthrough.md "Current state"
- Rambles: on `ramble/*`, read only the frontmatter, Question and last 5 Log entries of that ramble's notes in `.spec/rambles/` (frontmatter `branch` matches); follow ramble mode; `/donatello:ramble` resumes
- Commands: /donatello:phase-new · /donatello:phase-discuss · /donatello:phase-run · /donatello:phase-archive · /donatello:ramble · /donatello:tour
<!-- donatello:end -->
