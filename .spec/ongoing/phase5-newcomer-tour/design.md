# Phase 5: Design

### A separate tour, not an onboard mode
**Decision:** orientation is its own skill, `/donatello:tour`; onboard only asks whether you're new and hands off to it.
**Why:** onboard writes shared files and is already long; a read-only tour is safe to repeat and to run on any repo.
**Tradeoff:** one more command in the block and the README.

### Chat only
**Decision:** the tour writes nothing: no notes file, no shared onboarding doc.
**Why:** a shared doc would duplicate the architecture doc, and the docs plus the scout already hold the facts; repeating the tour is cheap.
**Tradeoff:** a newcomer who wants notes has to ask for them.

### Model-invocable tour
**Decision:** unlike the other skills, `tour` leaves model invocation on.
**Why:** it writes nothing, so triggering on "show me around" is harmless and useful.
**Tradeoff:** its description is always loaded (~30 tokens).

### Newcomer default from git authorship
**Decision:** onboard guesses "new" when the user has no commits, or under 5% of them, and asks to confirm.
**Why:** it's the cheapest signal available, and the question is still asked, so a wrong guess costs one answer.
**Tradeoff:** shared or changed emails can skew the guess.

### "Don't know" goes to Open questions
**Decision:** in newcomer mode, unanswered architecture questions are listed in an Open questions section of the architecture doc instead of being guessed or dropped.
**Why:** the team can fill them in later, and the agents don't build on guesses.
**Tradeoff:** the architecture doc carries a short unresolved list until someone answers it.
