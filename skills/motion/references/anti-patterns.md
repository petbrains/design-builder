# Motion anti-patterns

The Layer 2 Anti-Pattern filter (`design` skill) consumes this list when motion is in scope. Each item is a P0/P1 finding for `/improve` and `/review`.

## P0 — accessibility breakers

| Anti-pattern | Why | Fix |
|---|---|---|
| Motion without a `prefers-reduced-motion` fallback | Vestibular disorders, migraine, ADHD users get nauseated; WCAG 2.3.3 (AAA) explicitly calls this out | CSS: `@media (prefers-reduced-motion: reduce) { animation: none }`. GSAP: `gsap.matchMedia({ reduceMotion: ... })`. WAAPI/Anime/Lottie/Three: gate `if (matchMedia(...).matches) return` and provide a static alternative. |
| Auto-rotating carousel < 8 s with no pause control | Users with reading disabilities can't catch up; SC 2.2.2 violation | Pause on hover/focus, give a play/pause control, default to ≥ 8 s per slide |
| Infinite text marquee | Same — and most marquees serve no real content goal | Replace with a static list, or a loop that pauses on focus and respects reduce-motion |
| Parallax that doesn't respect reduced motion or mobile viewport | Motion sickness; on mobile parallax usually hides behind the fold anyway | Disable on `(prefers-reduced-motion: reduce)` and `(max-width: 768px)` |

## P1 — taste / "AI slop" tells

| Anti-pattern | Why | Fix |
|---|---|---|
| Linear easing on UI motion | Reads as "AI default"; nothing in the physical world moves linearly | `cubic-bezier(0.2, 0, 0, 1)` for entrance, `cubic-bezier(0.4, 0, 1, 1)` for exit, `power2.out`/`power3.out` in GSAP |
| Bounce or elastic easing on every interaction | Devalues the flourish; reads as cartoonish | Reserve `back.out(1.7)` / `elastic.out` / `bounce.out` for moments that genuinely call for it (success states, deliberate playful brand) |
| Animating layout properties (`width`, `height`, `top`, `left`, `margin`) | Triggers layout/paint, drops frames, jitters on mobile | Use `transform: translate/scale` and `opacity`. If you need a width animation, animate `transform: scaleX` and counter-scale the contents. |
| Entrance animation on every element on first paint | Death by ten thousand fades; delays LCP | Animate the hero element only on first paint; defer everything else to `IntersectionObserver` triggers below the fold |
| Same entrance pattern on every element in a scene | Reads as templated; loses choreography | Vary easing across at least 3 different curves per scene; vary direction/distance |
| Multi-second fade-in (`duration: 2s`) on UI elements | Feels broken; users don't wait | UI motion stays in 200–600 ms. Reserve > 1 s for hero / scene transitions only |
| `repeat: -1` / `animation-iteration-count: infinite` on UI | Drains battery, hides bugs (the motion masks layout shifts), kills tab-switch heuristics | Compute a finite repeat from a known cycle, gate on `prefers-reduced-motion`, pause on `document.hidden` |
| Two animation libraries in the same component | Bundle bloat + competing render loops | Pick one — see `selection.md` |
| Scroll-driven motion with no end state | Element jitters past the viewport | Define a clear `end` and `toggleActions` (GSAP ScrollTrigger) or animation range (CSS scroll-driven) |
| Hover animation > 250 ms | Pointer leaves before the animation finishes; feels laggy | Hover/focus motion stays in 100–250 ms |

## P2 — refinement

| Anti-pattern | Why | Fix |
|---|---|---|
| `will-change: transform` on always-animating elements | Permanently allocates a layer; defeats the optimization | Add `will-change` only when motion is about to start; remove after |
| Animations that start at `t = 0` | Reads as a hard cut; no anticipation | Offset the first animation by 0.1–0.3 s |
| Decorative motion competing with reading | Background loops keep redrawing while the user reads body copy | Pause decorative motion when reading-zone is in the viewport |
| Stagger > 200 ms between siblings | Feels slow; user reads the list before it finishes appearing | Default stagger 60–120 ms; cap total stagger duration around 0.4–0.6 s |

## P3 — video-mode-specific

When in video mode (HyperFrames), additional anti-patterns:

- **Exit animation before a scene transition.** The transition IS the exit. The outgoing scene must be fully visible at transition start. Only the final scene may fade elements out.
- **`Math.random()` / `Date.now()` in a composition.** The capture engine seeks frames out of order; non-deterministic values produce inconsistent renders.
- **Async timeline construction.** `window.__timelines[id] = tl` must be set synchronously after page load, not inside a Promise.
- **Animating `visibility` or `display`.** GSAP can't tween them meaningfully; use `opacity` and `pointer-events`.
- **Full-screen linear gradients on dark backgrounds.** H.264 banding. Use radial or solid + localized glow.

## How to apply

When `/improve` or `/review` runs and the target involves motion:

1. Walk the P0 list — any hit is a hard fix.
2. Walk the P1 list — list each violation in the report with the fix.
3. P2 / P3 — note in the report only if obvious.

When `/build` emits motion, all P0 and P1 items are constraints. The output should never violate them.
