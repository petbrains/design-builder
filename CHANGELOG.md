# Changelog

## v2.3.1 — 2026-05-03

### Changed
- `README.md` — added a "Motion — animation knowledge & short-video" section (web-UI mode + video mode summary, anti-patterns link); added a "Live web capture — `playwright-cli`" section documenting `/review` URL input and `/improve --restructure` live inspiration_pages capture; commands table updated to mention `--restructure` and HTTP(S) URL input; `design/` folder tree comments updated for auto-capture paths; "What's inside" lists `skills/motion/`, `skills/playwright-cli/`, `skills/figma-use/`.
- `.claude-plugin/plugin.json` — version 2.3.1.

## v2.3.0 — 2026-05-03

### Added
- **`skills/playwright-cli/`** — vendored browser-automation skill (`@playwright/cli`, Apache-2.0, Microsoft). Used by `/review` and `/improve --restructure` to ground designs in real rendered pixels. Runtime: `npx @playwright/cli` — no global install required. References cover navigation, snapshots, screenshots, DOM inspection, request mocking, tracing, video recording, spec-driven testing.
- **`/design-builder:review` accepts HTTP(S) URLs.** Pass a single URL or a comma-separated list; the auditor invokes `playwright-cli` to capture each at desktop (1440×900) + mobile (375×812) viewports into `design/screenshots/<slug>.<viewport>.png`, then runs visual mode. Replaces the v2.0 "future hook" stub.
- **`/design-builder:improve --restructure` captures live inspiration_pages.** When a chosen anchor has a `url` field, the live page is captured to `design/.cache/inspiration/<page_id>.png` (gitignored, reused on subsequent runs) so the rebuild plan can reference real composition rather than only metadata.
- Source attribution in `NOTICE.md` for vendored knowledge from `@playwright/cli` (Microsoft, Apache 2.0).

### Changed
- `agents/design-auditor.md` — added `urls` and `viewports` inputs; new "Visual acquisition" section runs before visual-mode dimensions when `urls` is supplied; code-only caveat now mentions the `urls` escape hatch.
- `commands/review.md` — Phase 1 detects HTTP(S) URLs; Phase 2 forwards `urls` and `viewports` to the auditor; "Future hooks" replaced with present-tense "Capture mechanisms" section.
- `commands/improve.md` — `--restructure` step 2 documents the optional live-capture pattern.
- `skills/design/SKILL.md` — KB-EXTENSION block now points to playwright-cli as a *visual augmenter* for Layer 1 sources whose records carry a `url` field (today: inspiration_pages).
- `.claude-plugin/plugin.json` — version 2.3.0, description mentions the new skill.
- `CLAUDE.md` — refreshed: 6 commands (was stale at "4"), all four knowledge skills listed, `/review` URL input documented.

### Removed
- `playwright-cli-main/` source folder — deleted after vendoring. Wasn't checked in to git (untracked working tree); SKILL.md + references now live at `skills/playwright-cli/`.

### Notes
- `playwright-cli` skill is `user-invocable: false` — invoked by the auditor and `/improve --restructure`, not via slash command.
- First call to `npx @playwright/cli` triggers a one-time download. If the user opts out (offline / no Node), capture steps fall back gracefully and the auditor surfaces a Caveats line.

## v2.2.0 — 2026-05-03

### Added
- **`skills/motion/`** — new sister skill alongside `skills/design/`. Two modes:
  - **Web-UI mode** — production-app animation knowledge per library: GSAP, WAAPI, CSS, Anime.js, Lottie, Three.js. Includes a selection decision matrix (`references/selection.md`), `prefers-reduced-motion` patterns for each library, and bundle-cost guidance.
  - **Video mode** — short-video generation via the HyperFrames pipeline (HTML compositions seeked frame-by-frame, FFmpeg encode). Composition contract + CLI commands (`init / lint / inspect / preview / render / transcribe / tts`) + Tailwind v4 browser runtime notes.
- **`skills/motion/references/anti-patterns.md`** — P0–P3 motion checklist consumed by the Anti-Patterns Layer 2 filter; integrated into `/improve` Phase 2 step 5 and into `agents/design-auditor.md` Motion dimension.
- Source attribution in `NOTICE.md` for vendored knowledge from `heygen-com/hyperframes` (lockfile at `skills-lock.json`).

### Changed
- `skills/design/SKILL.md` — Layer 1 KB-EXTENSION block points to the motion sub-source for `type='animation'`. Frontend Aesthetics → Motion + Anti-Patterns sections cross-link to the new motion skill.
- `commands/build.md` Phase 3.2 — hand-rolled animations now route through `skills/motion/references/selection.md` and the matching library reference; every emit must respect P0/P1 anti-patterns.
- `agents/design-auditor.md` — Motion dimension cites the new motion anti-patterns checklist.
- `.claude-plugin/plugin.json` — version 2.2.0, description updated.

### Notes
- `motion` skill is `user-invocable: false` (loaded by `design` and the relevant commands, not via slash command).
- The HyperFrames knowledge is vendored at `.agents/skills/` and tracked by `skills-lock.json`. The `references/video/` files in this skill are adapted excerpts; the full original references (captions, TTS, audio-reactive, beat-direction, transitions) remain at `.agents/skills/hyperframes/references/` and are read on demand when their topic comes up.

## v2.1.1 — 2026-04-30

### Changed
- `.claude-plugin/plugin.json` — author simplified to `{ "name": "Petbrains" }` (no personal name/email).

### Fixed
- `/setup` no longer writes `design/interview.md` (was still happening in v2.1.0 despite changelog claim — `commands/setup.md` Phase 1 still instructed Claude to write the file). Phase 1 now keeps interview answers in memory until Phase 5e persists them to `design/.cache/interview.json`.
- Stale `design/interview.md` references replaced with `design/.cache/interview.json` across `CLAUDE.md`, `README.md`, `skills/design/SKILL.md`, `skills/design/references/layer1-resolvers.md`, `skills/design/references/system/web-pipeline.md`.
- `README.md` rewritten for v2.1: 6 commands instead of 4, spec-first explanation, `/design_page`/`/design_screen`/`/build` flow, "Project output" section updated to reflect v2.1 file layout (three foundation files, `pages/`, `screens/`, `.cache/`).

## v2.1.0 — 2026-04-29

### Added
- **`/setup` Phase 0** — scaffolds `design/` folder structure (`pages/`, `screens/`, `references/`, `.cache/`) before any interview question. Detects project stack (Next.js / Vite / SwiftUI / Flutter / static HTML, plus Tailwind / shadcn variants), confirms with the user, suggests scaffold command if no project is initialized.
- **`/setup` Phase 5 emits three foundation files** instead of one:
  - `design/design-system.md` (tokens + Stack section)
  - `design/style-guide.md` (a11y values with numerical contrast computed via `compute_contrast.py`, touch targets, platform constraints, density/motion rules, full anti-pattern prose, component states required)
  - `design/content-library.md` (voice & tone, UI states, forms, notifications)
- **`/design_page <name>`** — new command. Produces a design spec (markdown) for a web page at `design/pages/<name>.md`. Documentation-aware (asks for PRD/brief/Figma URL up front, doesn't paraphrase). Animation choice fixed in spec; `/build` vendors the file later. Final gate offers Figma export, another spec, or `/build`.
- **`/design_screen <name>`** — new command. Same as `/design_page` for app screens. Uses `landing_patterns` + iOS HIG (inspiration_pages is web-only). Spec adds Navigation context, Gestures, Safe areas. Animation specs are HIG springs (catalog is React-only).
- **`/build [target]`** — new universal code generator. Reads spec(s) from `design/pages/` and/or `design/screens/`, writes code to source tree. Supports single (`build landing`), batch (`build all`), glob (`build pages/onboarding-*`), or interactive multi-select (`build` with no arg). Sequential, continue-on-fail. Stack-aware path conventions.
- `compute_contrast.py` — stdlib WCAG 2.1 contrast calculator. CLI: stdin JSON → stdout JSON.
- New references: `style-guide-template.md`, `content-library-template.md`, `page-spec-format.md`, `screen-spec-format.md`.

### Changed
- `/setup` no longer writes `design/interview.md` — captures live in `design/.cache/interview.json` (debug context, gitignored, not for users).
- `/improve`, `/review`, `agents/design-auditor.md` — read the three new foundation files; a11y findings cross-reference `style-guide.md` contrast table; voice findings cross-reference `content-library.md` principles.
- `skills/design/SKILL.md` — command map updated to 6 commands; Layer 1 source order extended (spec is primary Layer 1 source for `/build`).
- `references/architecture.md` — three-layer rule documents the spec as intermediate artefact between `/design_page` (synthesis) and `/build` (emit).
- `references/commands.md` — rewritten for the v2.1 command set.
- `.claude-plugin/plugin.json` — version 2.1.0, description updated.
- `design/references/` no longer has a `downloaded/` subfolder; reference images go directly into `design/references/` with `ref-` filename prefix for auto-downloads.

### Removed
- **`/create` command** — deleted. Hard cut from v2.0. The "design + emit code in one step" flow conflated planning and execution; v2.1 separates them with `/design_page` / `/design_screen` (planning) + `/build` (execution). For new v2.1 projects, run `/setup` then `/design_page <name>` then `/build <name>`.
- `design/interview.md` (was a v2.0 output) — data now lives in `design/.cache/interview.json`.

### Migration

No `--migrate` flag, no automated migration tooling. v2.1 is a hard cut for the project author (single user at this point). For older projects pinned to v2.0, stay on commit `4d331cc` until manually re-running `/setup` on v2.1.

## 2.0.0 — 2026-04-27

**Full rebrand to `design-builder`. Command-based architecture replaces pipeline/atomic model.**

### Added
- 4 user-facing commands: `/design-builder:setup` (alias `/start`), `/design-builder:create`, `/design-builder:improve`, `/design-builder:review`.
- `inspiration_pages` as the primary Layer 1 source for whole-page references (405 records via designlib MCP).
- Typed `get_design_reference(type, filters)` resolver in `skills/design/references/layer1-resolvers.md`.
- Standard project output folder `design/` with documented structure (system.md, tokens.css, interview.md, preview.html, references/, screenshots/, reviews/).
- AI-slop detector and code-only mode in `agents/design-auditor.md`.
- Mandatory `Next:` block at the end of every command output.

### Changed
- `agents/design-auditor.md` audit criteria expanded; output adds `mode: visual | code_only | mixed`.
- `skills/design/SKILL.md` slimmed: pipeline/atomic descriptions removed, knowledge base preserved.
- `skills/design/references/architecture.md` rewritten for command-based model.
- README and CLAUDE.md fully rewritten.

### Removed
- 22+ atomic commands (`/design system`, `/design craft`, `/design audit`, ...) — replaced by the 4 new commands.
- 5 lifecycle pipelines (`start`, `make`, `refine`, `review`, `ship`) — pipeline orchestration is gone.
- `.cursor-plugin/` — Cursor support dropped (may return in v2.x).
- `skills/design/references/pipelines.md`.

### Migration from 1.2
Hard cut, no aliases. Users on 1.2 should pin to commit `f43fdcc` (last v1.2 commit) until they migrate.

## 1.2.0 and earlier
See git history.
