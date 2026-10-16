---
category: model-usage
summary: Opus plans and coordinates; Haiku implements; Sonnet reviews and takes over when Haiku can't finish
---

# Model usage

- **Opus (main session):** brainstorming, interviews, planning phases, architecture questions, coordinating agents. Don't write code or read full diffs and build logs in the main session; delegate.
- **Haiku (`donatello:implementer`, `donatello:scout`):** implementing well-specified tasks, surveying repos, other legwork.
- **Sonnet (`donatello:reviewer`):** reviewing every implemented task against its spec and these preferences; fallback implementer.

## Escalation
- Review fails twice, or the implementer reports it's blocked → re-run the task with the implementer on Sonnet.
- Sonnet also fails, or the task turns out to need a design decision → stop and discuss in the main session.

## Keeping it cheap
- Pass agents paths and requirement IDs, not file contents.
- Agents report failures, not full logs.
- `/clear` between tasks; resume from the walkthrough's "Current state".
