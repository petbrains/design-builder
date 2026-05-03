# Picking the right motion library

A decision tree for the `motion` skill. Use this before reaching for a library reference.

## Step 1 — what is the motion *for*?

| Motion type | Best fit |
|---|---|
| Single element entrance, hover, micro-interaction | **CSS** (no JS) |
| Stagger across many elements, complex sequencing | **GSAP** |
| Native browser keyframes, want JS-controlled timing without a library | **WAAPI** |
| Designer hand-off (After Effects export) | **Lottie** |
| 3D, particles, shaders, depth | **Three.js** |
| Compact existing Anime.js code | **Anime.js** (don't introduce it; if user already has it, keep it) |

## Step 2 — bundle-cost check (web-UI mode only)

| Library | Min size (gzip) | When the cost is worth it |
|---|---|---|
| CSS animations | 0 KB | Always — no cost |
| WAAPI | 0 KB | Always — native browser |
| Anime.js v4 | ~7 KB | Already in the project, or one-off SVG flourish |
| GSAP core | ~24 KB | Multi-element choreography, scene sequencing |
| Lottie web (svg) | ~70 KB | When the asset is hand-crafted in After Effects and replacing it with code is a downgrade |
| dotLottie | ~30 KB | Smaller `.lottie` assets, modern player |
| Three.js | ~150 KB | Genuine 3D / WebGL need; a CSS pseudo-3D is almost always lighter |

**Rule:** if CSS or WAAPI can do it, do not reach for GSAP. If GSAP is already in the bundle, keep using it for new motion in the same area to avoid a second engine.

## Step 3 — what other libraries are already in the project?

Check the user's `package.json` (or equivalent) before recommending. If GSAP is already installed, do not propose Anime.js. If `framer-motion` is already in a React project, prefer it over GSAP for in-component motion (note: this skill does not currently document framer-motion in depth; recommend native React patterns and link to framer's docs).

## Step 4 — accessibility gate

Every recommendation must include a `prefers-reduced-motion` story:

- **CSS** — wrap motion in `@media (prefers-reduced-motion: no-preference) { ... }` or override inside `@media (prefers-reduced-motion: reduce) { animation: none; }`.
- **GSAP** — `gsap.matchMedia({ reduceMotion: "(prefers-reduced-motion: reduce)" }, ...)`.
- **WAAPI** — `if (matchMedia("(prefers-reduced-motion: reduce)").matches) return;` before `element.animate()`.
- **Anime.js / Lottie / Three.js** — same JS guard pattern; for Lottie/Three, swap to a static poster image instead of the animation.

## Step 5 — performance gate

- Animate `transform` and `opacity` only. Document any exception in code with a one-line comment naming the property and why.
- `will-change: transform` only on elements that actually animate, and remove it after.
- For continuous frequent updates (mouse-driven, scroll-driven), use `gsap.quickTo()` or a manually rAF-throttled WAAPI handle.

## Common requests → library map

| Request | Pick |
|---|---|
| "Fade in the hero on scroll" | CSS + `IntersectionObserver`, or GSAP if already in bundle |
| "Animate a list of cards in sequence" | GSAP `stagger`, or WAAPI loop with `delay: index * N` |
| "Logo reveal at the top of the page" | Lottie if designer provides JSON; else CSS |
| "Animated number counter" | WAAPI on `--counter` CSS variable, or GSAP |
| "Hero with floating 3D product" | Three.js, but recommend a video-poster fallback under 4G/slow networks |
| "Subtle background shimmer / grain" | CSS keyframes, finite repeat |
| "Page transition between routes" | View Transitions API where supported, GSAP fallback |
| "Hand-drawn highlight under a word" | CSS + SVG path with `stroke-dasharray` keyframe |
| "Scroll-driven parallax" | CSS scroll-driven animations (`animation-timeline: scroll()`) where supported, GSAP ScrollTrigger fallback. Always reduce-motion gate. |

## When to switch to video mode

Switch from web-UI mode to video mode (HyperFrames pipeline) when:

- The deliverable is a file (MP4, WebM, GIF), not a live interactive page.
- The user names a duration in seconds AND a delivery target (Instagram, Twitter, YouTube, ad spec).
- The motion is intended as a hero loop or product promo embedded as `<video>` rather than rendered live.

Do NOT switch to video mode for: hover states, scroll triggers, page transitions, in-app animation. Those are always web-UI even if the user describes them visually.
