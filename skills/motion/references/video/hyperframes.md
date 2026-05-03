# HyperFrames — composition authoring

HyperFrames produces video (MP4 / WebM) by seeking an HTML composition frame-by-frame in headless Chrome. The composition is plain HTML + CSS + a paused GSAP timeline. The framework owns the clock; you author the visuals and the timeline.

Use this reference when the user asks for a short video: hero loop, social ad, product promo, intro/outro card. For interactive web-app motion, see `../web-animations/`.

## Project shape

```
my-video/
├── index.html              # root composition (no <template>)
├── compositions/           # sub-compositions (loaded via data-composition-src)
├── assets/                 # images, video, audio
├── fonts/                  # .woff2 files, if custom
└── design.md               # optional brand spec (palette, fonts, motion preset)
```

Scaffold with `npx hyperframes init my-video`. See [`hyperframes-cli.md`](hyperframes-cli.md).

## Data attributes (the timing language)

| Attribute | Required | Values |
|---|---|---|
| `id` | yes | unique |
| `data-start` | yes | seconds, or another clip's id (`"intro + 2"`) |
| `data-duration` | required for `img`/`div`/compositions | seconds |
| `data-track-index` | yes | integer; same-track clips cannot overlap |
| `data-media-start` | no | trim offset into source |
| `data-volume` | no | 0–1 |

`data-track-index` does **not** affect visual layering — use CSS `z-index` for that.

### Composition clips

| Attribute | Required | Values |
|---|---|---|
| `data-composition-id` | yes | unique composition id |
| `data-start` | yes | root composition uses `"0"` |
| `data-duration` | yes | takes precedence over GSAP timeline length |
| `data-width` / `data-height` | yes | pixel dimensions (1920x1080 or 1080x1920 typical) |
| `data-composition-src` | no | path to external HTML file |

## Composition structure

**Standalone composition (root `index.html`)** — `data-composition-id` div directly in `<body>`. Do **not** wrap in `<template>` (that hides it from the browser).

**Sub-composition (loaded via `data-composition-src`)** — wrap in `<template>`:

```html
<template id="my-comp-template">
  <div data-composition-id="my-comp" data-width="1920" data-height="1080">
    <!-- content -->
    <style>
      [data-composition-id="my-comp"] { /* scoped */ }
    </style>
    <script src="https://cdn.jsdelivr.net/npm/gsap@3.14.2/dist/gsap.min.js"></script>
    <script>
      window.__timelines = window.__timelines || {};
      const tl = gsap.timeline({ paused: true });
      tl.from(".title", { y: 48, opacity: 0, duration: 0.6, ease: "power3.out" }, 0);
      window.__timelines["my-comp"] = tl;
    </script>
  </div>
</template>
```

Reference from root:

```html
<div id="el-1"
     data-composition-id="my-comp"
     data-composition-src="compositions/my-comp.html"
     data-start="0" data-duration="10" data-track-index="1"></div>
```

## Timeline contract

- All timelines start `{ paused: true }` — the framework controls playback.
- Register every timeline: `window.__timelines["<composition-id>"] = tl`.
- Duration comes from `data-duration`, not from GSAP timeline length.
- Sub-timelines auto-nest via the framework — do **not** manually `.add(child)`.

## Layout before animation

Position every element where it should be at its **most visible moment** — the frame where it's fully entered, correctly placed, not yet exiting. Build static HTML+CSS for that frame first, **then** add motion with `gsap.from()` to animate from offscreen/invisible into the resolved CSS position.

```css
.scene-content {
  display: flex;
  flex-direction: column;
  justify-content: center;
  width: 100%;
  height: 100%;
  padding: 120px 160px;
  gap: 24px;
  box-sizing: border-box;
}
```

Rules:

- `.scene-content` fills the scene with `width: 100%; height: 100%; padding`. Use padding to push content inward — never `position: absolute; top: Npx` on a content container (overflows when content is taller than the remaining space).
- Reserve `position: absolute` for decoratives.
- Add entrances with `gsap.from(...)` — the CSS position is the ground truth.
- In sub-compositions loaded via `data-composition-src`, prefer `gsap.fromTo(...)` to avoid hydration-ordering surprises.

## Determinism (non-negotiable)

- No `Math.random()`, `Date.now()`, `performance.now()`, or wall-clock logic.
- Use a seeded PRNG (mulberry32) for any pseudo-random values.
- No `async`, `setTimeout`, or Promises around timeline construction. The capture engine reads `window.__timelines` synchronously after page load.
- No `repeat: -1`. Compute repeats from composition duration: `repeat: Math.ceil(duration / cycleDuration) - 1`.

## GSAP-specific rules in video mode

- Animate visual properties only (`opacity`, `x`, `y`, `scale`, `rotation`, color, `borderRadius`, transforms).
- Do **not** animate `visibility` or `display` — toggle by `opacity` and `pointer-events` instead.
- Never call `video.play()` or `audio.play()` — framework owns media playback.
- Never animate the same property on the same element from two timelines simultaneously.

## Scene transitions (multi-scene compositions)

The four non-negotiable rules:

1. **Always use transitions between scenes.** No jump cuts.
2. **Always use entrance animations on every scene.** Every element animates IN via `gsap.from()`. No element appears fully-formed.
3. **Never use exit animations** except on the final scene. The transition IS the exit. The outgoing scene's content must be fully visible at the moment the transition starts.
4. **Final scene only:** the last scene may fade elements out. This is the only scene where `gsap.to(..., { opacity: 0 })` is allowed.

```js
// CORRECT — entrance only, transition handles exit
tl.from("#s1-title",    { y: 50, opacity: 0, duration: 0.7, ease: "power3.out" }, 0.3);
tl.from("#s1-subtitle", { y: 30, opacity: 0, duration: 0.5, ease: "power2.out" }, 0.6);
// transition at 7.2s handles scene change — do NOT add tl.to(..., opacity: 0) here
tl.from("#s2-heading",  { x: -40, opacity: 0, duration: 0.6, ease: "expo.out" }, 8.0);
```

## Animation guardrails

- Offset first animation 0.1–0.3 s (not `t=0`).
- Vary eases across entrance tweens — at least 3 different eases per scene.
- Don't repeat an entrance pattern within a scene.
- Avoid full-screen linear gradients on dark backgrounds (H.264 banding — use radial or solid + localized glow).
- 60 px+ headlines, 20 px+ body, 16 px+ data labels for rendered video.
- `font-variant-numeric: tabular-nums` on number columns.

## Video and audio

```html
<video id="el-v" data-start="0" data-duration="30" data-track-index="0"
       src="video.mp4" muted playsinline></video>
<audio id="el-a" data-start="0" data-duration="30" data-track-index="2"
       src="video.mp4" data-volume="1"></audio>
```

Video must be `muted playsinline`. Audio is always a separate `<audio>` element.

## Library adapters (when not using GSAP)

HyperFrames also seeks:

- **Anime.js** instances pushed to `window.__hfAnime` (set `autoplay: false`).
- **Lottie** players (`lottie-web` or dotLottie) pushed to `window.__hfLottie` (set `autoplay: false, loop: false`).
- **CSS keyframes** with finite `animation-duration` and `animation-iteration-count`, `animation-fill-mode: both`.
- **WAAPI** animations created with `element.animate(...)` and `fill: "both"`, finite `duration` and `iterations`.
- **Three.js** scenes that listen for `hf-seek` and render at `event.detail.time`.

For each, the registration pattern is identical to web-UI mode but with `autoplay: false` and finite repeats.

## Output checklist

Run before declaring done — see [`hyperframes-cli.md`](hyperframes-cli.md):

- [ ] `npx hyperframes lint` passes
- [ ] `npx hyperframes inspect` passes (text overflow, off-canvas elements)
- [ ] WCAG contrast warnings addressed
- [ ] `npx hyperframes render --quality draft` produces a clean MP4
- [ ] Final delivery: `npx hyperframes render --quality high --fps 30` (or 60 for cinematic)

## Source

Original skill: `.agents/skills/hyperframes/SKILL.md` (heygen-com/hyperframes). License: see `NOTICE.md`. The original skill includes deeper references (captions, TTS, audio-reactive, beat-direction, transitions catalog, motion-principles, typography). Read those directly in `.agents/skills/hyperframes/references/` when their topic comes up.
