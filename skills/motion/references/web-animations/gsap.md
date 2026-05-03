# GSAP for production web apps

GSAP is the default for non-trivial UI motion: stagger, multi-element choreography, scroll-driven motion, and route transitions. ~24 KB gzip for the core; add ScrollTrigger / Flip / Observer plugins as needed.

This reference is for **web-UI mode**. For the HyperFrames video adapter (timeline registration on `window.__timelines`, no `tl.play()` for render-critical motion), see [`../video/hyperframes.md`](../video/hyperframes.md).

## Install

```bash
npm install gsap
```

```js
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
gsap.registerPlugin(ScrollTrigger);
```

## Core tween methods

- **`gsap.to(targets, vars)`** — animate from current state to `vars`. Most common.
- **`gsap.from(targets, vars)`** — animate from `vars` to current state. Use for entrances; the CSS position is the ground truth, the tween describes the journey to get there.
- **`gsap.fromTo(targets, fromVars, toVars)`** — explicit start and end. Use when you can't trust the resolved current state (component remounts, hydration boundaries).
- **`gsap.set(targets, vars)`** — apply immediately (duration 0).

Always use **camelCase** property names (`backgroundColor`, `rotationX`).

## Common vars

- `duration` — seconds (default 0.5).
- `delay` — seconds before start.
- `ease` — `"power1.out"` (default), `"power3.inOut"`, `"back.out(1.7)"`, `"elastic.out(1, 0.3)"`, `"none"`.
- `stagger` — number `0.1` or object: `{ amount: 0.3, from: "center" }`, `{ each: 0.1, from: "random" }`.
- `overwrite` — `false` (default), `true`, or `"auto"`.
- `repeat`, `yoyo` — for ambient loops, prefer a finite number with a defined exit. Avoid `-1` in production UI; if used, gate on `prefers-reduced-motion` and provide a stop signal (visibility, route change).
- `onComplete`, `onStart`, `onUpdate` — callbacks.

## Transform aliases (prefer over raw `transform` strings)

| GSAP property | Equivalent |
|---|---|
| `x`, `y`, `z` | translateX/Y/Z (px) |
| `xPercent`, `yPercent` | translateX/Y in % |
| `scale`, `scaleX`, `scaleY` | scale |
| `rotation` | rotate (deg) |
| `rotationX`, `rotationY` | 3D rotate |
| `skewX`, `skewY` | skew |
| `transformOrigin` | transform-origin |

- `autoAlpha` — prefer over `opacity`. At 0 also sets `visibility: hidden` (saves layout passes and accessibility).
- CSS variables: `"--hue": 180`.
- `clearProps: "all"` — removes inline styles on complete. Useful when you want CSS to take over after the tween.
- Relative values: `"+=20"`, `"-=10"`, `"*=2"`.

## Easing

Built-ins: `power1`–`power4`, `back`, `bounce`, `circ`, `elastic`, `expo`, `sine`. Each has `.in`, `.out`, `.inOut`.

**For UI motion, default to `.out` curves.** `.in` curves feel sluggish (slow start). `.inOut` is for elements that travel a long distance. `power2.out` and `power3.out` cover ~80% of UI cases. Reserve `back.out(1.7)`, `elastic.out`, `bounce.out` for moments that genuinely call for a flourish — overuse is the #1 motion anti-pattern.

## Defaults

```js
gsap.defaults({ duration: 0.6, ease: "power2.out" });
```

Set defaults at app boot once; don't repeat them on every tween.

## Timelines

```js
const tl = gsap.timeline({ defaults: { duration: 0.5, ease: "power2.out" } });
tl.from(".title", { y: 48, opacity: 0 }, 0)
  .from(".subtitle", { y: 32, opacity: 0 }, 0.15)
  .from(".cta", { y: 24, opacity: 0 }, 0.3);
```

### Position parameter

Third argument controls placement:

- Absolute: `1` — at 1s
- Relative: `"+=0.5"` — after end; `"-=0.2"` — before end
- Label: `"intro"`, `"intro+=0.3"`
- Alignment: `"<"` — same start as previous; `">"` — after previous ends; `"<0.2"` — 0.2s after previous starts

```js
tl.to(".a", { x: 100 }, 0);
tl.to(".b", { y: 50 }, "<");        // same start as .a
tl.to(".c", { opacity: 0 }, "<0.2"); // 0.2s after .b starts
```

### Labels

```js
tl.addLabel("intro", 0)
  .to(".a", { x: 100 }, "intro")
  .addLabel("outro", "+=0.5");

tl.tweenFromTo("intro", "outro");
```

### Playback

`tl.play()`, `tl.pause()`, `tl.reverse()`, `tl.restart()`, `tl.time(2)`, `tl.progress(0.5)`, `tl.kill()`.

## `gsap.matchMedia()` — responsive + accessibility

This is the **required** wrapper for any non-trivial motion. It runs setup only when a media query matches and auto-reverts when it stops matching.

```js
const mm = gsap.matchMedia();
mm.add(
  {
    isDesktop: "(min-width: 800px)",
    reduceMotion: "(prefers-reduced-motion: reduce)",
  },
  (context) => {
    const { isDesktop, reduceMotion } = context.conditions;

    if (reduceMotion) {
      gsap.set(".hero", { opacity: 1, y: 0 });
      return;
    }

    gsap.from(".hero", {
      y: isDesktop ? 60 : 30,
      opacity: 0,
      duration: 0.7,
      ease: "power3.out",
    });
  }
);

// Cleanup when the component unmounts:
mm.revert();
```

In React: store the `mm` in a ref and call `revert()` in the `useEffect` cleanup.

## `gsap.quickTo()` — frequent updates

For mouse-driven, scroll-driven, or sensor-driven motion that runs many times per second:

```js
const xTo = gsap.quickTo("#cursor", "x", { duration: 0.4, ease: "power3" });
const yTo = gsap.quickTo("#cursor", "y", { duration: 0.4, ease: "power3" });

window.addEventListener("mousemove", (e) => {
  xTo(e.pageX);
  yTo(e.pageY);
});
```

`quickTo` short-circuits the tween-construction overhead. Don't use raw `gsap.to` in a `mousemove` handler.

## ScrollTrigger essentials

```js
gsap.from(".reveal", {
  y: 60,
  opacity: 0,
  duration: 0.8,
  ease: "power3.out",
  scrollTrigger: {
    trigger: ".reveal",
    start: "top 80%",
    end: "top 30%",
    toggleActions: "play none none reverse",
  },
});
```

- `toggleActions: "play none none reverse"` plays on enter, reverses on leave-back. The default `"play none none none"` stays played.
- `scrub: true` ties progress to scroll position. Use `scrub: 0.5` for smoothing.
- Always `ScrollTrigger.refresh()` after layout changes (route swap, image load, font swap).
- Reduce-motion gate via `gsap.matchMedia()` — disable scrub for `reduceMotion`.

## Performance

- Animate `transform` and `opacity`. Avoid `width`, `height`, `top`, `left`, `margin` when transforms achieve the same effect.
- `will-change: transform` only on elements that actually animate; remove after.
- Use `stagger` instead of N tweens with manual delays.
- Pause or `kill()` off-screen animations.

## Do not

- Animate layout properties when transforms suffice.
- Use both `svgOrigin` and `transformOrigin` on the same SVG element.
- Chain animations with `delay` when a timeline can sequence them.
- Create tweens before the DOM exists.
- Skip cleanup — always `kill()` or use `gsap.context()` / `matchMedia` for component-scoped lifecycle.
- Use `repeat: -1` without a `prefers-reduced-motion` and visibility-change pause.

## References

- GSAP docs: https://gsap.com/docs/v3/
- ScrollTrigger: https://gsap.com/docs/v3/Plugins/ScrollTrigger/
- `matchMedia`: https://gsap.com/docs/v3/GSAP/gsap.matchMedia()/
