# Anime.js for production web apps

Anime.js v4 is a compact (~7 KB gzip) animation library with concise SVG and DOM motion syntax. **Don't introduce Anime.js into a project that doesn't already use it** — GSAP covers everything Anime.js does, with a larger ecosystem (ScrollTrigger, Flip, Observer). Use this reference when the user already has Anime.js installed or specifically asks for it.

## Install

```bash
npm install animejs@^4
```

```js
import { animate, createTimeline } from "animejs";
```

## Basic pattern

```js
animate(".mark", {
  translateX: 280,
  rotate: "1turn",
  opacity: [0, 1],
  duration: 1200,
  easing: "easeOutExpo",
});
```

- Property arrays `[from, to]` give explicit start and end values.
- Strings like `"1turn"` and `"100%"` are accepted.
- camelCase property names.

## Timeline pattern

```js
const tl = createTimeline({ defaults: { easing: "easeOutCubic" } });

tl.add(".title",   { translateY: [40, 0], opacity: [0, 1], duration: 650 })
  .add(".accent",  { scaleX: [0, 1], duration: 450 }, 250);
```

Position parameter (`250` above) is in milliseconds, absolute or relative.

## Easing

Anime.js v4 names: `easeOut[Sine|Quad|Cubic|Quart|Quint|Expo|Circ|Back|Elastic|Bounce]` and the `In` and `InOut` variants. Custom: `cubicBezier(0.2, 0, 0, 1)`.

For UI motion, default to `easeOutCubic` or `easeOutExpo`. Reserve `easeOutBack` and `easeOutElastic` for moments that genuinely need a flourish.

## Reduced motion

```js
const reduce = matchMedia("(prefers-reduced-motion: reduce)").matches;
if (!reduce) {
  animate(".hero", { translateY: [40, 0], opacity: [0, 1], duration: 700 });
}
```

## Controlling animations

```js
const a = animate(".x", { translateX: 200, duration: 800 });
a.pause();
a.play();
a.reverse();
a.seek(400); // ms
a.complete();
```

## Good uses

- Compact SVG and DOM flourishes where the syntax is shorter than GSAP equivalent.
- Imported Anime.js examples (codepen ports, etc.) when the user wants to keep them as-is.
- Multiple independent micro-animations on the same page.

## Avoid

- Adding Anime.js to a project that already has GSAP. Pick one engine.
- Infinite loops without a pause condition.
- Building animations inside async code, timers, or event handlers without a reduced-motion gate.

## References

- Anime.js v4 docs: https://animejs.com/documentation/
