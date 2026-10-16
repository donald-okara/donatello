---
category: android-ui
summary: Compose only; previews mandatory, at the bottom, wrapped and grouped via :core:ui; screens own their Scaffold; hoisted state; components take only what they need
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

### Preview wrappers (smart wrapping)
Every preview goes through a wrapper so it looks the way it will at runtime. Wrappers live in `:core:ui`.
- **Component wrapper:** project theme → `Surface` → small padding, so the component isn't cramped against the edges.
- **Screen wrapper:** project theme → `Surface(Modifier.fillMaxSize())`, no padding. It provides only what the navigation host (e.g. `NavDisplay`) provides at runtime, which is ideally nothing beyond the theme, since each screen brings its own `Scaffold` and background.
- **Scene wrappers:** screens shown as Navigation 3 scenes (dialogs, bottom sheets, side panes, list-detail) get a wrapper that reproduces that scene's decoration, so the preview matches how the scene presents them.

### Preview annotations
- Group preview configurations into named multipreview annotations, defined once per project in `:core:ui`: one for components (themes, font scales) and one for screens (adds screen sizes). Name them for the project.
- Use the grouped annotations instead of stacking individual `@Preview`s.

### Setting up `:core:ui`
- Create the preview wrappers and grouped annotations first, before any feature UI, so every later preview uses them from the start.

## State
- Hoist state. Composables receive state and callbacks; they don't own business state.
- With MVI (one consolidated screen state), each component takes **only the fields it needs**, not the whole state object, so it doesn't recompose when unrelated fields change.
