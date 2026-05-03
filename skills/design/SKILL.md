---
name: design
description: "Knowledge base and rules for the design-builder plugin. Three-layer model (commands -> filters -> knowledge base) for building distinctive production-grade interfaces on web + iOS. Backed by designlib MCP (palettes, fonts, inspiration_pages, landing_patterns, icons, animations), iOS HIG references (iOS 18/26 Liquid Glass), designer motion perspectives (Emil/Jakub/Jhey), anti-pattern detection. This skill is the source of truth — actions live in the plugin's commands/."
user-invocable: false
---

# Design Knowledge Base — for design-builder

You are the knowledge base behind the `design-builder` plugin. Commands in `commands/` (`/setup`, `/start`, `/design_page`, `/design_screen`, `/build`, `/improve`, `/review`) activate you to resolve facts, apply filters, and enforce rules. **You do not execute commands directly.** Wait until a command activates you.

The two failure modes this skill exists to prevent — read first.

## Failure mode 1 — skipping discovery because "context looks complete"

You are a fluent text generator. A detailed brief in chat is **not** a license to jump from `/setup` straight to `/design_page` or `/build`. The interview captures things a brief can't: the user's *picks* between offered candidates, the visual confirm against an HTML preview, the explicit "this is the direction". Skipping it produces output the user had no say in.

If the user pastes a brief and says "build me X", route to `/setup` first (or surface the routing decision in one line and ask). Don't silently jump.

## Failure mode 2 — describing variants in prose instead of rendering

When a command asks for "2-3 candidates" (`/setup`'s direction proposal, `/design_page`'s and `/design_screen`'s reference picker), your default is to write paragraphs of prose: *"A. Editorial — light, warm. B. Forest-block — dark, confident…"* **Don't.**

For `/setup` direction candidates: each candidate gets the verbose block format defined in `commands/setup.md` Phase 3. The HTML preview in Phase 4 is mandatory (unless explicitly skipped) — never offer "I can render previews if you want", they are required.

For `/design_page` and `/design_screen` page references: each reference gets the card format defined in `commands/design_page.md` Phase 3.

If you find yourself drafting a generic A/B/C narrative — stop. Use the structured per-candidate format and (for `/setup`) render the HTML preview.

---

## Architecture — three layers

```
┌──────────────────────────────────────────────────────────────────┐
│  COMMANDS    (commands/setup,start,design_page,design_screen,build,improve,review.md)  │
│  Entry points the user invokes. Thin scenarios.                   │
└────────────────────────────┬─────────────────────────────────────┘
                             │ activate this skill, which:
                             ▼
┌──────────────────────────────────────────────────────────────────┐
│  Layer 2: DESIGN FILTERS  (gating — what we will not emit)        │
│  Direction · Dials · Aesthetics · Anti-Patterns ·                 │
│  Distinctiveness Gate · Output Rules                              │
│  ALL SIX always run before emit. Silent skip = AI slop.           │
└────────────────────────────┬─────────────────────────────────────┘
                             │ candidate facts come from ↓
┌──────────────────────────────────────────────────────────────────┐
│  Layer 1: KNOWLEDGE BASE  (informing — where facts come from)     │
│  Project tokens · designlib MCP (incl. inspiration_pages) ·       │
│  local CSV · iOS HIG · free generation                            │
│  Resolved via get_design_reference(type, filters)                 │
└──────────────────────────────────────────────────────────────────┘
```

**THE RULE:** every command emits design output by (1) resolving facts through Layer 1 in the fixed source order, (2) gating output through all six Layer 2 filters, in that order. Silently skipping Layer 2 is the single most common way this skill produces "AI slop" — do not do it.

Architecture detail + extension points: [`references/architecture.md`](references/architecture.md).

## Agent delegation

One sub-agent lives in `agents/`:

| Trigger             | Sub-agent        | Why this one is an agent                                                |
|---------------------|------------------|-------------------------------------------------------------------------|
| `/review`           | `design-auditor` | Loads many references (a11y, perf, HIG, motion, AI-slop criteria), greps the project, returns a structured P0-P3 report. Heavy multi-file reads + structured JSON output — both qualities of a useful agent. |

`/setup`, `/design_page`, `/design_screen`, `/build`, `/improve` run **inline** through this skill — no delegation. The earlier v1.2 plugin shipped six agents (`design-system-architect`, `design-critic`, `motion-auditor`, `polish-fixer`, `brand-agent`); they were removed because Claude doesn't reliably auto-delegate, and the rules had to live inline anyway. Only the auditor survives.

---

# Layer 1: Knowledge Base — where facts come from

All Layer 1 access goes through `get_design_reference(type, filters)`. See [`references/layer1-resolvers.md`](references/layer1-resolvers.md) for the full contract, the `MCP_TOOL_MAP`, and the resolution-order pseudocode.

## Source order (strict)

1. **Project tokens** — `design/tokens.css`, `design/design-system.md`, `design/style-guide.md`, `design/content-library.md`, `tailwind.config.*`, `*.xcassets`. Read when a command wants "use what the user has".
1.5. **Spec files (only for /build)** — `design/pages/<name>.md`, `design/screens/<name>.md`. /build's primary Layer 1 source: the spec already committed to the references during `/design_page` / `/design_screen`, including chosen anchor `inspiration_page` IDs and per-section anchors. /build does NOT re-resolve references; it honors what the spec captured. (For /design_page, /design_screen, /setup, /improve, /review — sources 1, 2, 3, 4, 5 apply as before, no spec source.)
2. **designlib MCP** — primary source for most types. New in v2.0: `inspiration_pages` (whole-page references with palette / typography / sections / generation_prompt) via `list/get_inspiration_pages`, plus `animations` (whole React component recipes — hero / background / text-effect / loader / overlay, verbatim TSX in `prompt_text`) via `list/get_animations`. Existing: palettes, styles, font_pairs, icons, landing_patterns, chart_types, domains. **Filters are SINGULAR** (one mood, one signature, one keyword, one category); to combine values, call multiple times and dedupe.
3. **Local CSV** — [`scripts/search.py`](scripts/search.py) over `data/`. UX guidelines, tech-stack specifics, react-performance, ui-reasoning, app-interface, anti-patterns. (Charts, landing_patterns, icons → designlib MCP, no longer local.)
4. **iOS HIG references** — [`references/ios/`](references/ios/README.md). Apple-specific rules. Use when filters specify `platform='ios'`.
5. **Free generation** — last resort. Mark `source='free'` on the result.

If `designlib` MCP is NOT connected and a user starts a design system from scratch, tell them once:
> "designlib MCP is not connected. It gives authoritative palettes/fonts/inspiration_pages for web + iOS. Install: `claude mcp add --transport http designlib https://designlib-production.up.railway.app/mcp`. Proceeding with local CSV fallback."

### Motion sub-source (KB-EXTENSION)

When the resolved type is `animation` or the spec includes an `animation:` block, the `designlib` MCP gives **what** to do (the recipe / inspiration); the **`motion` sister skill** at [`../motion/SKILL.md`](../../motion/SKILL.md) gives **how** to implement it (library choice, registration patterns, easing, performance, accessibility). Always consult both — the MCP picks the candidate, the motion skill writes correct code.

The motion skill also owns **video mode** (HyperFrames) when the user asks for an MP4 / WebM / short-video deliverable rather than UI motion.

### Live web reference capture (visual augmenter, KB-EXTENSION)

Not a Layer 1 source on its own — a *visual augmenter* for sources that already return a record with a `url` field (today: `inspiration_pages.url`; future: any whole-page reference). The canonical fact still comes from designlib MCP; `playwright-cli` ([`../playwright-cli/SKILL.md`](../../playwright-cli/SKILL.md)) just adds rendered pixels so commands can reason about composition, not only metadata.

Used by `/improve --restructure` (caches inspiration_page captures to `design/.cache/inspiration/<page_id>.png`) and `/review` (captures user-supplied URLs to `design/screenshots/<slug>.<viewport>.png` for the auditor). Skip silently if `playwright-cli` is unavailable — never block a rebuild on it.

<!-- KB-EXTENSION: add new source here -->

## Schema map for `inspiration_pages`

[`references/inspiration_pages.md`](references/inspiration_pages.md) — vocab, filter triad, field semantics, known issues. Load whenever a command resolves `type='page'`. Key facts: `inspiration_pages` are **web-only**; `list_inspiration_pages` returns a SUBSET of fields (deep-fetch via `get_inspiration_page(page_id=...)` for palette/typography/sections/generation_prompt).

## Platform router

Primary platforms: `web` · `ios` · `cross`.

Inference: SwiftUI/UIKit code present → `ios`; `tailwind.config` / `package.json` with React/Next → `web`; both → ask the user.

iOS output: xcassets, SwiftUI Color extensions, Dynamic Type, `.sensoryFeedback`, Liquid Glass on iOS 26.
Web output: CSS custom properties + Tailwind/shadcn.

iOS deep references: [`references/ios/`](references/ios/README.md) — 18 files. Load on demand.

## Context gathering

Required minimum: target audience, top 3 use cases, brand personality (3 concrete words), platform.

Where to find: `design/.cache/interview.json` (written by `/setup`), or by reading the project's brief/PRD.

**Code-only inference is insufficient.** Code says what was built; not who it's for or how it should feel. If a command runs without `design/.cache/interview.json` AND without project tokens AND without an explicit user-supplied brief, the command must stop and ask — never invent context.

## Reference index

### Token sources
| Path | Purpose |
|---|---|
| [`references/designlib-mcp.md`](references/designlib-mcp.md) | designlib MCP guide |
| [`references/inspiration_pages.md`](references/inspiration_pages.md) | inspiration_pages schema map (vocab, filters, field semantics) |
| [`references/animations.md`](references/animations.md) | animations schema map (vocab, filters, field semantics, mood→style_tag map) |
| [`references/layer1-resolvers.md`](references/layer1-resolvers.md) | `get_design_reference()` contract + MCP_TOOL_MAP |
| [`references/system/web-pipeline.md`](references/system/web-pipeline.md) | Web token output (tokens.css, Tailwind, shadcn) |
| [`references/system/ios-pipeline.md`](references/system/ios-pipeline.md) | iOS token output (xcassets, SwiftUI theme files) |

### Platform-agnostic
| Path | Purpose |
|---|---|
| [`references/architecture.md`](references/architecture.md) | Three-layer model + extension points |
| [`references/design-dials.md`](references/design-dials.md) | Design Dials detail (VARIANCE / MOTION / DENSITY) |
| [`references/distinctiveness-gate.md`](references/distinctiveness-gate.md) | Pre-emit gate; HARD on `/setup`, SOFT on `/design_page` and `/design_screen`, evaluator inside `/review` |
| [`references/ux-writing.md`](references/ux-writing.md) | General UX writing principles |

### Web
| Path | Purpose |
|---|---|
| [`references/web/`](references/web/README.md) | Typography, color, spatial, motion, interaction, responsive, style presets, craft/extract/redesign workflows |
| [`references/web/motion/`](references/web/motion/README.md) | Designer perspectives (Emil/Jakub/Jhey), audit checklist, motion gaps, enter/exit recipes |
| [`references/ui-styling/`](references/ui-styling/) | shadcn/ui components, Tailwind utilities, theming |
| [`references/design-system/`](references/design-system/) | Token architecture (primitive→semantic→component) |

### iOS
| Path | Purpose |
|---|---|
| [`references/ios/`](references/ios/README.md) | 18 HIG-sourced refs |

### Figma (when MCP available)
| Path | Purpose |
|---|---|
| [`references/figma/README.md`](references/figma/README.md) | Routing hub |
| [`references/figma/ios-swiftui.md`](references/figma/ios-swiftui.md) | Figma → iOS/SwiftUI |
| [`references/figma/implement-design/`](references/figma/implement-design/SKILL.md) | Figma → web code |
| [`references/figma/generate-library/`](references/figma/generate-library/SKILL.md) | Build / update DS in Figma |
| [`references/figma/generate-design/`](references/figma/generate-design/SKILL.md) | Build screens in Figma |
| [`references/figma/create-new-file/`](references/figma/create-new-file/SKILL.md) | Create blank Figma file |
| [`references/figma/design-system-rules/`](references/figma/design-system-rules/SKILL.md) | Project-specific Figma-to-code rules |
| [`references/figma/code-connect-batch.md`](references/figma/code-connect-batch.md) | Batch Code Connect mapping |
| [`../../figma-use/`](../../figma-use/SKILL.md) | **Top-level skill** — mandatory prerequisite for every Figma write via `use_figma` |

### Brand & assets
| Path | Purpose |
|---|---|
| [`references/brand/`](references/brand/) | Voice, visual identity, color, typography, logo, messaging |
| [`references/design/`](references/design/) | Logo, CIP, banner sizes, icons, social photos |
| [`references/slides/`](references/slides/) | Presentation creation, layout patterns, copywriting |

### Data & scripts
| Path | Purpose |
|---|---|
| [`data/`](data/) | CSV databases (UX guidelines, tech stacks, etc.). Charts/landing/icons live in designlib MCP. |
| [`scripts/search.py`](scripts/search.py) | BM25 search engine |
| [`scripts/design_system.py`](scripts/design_system.py) | Design system generator (fallback when MCP offline) |
| [`scripts/generate_system_preview.py`](scripts/generate_system_preview.py) | Renders HTML preview of system candidates with 1/2/3 switcher |
| [`scripts/detect-antipatterns.mjs`](scripts/detect-antipatterns.mjs) | Anti-pattern detector (30+ checks) |

### Templates
| Path | Purpose |
|---|---|
| [`templates/ios/Theme/`](templates/ios/README.md) | SwiftUI theme starters |
| [`templates/web/`](templates/web/) | CSS / Tailwind / shadcn starters |
| [`templates/system-preview.html`](templates/system-preview.html) | HTML shell for `generate_system_preview.py` |
| [`templates/brand-guidelines-starter.md`](templates/brand-guidelines-starter.md) | Brand guidelines template |

---

# Layer 2: Design Filters — what we will not emit

**Every command-emitted output MUST pass through all six filters before emit.** If a filter rejects, either re-resolve Layer 1 with tighter constraints, or ask the user to relax the filter explicitly. Do not bypass silently.

<!-- FILTER-EXTENSION: add new filter here -->

## Distinctiveness Gate

The Anti-Pattern filter catches *technical* AI-slop (3-column rows, gradient text, side-stripe). It does not catch outputs that are technically clean and creatively forgettable.

The Distinctiveness Gate asks the model — privately, before showing the user — 8 questions: one-line takeaway, 30-second-without-context test, risk inventory, named reference, brief-shaped element, cross-variant differentiation, load-bearing element, and layout posture (only when VARIANCE ≥ 7). Adjective answers fail; concrete answers pass.

- **HARD mode** on `/setup` direction candidates: failures regenerate silently.
- **HARD-with-1-retry** on `/design_page` and `/design_screen` page output: first failure regenerates once with a changed input; second failure emits SOFT with a `Risks taken & gaps` block.
- **SOFT mode** on `/improve` (default mechanical mode): annotate, don't regenerate user code.
- **Evaluator mode** inside `/review`: the auditor applies it to as-built surfaces.

Full spec: [`references/distinctiveness-gate.md`](references/distinctiveness-gate.md).

## Design Direction

Commit to a BOLD direction. Purpose / tone / differentiation. Bold maximalism and refined minimalism both work — the key is intentionality. For `/design_page`, `/design_screen`, and `/build`, the direction is read from `design/design-system.md` (set by `/setup`); commands must respect it, not override.

## Design Dials

Three tunable parameters: `DESIGN_VARIANCE` (1-10, default 8), `MOTION_INTENSITY` (1-10, default 6), `VISUAL_DENSITY` (1-10, default 4). User sets in chat or via `/setup` (persisted to `design/.cache/interview.json`). Detail: [`references/design-dials.md`](references/design-dials.md).

## Output Rules

A partial output is a broken output.

Banned: `// ...`, `// rest of code`, `// TODO`, `/* ... */`, bare `...`, "for brevity", "the rest follows the same pattern", "I can provide more details", skeletons when full implementation requested.

When approaching token limit: write at full quality to a clean breakpoint, then:
```
[PAUSED — X of Y complete. Send "continue" to resume from: next section]
```

## Frontend Aesthetics — core rules

Web detail in [`references/web/`](references/web/README.md), iOS detail in [`references/ios/`](references/ios/README.md). Top-level summary follows; drill in when working.

### Typography
Pair distinctive display + refined body. Web: [`references/web/typography.md`](references/web/typography.md). iOS Dynamic Type: [`references/ios/layout.md`](references/ios/layout.md).

### Color
Cohesive palette; dominant + sharp accents. Web: [`references/web/color-and-contrast.md`](references/web/color-and-contrast.md). iOS: [`references/ios/color.md`](references/ios/color.md). LILA BAN detail in BAN 3.

### Layout & Space
Web: [`references/web/spatial-design.md`](references/web/spatial-design.md). iOS: [`references/ios/layout.md`](references/ios/layout.md).

### Motion
High-impact moments > scattered micro-interactions. Web: [`references/web/motion-design.md`](references/web/motion-design.md) + [`references/web/motion/`](references/web/motion/README.md). iOS: [`references/ios/motion.md`](references/ios/motion.md).

For implementation (which library, how to register, easing/performance/accessibility) and for short-video output (HyperFrames pipeline), use the sister skill: [`../motion/SKILL.md`](../../motion/SKILL.md).

### Interaction
Web: [`references/web/interaction-design.md`](references/web/interaction-design.md). iOS: [`references/ios/gestures.md`](references/ios/gestures.md), `modals.md`, `controls.md`.

### Responsive (web)
[`references/web/responsive-design.md`](references/web/responsive-design.md).

### Haptics (iOS)
[`references/ios/haptics.md`](references/ios/haptics.md).

### UX Writing
Web: [`references/ux-writing.md`](references/ux-writing.md). iOS: [`references/ios/ui-writing.md`](references/ios/ui-writing.md).

## Anti-Patterns (The AI Slop Test)

If you showed this interface to someone and said "AI made this," would they believe immediately? If yes, that's the problem.

### Absolute Bans

- **BAN 1:** Side-stripe borders — `border-left:`/`border-right:` > 1px on cards/list items/callouts/alerts.
- **BAN 2:** Gradient text — `background-clip: text` with gradient background.
- **BAN 3:** AI color palette — cyan-on-dark, purple-to-blue gradients, neon accents on dark backgrounds.
- **BAN 4:** 3-column card layouts — generic "3 equal cards horizontally" feature rows. Use 2-column zig-zag, asymmetric grid, or horizontal scroll.

### Technical rules
- Animate only `transform` / `opacity` (GPU-accelerated) on web.
- iOS: `.animation()` scoped to value; Reduce Motion = substitute (not remove).
- No `h-screen` — use `min-h-[100dvh]`.
- No `window.addEventListener('scroll')` — use IntersectionObserver / Framer Motion hooks.
- No `z-50` / `z-10` spam — z-index only for systemic layers.
- Check `package.json` / Swift Package deps before importing any 3rd-party library.

### Motion-specific anti-patterns
Full P0–P3 list at [`../motion/references/anti-patterns.md`](../../motion/references/anti-patterns.md). When a candidate output includes motion, walk that list as part of the Anti-Patterns filter pass — P0 (accessibility breakers) and P1 (AI-slop tells) are constraints on emitted code.

---

## Implementation Principles

Match implementation complexity to aesthetic vision. Maximalist → elaborate animations/effects. Minimalist → restraint, precision, spacing.

Interpret creatively. Make unexpected choices. **NEVER converge on common choices across generations.**
