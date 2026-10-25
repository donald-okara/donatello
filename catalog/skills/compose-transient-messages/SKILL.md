---
name: compose-transient-messages
description: Show toasts or snackbars in a Compose Multiplatform app so they survive navigation, and let ViewModels trigger them without holding UI types. Use when adding a snackbar or toast host, wiring one-off ViewModel events to a screen, or debugging a snackbar that vanishes when the screen pops.
---

# Transient messages that survive navigation

## The rule

A message survives navigation when its **host stays composed**, not because its state is a singleton. Put one host above the navigation display. Never put a `SnackbarHost` in each screen's `Scaffold` if a message must outlive the screen.

## Don't call `showSnackbar` from a screen's scope

`showSnackbar` suspends until the message is dismissed. If it runs in a ViewModel's `viewModelScope` or a screen's `LaunchedEffect`, popping the screen cancels the coroutine, and cancelling dismisses the message. "Save, then go back" kills the message in exactly the case you want to keep.

## Shape

1. **Root provider.** A composable above the nav display owns the `SnackbarHostState`, draws the host over its content, and provides a controller through a `CompositionLocal`. Outside a provider, the local defaults to a controller that drops messages (previews, tests).
2. **Fire-and-forget controller.** `show(message)` never suspends. It does `trySend` into an unlimited `Channel` and returns a handle with `dismiss()`. The provider drains the channel in its own scope, one message after another. Use a `Channel`, not a `SharedFlow`: a message sent before the host collects (startup, rotation) is kept.
3. **ViewModel sends events, not messages.** The ViewModel owns an `Events<E>` (an unlimited `Channel` exposed with `receiveAsFlow()`) and sends plain events such as `Saved`. It knows nothing about toasts or navigation.
4. **The Route maps events to presentation.** A `CollectEvents(events) { event -> }` composable in the Route decides what each event means: resolve the string, show the message, navigate. Message-then-navigate is two adjacent lines, and it works because `show` returns at once.
5. **Don't hand callbacks or a controller to the ViewModel.** The ViewModel outlives the composition, so after a rotation it holds a stale controller or navigator and events vanish.

## Lifecycle

- Collect events and drain the queue only while `STARTED`: `lifecycle.repeatOnLifecycle(Lifecycle.State.STARTED) { withContext(Dispatchers.Main.immediate) { ... } }`. A message that arrives in the background then waits in the channel instead of timing out unseen.
- `Main.immediate` matters: it stops a cancellation at a stop from dropping an element that was received but not yet handled.
- Wrap the handler in `rememberUpdatedState` so it always runs the latest lambda.

## Actions outlive the screen

A message's action callback can run after the screen that sent it is gone. Make the safe choice the easy one:

- `show(message)` on the screen's controller is **screen-bound**: it is dismissed, or skipped if still queued, when the screen leaves composition (`DisposableEffect` + `onDispose` dismissing the remembered handles). Its action may use the screen, for example "Retry".
- `showAppScoped(message)` **outlives the screen**: use it for a success message sent just before navigating and for "Undo". Its action may capture only ids and app-scoped functions, never a ViewModel, a navigator or screen state.
- A forgotten choice then gives a message that vanishes early, which is harmless, not a stale action running against a cleared ViewModel.
- Leaving composition also happens when another screen is pushed on top, because Navigation 3 stops composing entries below the top. Screen-bound messages on the screen underneath are dismissed then.

## Failing actions

Run the action inside `try/catch`: rethrow `CancellationException`, pass anything else to an `onActionError` callback. Without this, an exception ends the drain loop and crashes through the `LaunchedEffect`, and every later message is lost.

## Testing

`SnackbarHostState` works in plain `runTest` without composition. Collect the controller in `backgroundScope`, `runCurrent()`, then assert on `hostState.currentSnackbarData?.visuals?.message`, `performAction()` and `dismiss()`. Assert results (what is showing), not calls.

## Koin

If the controller is created by the root provider with `remember`, nothing needs a Koin binding, and `core/ui` can stay Koin-free. The Koin compiler plugin checks missing bindings at build time, including `koinInject` in composables, but how it treats a lookup in a feature module whose binding lives in another module is unverified.
