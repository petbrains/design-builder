# Web Animations API for production web apps

Native browser keyframes with JS-driven timing. Zero bundle cost. Reach for it when CSS keyframes are too rigid and the motion isn't complex enough to justify GSAP.

This reference is for **web-UI mode**. For HyperFrames seek behavior, see [`../video/hyperframes.md`](../video/hyperframes.md).

## Basic pattern

```js
const orb = document.getElementById("orb");
const animation = orb.animate(
  [
    { transform: "translate3d(-160px, 0, 0) scale(0.8)", opacity: 0 },
    { transform: "translate3d(0, 0, 0) scale(1)", opacity: 1, offset: 0.35 },
    { transform: "translate3d(120px, 0, 0) scale(1.08)", opacity: 1 },
  ],
  {
    duration: 3000,
    delay: 200,
    easing: "cubic-bezier(0.2, 0, 0, 1)",
    fill: "both",
    iterations: 1,
  }
);
```

- `fill: "both"` — applies the start state immediately and holds the end state. Without this, the element snaps back to its CSS values once the animation finishes.
- `easing: "cubic-bezier(0.2, 0, 0, 1)"` — the closest Material-style "out" curve. Use this as the default; `"ease"`, `"ease-out"`, `"linear"` are blunt instruments.

## Stagger

```js
document.querySelectorAll(".token").forEach((token, index) => {
  token.animate(
    [
      { transform: "translateY(24px)", opacity: 0 },
      { transform: "translateY(0)", opacity: 1 },
    ],
    {
      duration: 620,
      delay: index * 80,
      easing: "cubic-bezier(0.2, 0, 0, 1)",
      fill: "both",
    }
  );
});
```

## Reduced motion

WAAPI has no built-in `matchMedia` integration. Gate manually:

```js
const reduce = matchMedia("(prefers-reduced-motion: reduce)").matches;
if (reduce) {
  el.style.opacity = 1;
  el.style.transform = "none";
} else {
  el.animate(keyframes, options);
}
```

Wrap in a helper if you do this more than twice.

## Controlling animations

```js
const a = el.animate(keyframes, options);
a.pause();
a.play();
a.reverse();
a.cancel();          // jumps to original CSS state
a.finish();          // jumps to end state
a.currentTime = 500; // ms — seek
a.playbackRate = 0.5;
a.finished.then(() => { /* cleanup */ });
```

`document.getAnimations()` returns every running animation on the document. Useful for global pause (modal opens, route change) but expensive in large apps — prefer keeping a reference array.

## Scroll-driven animations

Modern browsers support `animation-timeline` in CSS and as a WAAPI option:

```js
el.animate(keyframes, {
  duration: 1, // arbitrary; timeline overrides
  fill: "both",
  timeline: new ScrollTimeline({ source: scrollContainer }),
});
```

Feature-detect with `"ScrollTimeline" in window` and provide a no-motion fallback for unsupported browsers.

## Good uses

- Lightweight DOM motion where CSS keyframes are too rigid and GSAP is unnecessary.
- Generated animations from structured data (e.g., per-row stagger driven by API response length).
- Simple sequenced motion that fits inside a single keyframe array with `offset` markers.

## Avoid

- Infinite `iterations` without a stop signal. Browsers throttle background tabs but the object lives forever.
- Mutating render-critical DOM inside `animation.finished.then(...)` — use it for cleanup only.
- Animating layout properties when transforms and opacity express the motion.
- Mixing WAAPI and CSS animations on the same property of the same element — the cascade is well-defined but unintuitive.

## References

- MDN — Using the Web Animations API: https://developer.mozilla.org/docs/Web/API/Web_Animations_API/Using_the_Web_Animations_API
- MDN — `Animation.currentTime`: https://developer.mozilla.org/en-US/docs/Web/API/Animation/currentTime
- CSS Scroll-driven animations spec: https://drafts.csswg.org/scroll-animations-1/
