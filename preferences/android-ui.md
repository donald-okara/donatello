---
category: android-ui
summary: Compose only; previews mandatory, at the bottom, with full coverage; hoisted state; components take only what they need
applies-to: [android]
paths: ["**/*.kt"]
---

# Android UI

## Compose
- Jetpack Compose only, unless decided otherwise for the project.
- See the UI before worrying about the code: build the preview early, then refine.

## Previews
- Every composable has a preview. Previews go **at the bottom of the file**.
- Aim for coverage:
  - light and dark theme
  - text sizes (font scale)
  - screen sizes, for screen-level previews
  - states via `@PreviewParameter` providers rather than one preview per state
- Previews render the stateless `...Content` / component, never the `...Route`.

## State
- Hoist state. Composables receive state and callbacks; they don't own business state.
- With MVI (one consolidated screen state), each component takes **only the fields it needs**, not the whole state object, so it doesn't recompose when unrelated fields change.
