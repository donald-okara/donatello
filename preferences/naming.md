---
category: naming
summary: Case by case; screens use Route + Content; components are [Title][Type]
applies-to: [android]
---

# Naming

- Mostly case by case; follow what the surrounding code does.
- **Screens:** Route + Content pattern. `ProfileRoute` connects to the ViewModel, collects state, and handles navigation and effects; `ProfileContent` is stateless, takes state and callbacks, and is what previews render.
- **Components:** `[Title][Type]`, e.g. `ProfileHeader`, `CelebrationSection`.
