# CSS animations for production web apps

Zero JS, zero bundle cost, GPU-accelerated when written correctly. The first thing to reach for when motion belongs to one element and has a fixed duration.

## Basic pattern

```css
.pulse-ring {
  animation-name: pulse-ring;
  animation-duration: 1200ms;
  animation-timing-function: cubic-bezier(0.2, 0, 0, 1);
  animation-iteration-count: 3;
  animation-fill-mode: both;
}

@keyframes pulse-ring {
  from { opacity: 0; transform: scale(0.82); }
  35%  { opacity: 1; }
  to   { opacity: 0; transform: scale(1.18); }
}
```

- `animation-fill-mode: both` — applies the `from` state before the animation starts and holds the `to` state after. Without it, the element snaps back to its base CSS values.
- Prefer `cubic-bezier(0.2, 0, 0, 1)` (Material standard) over the named `ease` keywords; the named keywords are blunt.

## Shorthand

```css
.fade-in {
  animation: fade-in 0.6s cubic-bezier(0.2, 0, 0, 1) both;
}
```

Order: `name | duration | timing-function | delay | iteration-count | direction | fill-mode | play-state`. Always include `fill-mode` (use `both`) when the animation should leave the element in its end state.

## Stagger via custom properties

Avoid duplicating keyframes for staggered children:

```html
<div class="dots">
  <span style="--i: 0"></span>
  <span style="--i: 1"></span>
  <span style="--i: 2"></span>
</div>
```

```css
.dots span {
  animation: dot-pop 900ms ease-out both;
  animation-delay: calc(var(--i) * 120ms);
}
```

## Reduced motion (mandatory)

```css
@media (prefers-reduced-motion: reduce) {
  .pulse-ring,
  .fade-in,
  .dots span {
    animation: none;
  }
}
```

Or, for more nuance, gate the *start* of motion:

```css
@media (prefers-reduced-motion: no-preference) {
  .hero { animation: hero-enter 0.8s cubic-bezier(0.2, 0, 0, 1) both; }
}
```

The `no-preference` form is preferred — the default CSS leaves the element in its hero-frame state, and motion only ships when the user has not requested reduce.

## Pausing and play state

```css
.marquee:hover { animation-play-state: paused; }
```

Useful for marquees that should respect cursor focus. Pair with `tabindex="0"` and `:focus-within { animation-play-state: paused }` for keyboard users.

## Good uses

- Decorative loops with a known repeat count.
- Mask, glow, shimmer, grain, and subtle parallax layers.
- Single-element entrances where a full JS timeline would be excessive.
- Hover states (`transition` is usually better than `animation` here, but keyframes shine for multi-step hover sequences).

## Avoid

- Infinite `animation-iteration-count: infinite` without a stop signal. Pair with `animation-play-state` or `prefers-reduced-motion` reduction.
- Animating layout properties (`top`, `left`, `width`, `height`, `margin`, `padding`) when transforms work. These trigger layout and read poorly.
- Relying on hover/focus to trigger render-critical motion (a screenshot test won't catch hover state regressions).
- Changing animation classes mid-animation unless you understand `animation-play-state` and `@starting-style`.

## CSS scroll-driven animations (modern browsers)

Where supported, `animation-timeline: scroll()` drives a CSS animation by scroll position with no JS:

```css
.parallax {
  animation: parallax linear;
  animation-timeline: scroll(root);
}

@keyframes parallax {
  to { transform: translateY(-200px); }
}
```

Feature-detect with `@supports (animation-timeline: scroll())`. Provide a no-motion or reduced fallback for unsupported browsers.

## References

- MDN — `animation` shorthand: https://developer.mozilla.org/en-US/docs/Web/CSS/animation
- MDN — `animation-fill-mode`: https://developer.mozilla.org/en-US/docs/Web/CSS/animation-fill-mode
- MDN — `prefers-reduced-motion`: https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion
- CSS scroll-driven animations: https://developer.chrome.com/articles/scroll-driven-animations/
