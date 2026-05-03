# Lottie for production web apps

Lottie plays JSON-encoded vector animations exported from After Effects (Bodymovin plugin) or designed in LottieFiles. The right pick when:

- A designer hands you a `.json` or `.lottie` file.
- The animation is illustrative (mascot, logo reveal, decorative loop) rather than UI choreography.
- The motion is too complex or organic to author with code.

Two players: `lottie-web` (Airbnb's original, ~70 KB gzip, SVG/Canvas/HTML renderers) and `dotLottie` (LottieFiles' modern player, ~30 KB gzip, accepts `.lottie` zip-bundled assets with smaller file size).

## lottie-web pattern

```bash
npm install lottie-web
```

```html
<div id="logo-lottie" class="lottie-layer"></div>
```

```js
import lottie from "lottie-web";

const anim = lottie.loadAnimation({
  container: document.getElementById("logo-lottie"),
  renderer: "svg",       // "canvas" or "html" also available
  loop: false,
  autoplay: true,
  path: "/animations/logo-reveal.json",
});
```

```css
.lottie-layer { width: 100%; height: 100%; }
```

### Renderer choice

- **SVG** (default) — sharpest, most accurate, supports masks and gradients. Use this for hero motion and logos.
- **Canvas** — best performance for many simultaneous animations.
- **HTML** — only when you need DOM elements (rare).

## dotLottie pattern

```bash
npm install @lottiefiles/dotlottie-web
```

```html
<canvas id="product-lottie" class="lottie-canvas"></canvas>
```

```js
import { DotLottie } from "@lottiefiles/dotlottie-web";

const player = new DotLottie({
  canvas: document.getElementById("product-lottie"),
  src: "/animations/product-flow.lottie",
  autoplay: true,
  loop: false,
});
```

```css
.lottie-canvas { width: 100%; height: 100%; display: block; }
```

`.lottie` files are typically 60–80 % smaller than the equivalent `.json` because they bundle and compress assets.

## Reduced motion

Replace the player with a static poster image when the user prefers reduced motion:

```js
const reduce = matchMedia("(prefers-reduced-motion: reduce)").matches;
if (reduce) {
  container.innerHTML = `<img src="/animations/logo-poster.svg" alt="" />`;
} else {
  lottie.loadAnimation({ /* ... */ });
}
```

This is non-negotiable for hero/logo motion. Static poster files should be checked in alongside the `.json`/`.lottie`.

## Controlling playback

```js
anim.play();
anim.pause();
anim.stop();           // jumps to frame 0
anim.goToAndStop(60, true); // frame 60, true = use frames not ms
anim.goToAndPlay(0);
anim.setSpeed(1.5);
anim.setDirection(-1); // reverse
anim.addEventListener("complete", () => { /* ... */ });
```

## Scroll-driven Lottie

Bind `goToAndStop(progress * totalFrames, true)` to scroll progress. Throttle with `requestAnimationFrame` or use a library like Lenis for smooth scroll progress.

```js
const totalFrames = anim.totalFrames;
function onScroll() {
  const progress = window.scrollY / document.documentElement.scrollHeight;
  anim.goToAndStop(Math.min(progress * totalFrames, totalFrames - 1), true);
}
```

For ScrollTrigger users (already-installed GSAP):

```js
ScrollTrigger.create({
  trigger: ".hero",
  start: "top top",
  end: "bottom top",
  scrub: 0.5,
  onUpdate: (self) => anim.goToAndStop(self.progress * anim.totalFrames, true),
});
```

## Good uses

- After Effects exports already validated by the designer.
- Logo reveals, icon loops, decorative accents, success/empty-state illustrations.
- Onboarding sequences where the brand wants signature motion.

## Avoid

- Loading remote `path` URLs that aren't on your CDN — render-time fetch failures kill the animation silently.
- Starting with `autoplay: true` and `loop: true` on hero motion without a stop signal — the animation runs forever and drains battery.
- Assuming every After Effects effect survives export. Test the JSON in a browser before shipping. Common casualties: text layers with custom shapes, expressions, blur effects above a threshold.
- Lazy-loading the Lottie player on first paint when the animation is above-the-fold — preload the player and the JSON.

## References

- lottie-web (Airbnb): https://github.com/airbnb/lottie-web
- `loadAnimation` options: https://github.com/airbnb/lottie-web/wiki/loadAnimation-options
- dotLottie web player: https://developers.lottiefiles.com/docs/dotlottie-player/dotlottie-web/
- LottieFiles editor: https://lottiefiles.com/
