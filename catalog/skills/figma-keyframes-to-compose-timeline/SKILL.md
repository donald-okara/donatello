---
name: figma-keyframes-to-compose-timeline
description: Use when turning a Figma keyframe animation (get_motion_context output, CSS @keyframes, linear() spring curves) into Jetpack Compose or Compose Multiplatform, especially multi-layer choreography such as a celebration burst, confetti, or staggered fade-ups.
---

# Figma keyframes to a Compose timeline

Recreate a Figma prototype animation as **one clock and one window per layer**, drawn without recomposition.

## 1. Read the design, not just the motion
- `get_motion_context` (recursive) gives keyframes: percentages of one timeline, translate/rotate/scale/opacity, and easing per segment. It has **no fills, sizes or resting positions**.
- `get_design_context` on the component gives sizes, resting positions, token colours and the description. Don't guess geometry from the motion data: start offsets are relative to the resting place, and "starts at the centre" is only approximately true.
- When the component description contradicts the keyframes, follow the keyframes and list the conflict for the designer.
- Check the colours against the project's brand rules. A Figma fill can break them (for example, using a logo-only gradient as an accent).

## 2. Convert the timeline
- Percent × the loop length gives milliseconds. Figma loops prototypes; in the app it plays once.
- Each keyframe segment becomes a `TimelineWindow(startMillis, durationMillis, easing)`, with `fractionAt(t)` giving the eased 0..1 inside it.
- CSS easings map to Compose's own: `ease-out` is `EaseOut`, `ease-in-out` is `EaseInOut`, and `cubic-bezier(0.05, 0.7, 0.1, 1)` is M3 emphasized decelerate.
- `linear(...)` lists are springs. Estimate the damping from the peak overshoot, ζ = −ln(OS) / √(π² + ln²(OS)), and the stiffness from the time to peak, tp = π / (ω√(1−ζ²)) with stiffness = ω². Prefer an existing spring token if it's close. Use `springEasing(spring, durationMillis)` to put it in a window.
- Staggers are usually a motion token (for example, 40ms steps). Precompute per-item windows once; don't build them per frame.

## 3. Drive it
- One constant-speed `Animatable(0f)` running to 1 over the timeline (`rememberTimelinePlayback(duration, key)`). Expose it as lambdas (`TimelinePlayback(progress, alpha)`).
- Under reduce motion: jump to the resting frame and fade it in over the short duration. Haptics stay on; they aren't motion.
- Side effects such as haptics go in a `LaunchedEffect` timed on the clock, never in draw code.

## 4. Draw it
- Put the whole burst in a single `Modifier.drawWithCache`: build brushes and dp-to-px values in the cache block, and read `playback.progress()` only inside `onDrawBehind`. A running timeline then redraws without recomposing.
- Per piece: `withTransform({ translate; rotate(pivot = Offset.Zero); scale(pivot = Offset.Zero) })`, then draw centred on zero. Draw order sets the layering (pieces drawn before a medallion appear from behind it).
- Size-relative brushes (`Brush.horizontalGradient` without bounds) span the whole draw scope. `inset()` it to the shape's bounds first.
- Text and composables: use the lambda form of `graphicsLayer { alpha; translationY }` (for example, a `timelineFadeIn` modifier).
- Decorations that overflow an element: draw them in `drawBehind` before the element's own `graphicsLayer`, so they stay out of its layout and out of any pop scale.

## 5. Preview it
- Frozen frames: pass `TimelinePlayback(progress = { 0.3f })` through a `PreviewParameterProvider<Float>`.
- An interactive playground with Replay (change the key), a slider that scrubs `progress`, and a reduce-motion switch (provide the reduce-motion local).
- Cover light and dark, and 200% text wherever there's copy.

## Don't
- Reach for a particle library or a `withFrameNanos` physics loop for keyframed confetti that bursts and settles.
- Read animated values in composition (`by animateFloatAsState` read in the body), which recomposes every frame.
- Allocate per frame: no `indexOf`, no new windows, and no brushes inside `onDrawBehind`.
