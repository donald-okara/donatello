---
category: testing
summary: Test results, not implementation; JUnit + Mockito, Robolectric for UI; previews count as tests
applies-to: [android]
---

# Testing

- **Test results, not behavior.** Assert on what comes out (returned values, emitted state, what's on screen), not on how it was produced. A refactor that keeps results the same must not break tests.
- Avoid interaction assertions (`verify(...)` on internal calls) unless the interaction *is* the result, such as a call to an external service.
- **Tools:** JUnit, Mockito for test doubles, Robolectric for UI tests.
- **Screenshot tests:** Google's Compose Preview Screenshot Testing, built on previews (a dedicated skill will cover setup).
- **Previews are part of testing.** See the Android UI rules for preview coverage.
