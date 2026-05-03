# Tailwind v4 in HyperFrames compositions

`hyperframes init --tailwind` scaffolds a project with the Tailwind v4.2 browser runtime pinned to `@tailwindcss/browser@4.2.4`. Treat it as **Tailwind v4**, not v3 — the syntax is meaningfully different.

This is for HyperFrames composition HTML only. For Tailwind in a regular web app, follow the project's own setup (typically v3 with `tailwind.config.js` + PostCSS, or v4 with the CLI — neither uses the browser runtime).

## When to use

- The HF project was created with `init --tailwind`.
- You see `window.__tailwindReady` in `index.html`.
- You want utility classes, CSS-first theme tokens, custom utilities, or v3 → v4 migration guidance.
- The render shows missing styles and the project uses the browser runtime.

## Version contract

- Pinned runtime: `@tailwindcss/browser@4.2.4`.
- The browser runtime script is injected by the CLI. Do **not** replace it with `cdn.tailwindcss.com`.
- HyperFrames waits for `window.__tailwindReady` before frame capture starts.
- The readiness shim must stay deterministic — no render-loop polling, no clock-based retries, no extra runtime fetches.
- For offline / production-stable renders, compile Tailwind to CSS and include the stylesheet directly instead of relying on the browser runtime.

## v4 syntax (CSS-first)

```html
<style type="text/tailwindcss">
  @theme {
    --color-brand: oklch(0.68 0.2 252);
    --font-display: "Inter", sans-serif;
  }

  @utility headline-balance {
    text-wrap: balance;
    letter-spacing: 0;
  }
</style>
```

Then use generated utilities like `text-brand`, `font-display`, `headline-balance`.

## Do not (v3 patterns that don't work in browser-runtime v4)

```css
/* These do nothing in v4 browser-runtime compositions */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Do **not** add a `tailwind.config.js` to define colors, fonts, spacing, or utilities for a v4 browser-runtime composition. Use `@theme` and `@utility` in a `text/tailwindcss` style block.

If you have an existing JS config you must reuse for a compiled v4 build, load it explicitly from CSS with `@config`. Validate in the browser — v4 does not auto-detect v3 config files.

## Source

`.agents/skills/tailwind/SKILL.md` (heygen-com/hyperframes). License: see `NOTICE.md`.
