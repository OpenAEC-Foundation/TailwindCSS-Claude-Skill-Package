# Masterplan : Tailwind CSS

> Status : Phase 3 refined : research-backed, executable
> Generated : 2026-05-19
> Research basis : `docs/research/vooronderzoek-tailwind.md` (5668 words, 30 sources verified)

## Scope

- Technology : Tailwind CSS
- Versions : v3.4.x (stable, JIT, JS-config) AND v4.0.x (Oxide engine, CSS-first `@theme` config)
- Languages : CSS, HTML, JavaScript
- Prefix : tailwind
- License : MIT

## Mission

Deterministic Claude skills covering Tailwind CSS utility-first methodology, v3-to-v4 migration, configuration patterns, variant system, responsive design, dark mode, plugin authorship, JIT engine, build integration, and the tailwind-merge interop pattern needed for shadcn/ui contexts. Every skill that has v3-vs-v4 divergence ships explicit dual columns (per L-001).

---

## Refinement Decisions

Based on Phase 2 research findings. Every decision is backed by a section reference in `vooronderzoek-tailwind.md`.

| ID | Decision | Reden | Bron |
|----|----------|-------|------|
| D-01 | ADD `syntax-functional-utilities` (v4-only `--value()` / `--modifier()` / `--alpha()` / `--spacing()`) | Functional CSS API has no v3 equivalent, prominent in v4 plugin authoring | vooronderzoek §17.2 |
| D-02 | ADD `syntax-3d-transforms` (v4-only `rotate-x/y/z`, `translate-z`, `perspective-*`, `transform-3d`) | New utility family in v4, no v3 equivalent | vooronderzoek §17.4 |
| D-03 | ADD `syntax-gradients` (v4 expanded `bg-linear-*` / `bg-conic-*` / `bg-radial-*` + interpolation modifiers) | v4 expansion warrants own skill, v3 has only basic linear | vooronderzoek §17.5 |
| D-04 | ADD `syntax-modern-utilities` (bundles `field-sizing-content`, `color-scheme`, `font-stretch`, `@starting-style` / `starting:` variant, `inset-shadow-*`, `inset-ring-*`) | Modern CSS surfaced in v4, individually too small for own skills | vooronderzoek §17.6, §17.7, §17.8 |
| D-05 | ADD `errors-dynamic-classes` (template-literal anti-pattern + v4 `@source inline()` brace-expansion safelist) | #1 reported issue across v3+v4, v4 has unique fix that needs own skill | L-006, vooronderzoek §16, issue 18136 |
| D-06 | MERGE `syntax-pseudo-elements` INTO `syntax-state-modifiers` | Too thin standalone, semantically adjacent (both target sub-elements / states) | vooronderzoek §7 |
| D-07 | MERGE `impl-build-postcss` + `impl-build-cli` → `impl-build-postcss-cli` | Both target Node-less / vanilla pipelines, share content config | vooronderzoek §10.3 |
| D-08 | MERGE `impl-build-astro` + add Remix/Nuxt/SvelteKit → `impl-build-frameworks` | Individually too thin, cross-framework patterns share enough | vooronderzoek §10 |
| D-09 | KEEP `impl-build-vite` as single skill | v3 and v4 setups differ but the skill teaches both in side-by-side tables (per L-001 pattern) | vooronderzoek §10.1 |
| D-10 | DROP standalone `core-engine-model` skill | JIT vs Oxide differences absorbed into `core-v3-vs-v4` to avoid duplication | vooronderzoek §5 |
| D-11 | ADD `@reference` directive coverage to `impl-apply-directive` (not separate skill) | Critical v4 trap for scoped styles (Vue/Svelte/CSS-modules) but tied to @apply story | L-002 |
| D-12 | Every skill where v3+v4 diverge MUST present explicit `v3` and `v4` columns/tabs, never a single shared snippet | Research showed v4 is more divergent than raw masterplan assumed | L-001, L-003, L-004, L-005 |

---

## Final Skill Inventory (30 skills)

### Core (3)
| Skill | Scope |
|-------|-------|
| `tailwind-core-architecture` | Utility-first philosophy, content scanning, atomic CSS, "no design decisions in markup" doctrine, when utility-first wins vs loses |
| `tailwind-core-design-system` | Spacing/color/type/breakpoint scales (v3 + v4 P3 oklch palettes), CSS variable token model in v4 |
| `tailwind-core-v3-vs-v4` | Engine (JIT vs Oxide), config location (JS vs CSS-first), default behaviour shifts, decision tree "should we upgrade" |

### Syntax (10)
| Skill | Scope |
|-------|-------|
| `tailwind-syntax-utility-classes` | Base utility families : spacing, color, typography, layout, flexbox, grid, sizing, border, background, effects |
| `tailwind-syntax-variants` | hover/focus/focus-within/focus-visible/active/visited/target/group-*/peer-*/aria-*/data-*/supports-*/has-*/not-*/in-* (v4) |
| `tailwind-syntax-responsive` | Mobile-first breakpoints, `max-*` (v3.4+), arbitrary `min-[...]:`, container queries built-in v4 / plugin v3 |
| `tailwind-syntax-dark-mode` | media/class/selector strategies v3, `@custom-variant dark` v4, theme-token swap pattern |
| `tailwind-syntax-arbitrary-values` | `bg-[#xxx]`, `w-[calc()]`, modifier-arbitrary `[&_selector]:`, type hints `bg-[length:200px]` |
| `tailwind-syntax-state-modifiers` | first/last/odd/even, before/after pseudo-elements, placeholder, file, marker, selection, motion-safe/reduce |
| `tailwind-syntax-functional-utilities` | v4 `--value()` / `--modifier()` / `--alpha()` / `--spacing()` for functional plugin utilities |
| `tailwind-syntax-3d-transforms` | v4 `rotate-x/y/z-*`, `translate-z-*`, `perspective-*`, `transform-3d`, backface-visibility |
| `tailwind-syntax-gradients` | v4 `bg-linear-*` / `bg-conic-*` / `bg-radial-*`, interpolation modifiers `/oklch`, `/srgb` |
| `tailwind-syntax-modern-utilities` | `@starting-style`/`starting:`, `field-sizing-content`, `color-scheme`, `font-stretch`, `inset-shadow-*`, `inset-ring-*` |

### Implementation (11)
| Skill | Scope |
|-------|-------|
| `tailwind-impl-config-v3` | `tailwind.config.js` : theme, extend, content, plugins, presets, darkMode, corePlugins, safelist, prefix, important |
| `tailwind-impl-config-v4` | CSS-first `@theme`, `@source`, `@plugin`, `@utility`, `@variant`, `@custom-variant`, `@theme inline/static` |
| `tailwind-impl-build-vite` | `@tailwindcss/vite` (v4) vs PostCSS-via-Vite (v3), HMR, content config, monorepo nuances |
| `tailwind-impl-build-postcss-cli` | `@tailwindcss/postcss` (v4) / `tailwindcss` postcss plugin (v3), standalone CLI binary |
| `tailwind-impl-build-nextjs` | App Router + Pages Router, RSC, fonts pipeline, both v3 and v4 setup |
| `tailwind-impl-build-frameworks` | Astro, Remix, Nuxt, SvelteKit short-tour with v3/v4 install patterns each |
| `tailwind-impl-plugins-official` | `@tailwindcss/typography`, `@tailwindcss/forms`, `@tailwindcss/aspect-ratio` (deprecated v4), `@tailwindcss/container-queries` (built-in v4) |
| `tailwind-impl-plugins-custom` | `plugin()` API, `addUtilities`/`addComponents`/`addBase`/`addVariant`/`matchUtilities`/`matchVariant`, theme() access |
| `tailwind-impl-apply-directive` | `@apply` semantics, `@layer` ordering, `@reference` (v4 scoped trap), specificity, important-modifier `!` |
| `tailwind-impl-tailwind-merge` | `twMerge()` for conflict resolution, `clsx` + `cva` pairing (shadcn/ui pattern), config customisation |
| `tailwind-impl-migration-v3-v4` | `npx @tailwindcss/upgrade` tool + manual breaking-change catalogue + dual-version strategies |

### Errors (5)
| Skill | Scope |
|-------|-------|
| `tailwind-errors-utility-soup` | When to extract components vs accept utility-soup, headwind/prettier-plugin-tailwindcss lint, abstraction strategies |
| `tailwind-errors-build-failures` | Content path misses, JIT not detecting classes, Vite/Turbopack-specific quirks, monorepo path issues |
| `tailwind-errors-dynamic-classes` | `bg-${color}-500` template anti-pattern, static-map fix, inline-style fallback, v4 `@source inline()` safelist |
| `tailwind-errors-v4-migration` | Variant stacking order flip (L-004), default border-color / ring-width changes (L-005), removed `corePlugins`/`safelist`/`separator` (L-003), opacity-syntax shifts |
| `tailwind-errors-specificity` | `@apply` ordering, `@layer` conflicts, `!` important-modifier, plugin order, custom CSS interleaving |

### Agents (1)
| Skill | Scope |
|-------|-------|
| `tailwind-agents-validator` | Classname linter, anti-pattern detector, dynamic-class scanner, v3/v4 mixing detector, cross-skill consistency checker |

---

## Execution Plan : 10 Batches

3 skills per batch, 3 tmux workers in parallel. File-scope per worker is the SKILL folder.

| Batch | Worker 1 | Worker 2 | Worker 3 | Deps | Cat |
|-------|----------|----------|----------|------|-----|
| 1 | `core-architecture` | `core-design-system` | `core-v3-vs-v4` | none | core |
| 2 | `syntax-utility-classes` | `syntax-variants` | `syntax-responsive` | B1 | syntax |
| 3 | `syntax-dark-mode` | `syntax-arbitrary-values` | `syntax-state-modifiers` | B2 | syntax |
| 4 | `syntax-functional-utilities` | `syntax-3d-transforms` | `syntax-gradients` | B1 | syntax (v4) |
| 5 | `syntax-modern-utilities` | `impl-config-v3` | `impl-config-v4` | B1 | mixed |
| 6 | `impl-build-vite` | `impl-build-postcss-cli` | `impl-build-nextjs` | B5 | impl-build |
| 7 | `impl-build-frameworks` | `impl-plugins-official` | `impl-plugins-custom` | B5,B6 | impl-build/plugins |
| 8 | `impl-apply-directive` | `impl-tailwind-merge` | `impl-migration-v3-v4` | B5 | impl |
| 9 | `errors-utility-soup` | `errors-build-failures` | `errors-dynamic-classes` | B6 | errors |
| 10 | `errors-v4-migration` | `errors-specificity` | `agents-validator` | ALL | errors+agents |

Estimated duration : ~15 min per batch via tmux workers + ~5 min QG per batch = ~3.5 hours total Phase 4+5.

---

## Standard Agent Prompt Template

Every skill-builder worker receives a prompt of this exact shape (variables filled per skill).

```
Workspace : /home/freek/GitHub/TailwindCSS-Claude-Skill-Package/
Output dir : skills/source/{CATEGORY}/{SKILL_NAME}/
Files to create :
  - SKILL.md (max 500 lines, YAML folded scalar `>`, "Use when..." opener, Keywords-regel met technische + symptom-based + plain-language termen)
  - references/methods.md
  - references/examples.md
  - references/anti-patterns.md

Research input :
  - Vooronderzoek (mandatory read) : docs/research/vooronderzoek-tailwind.md sections {SECTIONS}
  - Topic research (Phase 4 output, read first if exists) : docs/research/topic-research/{SKILL_NAME}-research.md

Sources whitelist (WebFetch only against these) :
  See SOURCES.md primary table. Verify EVERY code snippet against an actual fetched URL.

Scope (bullets) :
  {SCOPE_BULLETS}

v3-vs-v4 handling :
  {V3_V4_NOTE}

Quality rules :
  - English-only (D-001)
  - Deterministic : "ALWAYS X" / "NEVER Y" (not "you might want to")
  - License : MIT in frontmatter
  - compatibility : "Designed for Claude Code. Requires Tailwind CSS v3.4 or v4.0+."
  - Section headings use `:` separator, NEVER em-dash
  - YAML description : folded scalar `>` opener, "Use when..." starter, end with "Keywords:" line mixing technical + symptom + plain-language terms
  - 3 reference files mandatory
  - Cross-link to companion skills via plain markdown links

Anti-patterns (apply globally) :
  - NEVER hallucinate API names : every directive/utility/function must trace to a SOURCES.md URL
  - NEVER quote YAML description (use folded `>`)
  - NEVER write a README.md inside the skill folder (L-010 QGIS pattern)
  - NEVER exceed 500 lines in SKILL.md (overflow to references/)

Quality gate at completion :
  - node /home/freek/GitHub/Skill-Package-Workflow-Template/scripts/validate-frontmatter.js <skill-pad> exit 0
  - node /home/freek/GitHub/Skill-Package-Workflow-Template/scripts/validate-line-count.js <skill-pad> exit 0
  - node /home/freek/GitHub/Skill-Package-Workflow-Template/scripts/validate-structure.js <skill-pad> exit 0
  - node /home/freek/GitHub/Skill-Package-Workflow-Template/scripts/validate-emdash.js <skill-pad> exit 0
  - wc -l SKILL.md must be < 500

Commit message format : feat(skill): tailwind-{cat}-{topic}
```

---

## Per-Skill Prompt Bodies

### Batch 1 : Core

#### `tailwind-core-architecture`
- SECTIONS : §3 Architecture, §4 Engine Model, §11 utility-first philosophy
- V3_V4_NOTE : Tech-version-agnostic skill. Mention v4 Oxide engine briefly, defer engine deep-dive to `core-v3-vs-v4`.
- SCOPE :
  - Utility-first philosophy and "no design decisions in markup" doctrine
  - Atomic CSS approach + tradeoffs vs CSS-in-JS, BEM, CSS Modules
  - Content scanning model (how Tailwind finds your classes)
  - When utility-first wins : design systems, rapid prototyping, design-token-driven projects
  - When utility-first loses : heavy CMS-driven content, content authored outside dev control
  - Decision tree : "should this project use Tailwind?"

#### `tailwind-core-design-system`
- SECTIONS : §6.3 Theme tokens, §9 Color system, §9 Spacing scale
- V3_V4_NOTE : Side-by-side tables for v3 (rgb/hsl + tailwind.config theme) vs v4 (oklch P3 + CSS @theme tokens)
- SCOPE :
  - Default spacing scale (0 to 96 in 0.25rem steps), arbitrary spacing
  - Default color palette families : slate/gray/zinc/neutral/stone + red/orange/amber/yellow/lime/green/emerald/teal/cyan/sky/blue/indigo/violet/purple/fuchsia/pink/rose
  - v4 oklch P3 wide-gamut colors vs v3 rgb/hsl
  - Type scale (text-xs to text-9xl), line-height pairing, font-feature settings
  - Default breakpoints (sm 640, md 768, lg 1024, xl 1280, 2xl 1536)
  - CSS variable token model in v4 (`--color-*`, `--spacing-*`, `--font-*`)
  - Opacity modifier syntax (`bg-red-500/50`, `text-white/[0.85]`)

#### `tailwind-core-v3-vs-v4`
- SECTIONS : §4 Engine, §5 Configuration, §13 Migration (decision tree only)
- V3_V4_NOTE : This IS the v3/v4 comparison skill. Tables everywhere.
- SCOPE :
  - Engine : JIT (v3, JavaScript) vs Oxide (v4, Rust + Lightning CSS)
  - Performance numbers from v4 launch blog (5x faster full builds, 100x incremental)
  - Config location : `tailwind.config.js` (v3) vs `@theme` in main.css (v4)
  - Default behaviour shifts : border-color, ring-width, hover on touch devices, scale renames
  - Removed in v4 : `corePlugins`, `safelist`, `separator` options
  - Decision tree : "should we upgrade now / partial / wait?"
  - Cross-link to `impl-migration-v3-v4` for upgrade execution

### Batch 2 : Syntax core

#### `tailwind-syntax-utility-classes`
- SECTIONS : §6 API surface, §11 spacing, padding, margin, §3 utility-first
- V3_V4_NOTE : Mostly shared. Note shadow scale and color renames in v4 in a small "v4 changes" callout.
- SCOPE :
  - Spacing : `p-*`, `px/py/pt/pr/pb/pl-*`, `m-*`, `space-x/y-*`, `gap-*`
  - Color : `text-*`, `bg-*`, `border-*`, `ring-*`, `divide-*`, `outline-*`, `accent-*`
  - Typography : `text-{size}`, `font-{weight}`, `leading-*`, `tracking-*`, `font-{family}`
  - Layout : `block`/`inline`/`flex`/`grid`/`hidden`, `position`, `top/right/bottom/left`, `z-*`
  - Flexbox : `flex-row/col`, `justify-*`, `items-*`, `flex-wrap`, `flex-1`, `shrink/grow`
  - Grid : `grid-cols-*`, `col-span-*`, `grid-rows-*`, `grid-flow-*`, `auto-cols/rows`
  - Sizing : `w-*`, `h-*`, `min/max-w/h-*`, fractional + arbitrary
  - Border : `border`, `border-{n}`, `rounded-*`, `border-*-{n}`, `divide-*`
  - Background : `bg-{color}`, `bg-[url(...)]`, `bg-{size}`, `bg-{position}`, `bg-{repeat}`
  - Effects : `shadow-*`, `opacity-*`, `mix-blend-*`, `backdrop-blur-*`

#### `tailwind-syntax-variants`
- SECTIONS : §7 Variant System (complete listing), §16 anti-patterns
- V3_V4_NOTE : v4 changed variant stacking order to left-to-right (L-004). Include explicit example.
- SCOPE :
  - State : `hover:`, `focus:`, `focus-within:`, `focus-visible:`, `active:`, `visited:`, `target:`
  - Relational : `group-{state}:`, `peer-{state}:`, named groups `group/sidebar:`
  - Attribute : `aria-*:` (e.g. `aria-checked:`), `data-*:` (e.g. `data-[state=open]:`), `supports-[selector]:`
  - Has selector : `has-[selector]:`, `not-[selector]:` (v4)
  - Position-in-parent : `in-*:` (v4), `*:` (direct children)
  - State stacking rules and order (v3 right-to-left vs v4 left-to-right per L-004)
  - Custom variants via plugin `addVariant()` (v3) and `@custom-variant` (v4)

#### `tailwind-syntax-responsive`
- SECTIONS : §8 Responsive + Container Queries
- V3_V4_NOTE : Container queries built-in v4 vs plugin v3. Must show both. Side-by-side examples.
- SCOPE :
  - Mobile-first principle : `sm:`, `md:`, `lg:`, `xl:`, `2xl:` apply from breakpoint up
  - `max-{breakpoint}:` (v3.4+) for desktop-first overrides
  - Arbitrary breakpoints : `min-[400px]:flex`, `max-[800px]:hidden`
  - Custom breakpoint registration v3 (`theme.screens`) vs v4 (`@theme { --breakpoint-* }`)
  - Container queries : `@container` parent + `@sm:`, `@md:`, `@[400px]:` children
  - v4 built-in vs v3 `@tailwindcss/container-queries` plugin (deprecated in v4)
  - Named containers : `@container/sidebar` + `@sm/sidebar:`

### Batch 3 : Syntax extended

#### `tailwind-syntax-dark-mode`
- SECTIONS : §9 Dark Mode Strategies, §5 Configuration
- V3_V4_NOTE : v3 uses `darkMode: 'class'` config, v4 uses `@custom-variant dark (&:where(.dark, .dark *))`. Show both completely.
- SCOPE :
  - Default `media` strategy : respects `prefers-color-scheme`
  - `class` strategy (v3 `darkMode: 'class'`) : add `class="dark"` to html
  - `selector` strategy (v3.4.1+) for attribute-based : `darkMode: ['class', '[data-theme="dark"]']`
  - v4 syntax : `@custom-variant dark (&:where(.dark, .dark *))`
  - Theme-token swap pattern : light vs dark CSS variable values
  - Toggle JavaScript snippet (localStorage + matchMedia)

#### `tailwind-syntax-arbitrary-values`
- SECTIONS : §14 JIT + Arbitrary Values
- V3_V4_NOTE : Identical syntax in v3 and v4. Note JIT performance in v4 Oxide engine.
- SCOPE :
  - Value-arbitrary : `bg-[#1da1f2]`, `w-[calc(100%-2rem)]`, `text-[14.5px]`
  - Modifier-arbitrary : `[&:nth-child(3)]:underline`, `[&>*]:p-4`
  - Variant-arbitrary : `data-[state=open]:bg-red-500`, `aria-[expanded=true]:rotate-180`
  - Type hints to disambiguate : `bg-[length:200px_100px]`, `bg-[image:url(...)]`, `text-[color:var(--my-color)]`
  - CSS variables : `bg-[--my-bg]`, `bg-[color:var(--my-bg)]`
  - Performance : JIT only generates classes for what is in your source

#### `tailwind-syntax-state-modifiers`
- SECTIONS : §7 Variants, pseudo-element coverage
- V3_V4_NOTE : Largely shared. Note `before:` / `after:` require `content-[""]` default in v4.
- SCOPE :
  - Position : `first:`, `last:`, `only:`, `odd:`, `even:`, `first-of-type:`, `last-of-type:`, `nth-[3n+1]:`
  - Pseudo-elements : `before:`, `after:`, `placeholder:`, `file:`, `marker:`, `selection:`, `first-letter:`, `first-line:`
  - Motion : `motion-safe:`, `motion-reduce:`
  - Form states : `required:`, `valid:`, `invalid:`, `disabled:`, `checked:`, `indeterminate:`, `default:`, `autofill:`, `read-only:`, `placeholder-shown:`
  - Content : `content-['hello']`, `content-[attr(data-label)]`
  - Empty / target / open : `empty:`, `target:`, `open:` (for `<details>` / `<dialog>`)

### Batch 4 : v4-only syntax

#### `tailwind-syntax-functional-utilities`
- SECTIONS : §17.2 Functional CSS functions
- V3_V4_NOTE : v4-only. Mark this skill compatibility "Tailwind CSS v4 only".
- SCOPE :
  - `--value()` : read a key from theme (e.g. `--value(--color-*)` to access any registered colour)
  - `--modifier()` : read the modifier part of a utility (e.g. `bg-red-500/50` modifier is `50`)
  - `--alpha()` : compose alpha into a color
  - `--spacing()` : access the spacing scale by step
  - Use cases in custom utilities and plugin authoring
  - Cross-link to `impl-plugins-custom`

#### `tailwind-syntax-3d-transforms`
- SECTIONS : §17.4 3D transforms
- V3_V4_NOTE : v4-only. v3 has only basic `rotate-*`, `translate-*`, `scale-*`.
- SCOPE :
  - `transform-3d` to opt-in to 3D rendering context
  - `perspective-*` (none/dramatic/near/normal/midrange/distant) + arbitrary `perspective-[800px]`
  - `rotate-x-*`, `rotate-y-*`, `rotate-z-*` (replaces 2D `rotate-*` for 3D scenes)
  - `translate-z-*` (z-axis offset)
  - `backface-visible` / `backface-hidden`
  - Card-flip + 3D-carousel example patterns

#### `tailwind-syntax-gradients`
- SECTIONS : §17.5 Expanded gradient APIs
- V3_V4_NOTE : v4 expanded vs v3 basic. Side-by-side.
- SCOPE :
  - Linear : `bg-linear-to-r`, `bg-linear-45`, `bg-linear-to-tr/srgb`, `bg-linear-to-r/oklch`
  - Conic : `bg-conic-*`, `bg-conic-90`, conic gradients with stops
  - Radial : `bg-radial-*`, `bg-radial-[at_top_left]`
  - Color stops : `from-{color}`, `via-{color}`, `to-{color}`, `from-{position}` (e.g. `from-10%`)
  - Interpolation modifiers : `/oklch` (default), `/srgb`, `/hsl`, `/longer-hue`, `/shorter-hue`
  - v3 equivalents (linear only via `bg-gradient-to-*`)

### Batch 5 : Modern utilities + Config

#### `tailwind-syntax-modern-utilities`
- SECTIONS : §17.6, §17.7, §17.8
- V3_V4_NOTE : All v4-only. Single skill bundles small individually-thin features.
- SCOPE :
  - `@starting-style` + `starting:` variant for CSS-only enter animations on `<dialog>` / popovers
  - `transition-discrete` for transitioning `display: none` to block
  - `field-sizing-content` for auto-resizing `<textarea>`
  - `color-scheme-light`, `color-scheme-dark`, `color-scheme-auto` for native form controls
  - `font-stretch-*` for variable-font width axes
  - `inset-shadow-*` (up to 4 layered inset shadows)
  - `inset-ring-*` (inset rings, separate from outer rings)

#### `tailwind-impl-config-v3`
- SECTIONS : §5 Configuration v3, §6.3 Theme tokens
- V3_V4_NOTE : v3-only skill. Cross-link `impl-config-v4` for v4 equivalent. Cross-link `impl-migration-v3-v4` for upgrade.
- SCOPE :
  - `tailwind.config.{js,mjs,ts}` location, ESM vs CJS
  - `content : []` globs, monorepo paths, `transform` for content scanning
  - `theme` (replace) vs `theme.extend` (additive) semantics
  - `presets : []` for shared base configs
  - `plugins : [plugin(({addUtilities,...}) => {})]`
  - `darkMode : 'media' | 'class' | ['class', '[data-theme=dark]']` (v3.4.1+)
  - `corePlugins : { float : false }` to disable built-ins (REMOVED in v4)
  - `safelist : ['bg-red-500']` to force-include (REMOVED in v4, see `errors-dynamic-classes` for v4 fix)
  - `prefix : 'tw-'`, `important : true | '#app'`, `separator : '_'`
  - `blocklist : []`, `future : { hoverOnlyWhenSupported : true }`

#### `tailwind-impl-config-v4`
- SECTIONS : §5 Configuration v4, §6.1 Directives, §17.9 @theme inline/static
- V3_V4_NOTE : v4-only skill. Cross-link `impl-config-v3` for v3 equivalent.
- SCOPE :
  - `@import "tailwindcss";` replaces all three `@tailwind` directives (L-001)
  - `@theme { --color-mint-500: oklch(0.72 0.11 178); --font-display: "Inter Variable", sans-serif; }`
  - `@theme inline` (substitutes values at build) vs `@theme static` (keeps CSS variables)
  - `@source "../**/*.{html,ts,tsx}"` for additional content scanning
  - `@source not "../legacy/**"` to exclude
  - `@source inline("bg-{red,blue}-500")` for safelisting with brace expansion (L-006)
  - `@plugin "./my-plugin.js";` (v4 way to load v3-style JS plugins)
  - `@utility tab-* { tab-size: --value(integer); }` for custom utilities
  - `@variant pointer-coarse (@media (pointer: coarse));` for custom variants
  - `@custom-variant dark (&:where(.dark, .dark *));`
  - `@reference "tailwindcss";` for scoped `@apply` (L-002, cross-link `impl-apply-directive`)
  - `@config "./tailwind.config.js";` for back-compat with v3 JS config

### Batch 6 : Build (high-traffic frameworks)

#### `tailwind-impl-build-vite`
- SECTIONS : §10.1 Vite, §6.1 Directives
- V3_V4_NOTE : v3 uses postcss-plugin via Vite, v4 uses dedicated `@tailwindcss/vite` plugin. Show both fully.
- SCOPE :
  - v4 : install `npm i tailwindcss @tailwindcss/vite`, add plugin to `vite.config.ts`, import `tailwindcss` in main.css
  - v3 : install `npm i -D tailwindcss postcss autoprefixer`, init configs, add `@tailwind` directives
  - Vite-specific : HMR works out of the box, content config not needed in v4 (auto-detect)
  - Monorepo : workspaces + content paths or `@source`
  - Common breakage : v4.0.8 Astro break (issue 16733), Turbopack arbitrary-value miss (issue 19825)

#### `tailwind-impl-build-postcss-cli`
- SECTIONS : §10.3 PostCSS + CLI
- V3_V4_NOTE : Both versions. v4 = `@tailwindcss/postcss`, v3 = `tailwindcss`. CLI binary identical philosophy across versions.
- SCOPE :
  - `postcss.config.js` setup for v3 vs v4
  - Standalone CLI : `npx @tailwindcss/cli -i input.css -o output.css --watch`
  - `--minify`, `--watch`, `--input`, `--output`
  - When to use CLI : Rails, PHP, Go, non-Node projects
  - Autoprefixer pairing (v3 needed it explicitly, v4 includes it via Lightning CSS)

#### `tailwind-impl-build-nextjs`
- SECTIONS : §10.5 Next.js
- V3_V4_NOTE : Both App and Pages routers. Both versions installation.
- SCOPE :
  - v4 App Router : `app/globals.css` with `@import "tailwindcss";`, import in `layout.tsx`
  - v3 App Router : PostCSS config + `@tailwind` directives
  - Pages Router : same CSS setup, import in `pages/_app.tsx`
  - Next.js fonts (`next/font/google`) : how to wire family into `@theme { --font-sans: var(--font-inter); }`
  - RSC : Tailwind is build-time so RSC has no runtime cost
  - Turbopack quirk : issue 19825 arbitrary value miss workaround

### Batch 7 : Build (other) + Plugins

#### `tailwind-impl-build-frameworks`
- SECTIONS : §10.7 Other frameworks
- V3_V4_NOTE : Per framework, dual columns.
- SCOPE :
  - Astro : `npx astro add tailwind` (v3) vs `@tailwindcss/vite` (v4)
  - Remix : Vite plugin pattern (same as Vite skill)
  - Nuxt : `@nuxtjs/tailwindcss` module (v3) vs `@tailwindcss/vite` (v4)
  - SvelteKit : Vite plugin + global CSS file
  - Per framework : where the CSS file lives, where to import it

#### `tailwind-impl-plugins-official`
- SECTIONS : §11.5 Plugins
- V3_V4_NOTE : `@tailwindcss/aspect-ratio` deprecated v4 (native CSS works). `@tailwindcss/container-queries` built-in v4.
- SCOPE :
  - `@tailwindcss/typography` : `prose` classes, `prose-*` modifiers, custom typography config
  - `@tailwindcss/forms` : `form-input`, `form-select`, strategy `'class'` vs `'base'`
  - `@tailwindcss/container-queries` : v3 plugin, v4 built-in (note in skill)
  - `@tailwindcss/aspect-ratio` : v3 plugin, v4 deprecated (use native `aspect-*` utilities)
  - Installation per version (v3 via `plugins: [require('@tailwindcss/typography')]`, v4 via `@plugin "@tailwindcss/typography"`)

#### `tailwind-impl-plugins-custom`
- SECTIONS : §12 Plugin Authorship
- V3_V4_NOTE : Plugin API mostly compatible v3-v4. Note `matchUtilities` extended in v4.
- SCOPE :
  - `plugin(({addUtilities, addComponents, addBase, addVariant, theme, matchUtilities}) => {})` signature
  - `addUtilities({ '.skew-10deg': { transform: 'skewY(-10deg)' } })`
  - `addComponents({ '.btn': { padding: '0.5rem 1rem', borderRadius: '0.25rem' } })`
  - `addVariant('my-variant', '&:hover:focus')`
  - `matchUtilities({ tab: value => ({ tabSize: value }) }, { values: theme('tabSize') })`
  - v4-only : `@utility` directive as alternative authoring path (cross-link `impl-config-v4`)
  - Theme access : `theme('colors.red.500')`, `theme('spacing.4')`
  - Packaging as npm module

### Batch 8 : Apply + tailwind-merge + Migration

#### `tailwind-impl-apply-directive`
- SECTIONS : §11 @apply + @layer, L-002 @reference
- V3_V4_NOTE : `@reference` is v4-only escape hatch for scoped stylesheets. Include explicit warning.
- SCOPE :
  - `@apply text-2xl font-bold` semantics : copies utility declarations
  - When `@apply` makes sense : component extraction with React/Vue isn't possible
  - When `@apply` is anti-pattern : utility extraction defeats utility-first philosophy
  - `@layer base { ... }`, `@layer components { ... }`, `@layer utilities { ... }`
  - Layer order : base then components then utilities (specificity ascends)
  - Important modifier : `font-bold!` (v4 trailing) vs `!font-bold` (v3 leading)
  - v4 scoped trap (L-002) : Vue SFC `<style>`, Svelte component, CSS modules need `@reference "../app.css";` as first line OR `@apply` fails with "Cannot apply unknown utility class"
  - Plugin-order vs `@layer` interaction

#### `tailwind-impl-tailwind-merge`
- SECTIONS : §15 tailwind-merge Interop
- V3_V4_NOTE : Library is version-aware. `tailwind-merge` 2.x for v3, 3.x for v4.
- SCOPE :
  - Why : `twMerge('p-2 p-4')` resolves to `p-4` deterministically (last-wins on conflict)
  - Install : `npm i tailwind-merge`
  - Basic usage : `twMerge(baseClasses, overrideClasses)`
  - `clsx` pairing : `twMerge(clsx('p-2', condition && 'p-4'))`
  - `cva` (class-variance-authority) : shadcn/ui pattern
  - Custom config : `extendTailwindMerge` for prefix or custom utilities
  - Performance : memoised, safe in render paths
  - When NOT to use : non-dynamic class lists (just static utilities)
  - Cross-link to companion shadcn pkg

#### `tailwind-impl-migration-v3-v4`
- SECTIONS : §13 Migration v3 to v4
- V3_V4_NOTE : This IS the migration skill. Cross-link `core-v3-vs-v4` for decision tree, `errors-v4-migration` for traps.
- SCOPE :
  - `npx @tailwindcss/upgrade` automatic tool : what it covers (config conversion, @tailwind to @import, variant order)
  - Manual breaking changes catalogue : removed config options, default behaviour shifts, opacity syntax, prefix syntax flip
  - Dual-version strategy : run both versions in monorepo until upgrade complete
  - Pre-migration checklist : audit `corePlugins`, `safelist`, `separator` usages, prefer-explicit border colors
  - Post-migration verification : visual regression, scan for `@apply` in scoped styles (L-002)

### Batch 9 : Errors

#### `tailwind-errors-utility-soup`
- SECTIONS : §16 anti-patterns
- V3_V4_NOTE : Version-agnostic.
- SCOPE :
  - "Utility soup" symptom : 30+ classes on a single element across multiple lines
  - When utility-soup is fine : leaf components used once
  - When to extract : repeated patterns (3+ uses) become React/Vue component or `@apply` component class
  - `prettier-plugin-tailwindcss` for stable ordering
  - `eslint-plugin-tailwindcss` for class validation
  - Headwind editor plugin for sorting

#### `tailwind-errors-build-failures`
- SECTIONS : §14 production build, §16 anti-patterns
- V3_V4_NOTE : Common failures shared, fixes may differ per version.
- SCOPE :
  - "No classes generated" : content config misconfigured, JIT cannot find your files
  - v3 `content : ['./src/**/*.{js,ts,jsx,tsx}']` setup
  - v4 auto-detect + `@source "./external/**/*.tsx"` for monorepo
  - Vite cache invalidation issues (delete `node_modules/.vite`)
  - Turbopack arbitrary-value miss (issue 19825) : disable Turbopack or use static values
  - Astro v4.0.8 break (issue 16733) : pin tailwind to specific minor
  - Monorepo path issues : absolute paths in `@source`

#### `tailwind-errors-dynamic-classes`
- SECTIONS : L-006, §16 anti-patterns, §16 issue 18136
- V3_V4_NOTE : This is THE anti-pattern. v4 has unique `@source inline()` fix not available in v3.
- SCOPE :
  - Symptom : `const bgClass = ` then backtick-bg-${color}-500-backtick works in dev (HMR re-scans) but not prod
  - Root cause : Tailwind scans source files as plain text, template literals are opaque
  - Fix 1 (preferred) : static map `{ red: 'bg-red-500', blue: 'bg-blue-500' }[color]`
  - Fix 2 : inline styles for truly dynamic values `style={{ background: color }}`
  - Fix 3 v3 : `safelist : ['bg-red-500', { pattern: /bg-(red|blue|green)-(50|100|500)/ }]`
  - Fix 4 v4 : `@source inline("{hover:,}bg-{red,blue,green}-{50,{100..900..100},950}")` with brace expansion
  - Why brace expansion is powerful : generates Cartesian product without listing each
  - Real example from issue 18136 walkthrough

### Batch 10 : Errors closing + Agent

#### `tailwind-errors-v4-migration`
- SECTIONS : L-003, L-004, L-005, §13 Migration
- V3_V4_NOTE : Catalogue of upgrade traps. Cross-link `impl-migration-v3-v4` for upgrade tool.
- SCOPE :
  - Variant stacking order : v3 right-to-left (`first:*:pt-0` is direct children, first) becomes v4 left-to-right (`*:first:pt-0`) (L-004)
  - Default border-color : v3 `gray-200` becomes v4 `currentColor`, add explicit `border-gray-200` on all bordered elements OR shim in `@layer base`
  - Default ring-width : v3 `3px` becomes v4 `1px`, add `ring-3` or shim
  - Shadow scale rename : `shadow-sm` shifted because `shadow-xs` was inserted
  - Same renaming applies to `blur-*`, `rounded-*`, `drop-shadow-*`, `backdrop-blur-*`
  - Removed config options : `corePlugins`, `safelist`, `separator` no replacement (L-003)
  - Opacity syntax change : `bg-opacity-50` removed, use `bg-black/50` (was already deprecated in v3.0+)
  - Prefix syntax flip : `tw-flex` becomes `tw:flex` in v4
  - `space-x-*` / `space-y-*` behaviour changed (uses `:where()` selector now)

#### `tailwind-errors-specificity`
- SECTIONS : §11 @apply + @layer, plugin-order
- V3_V4_NOTE : Version-agnostic mostly.
- SCOPE :
  - `@layer` order : base (low), components (mid), utilities (high)
  - When utility does not override component : check `@layer utilities` placement
  - Plugin order in v3 `plugins: []` array matters (last wins)
  - `@apply` ordering inside same class : declaration order, not specificity, last wins for same property
  - Important modifier : `font-bold!` (v4) vs `!font-bold` (v3), both produce `!important`
  - `@layer utilities` for custom utilities to participate in specificity correctly
  - Cross-link to `impl-apply-directive` and `impl-plugins-custom`

#### `tailwind-agents-validator`
- SECTIONS : ALL, this skill validates against other skills
- V3_V4_NOTE : Detects v3/v4 mixing as an error pattern.
- SCOPE :
  - Validation rules checklist :
    - No backtick-bg-template-literal-backtick class names (cross-ref `errors-dynamic-classes`)
    - No `@apply` in Vue SFC/Svelte/CSS-modules without `@reference` (cross-ref `impl-apply-directive`)
    - No `corePlugins`/`safelist`/`separator` in `@config`-loaded JS configs (cross-ref `errors-v4-migration`)
    - No undefined custom classes outside `@layer`
    - Variant-order consistency (left-to-right in v4) (cross-ref `syntax-variants`)
    - Border/ring elements have explicit colour/width post-v4 upgrade (cross-ref `errors-v4-migration`)
  - Output : actionable report referencing skill + rule violated
  - Integrates with codebase scan + per-skill cross-reference

---

## Phase 4 strategy (topic-research)

Per BOOTSTRAP-RUNBOOK §6.3 skip-criteria : the vooronderzoek is 5668 words covering all 30 skills directly. Topic-research will be SKIPPED for skills where the vooronderzoek section is self-sufficient (most skills) and EXECUTED for skills needing deeper drill-down (notably `impl-plugins-custom`, `impl-tailwind-merge`, `impl-migration-v3-v4`, `errors-dynamic-classes` where issue-mining could go deeper).

Documented in DECISIONS.md after Phase 3 approval.

---

## Verify Phase 3

```bash
test -f "docs/masterplan/tailwind-masterplan.md" && \
  grep -q "## Refinement Decisions" "docs/masterplan/tailwind-masterplan.md" && \
  grep -q "## Execution Plan" "docs/masterplan/tailwind-masterplan.md" && \
  test "$(grep -c '^#### ' docs/masterplan/tailwind-masterplan.md)" -ge 30
```

Expected : >= 30 per-skill sub-headings, decisions + execution plan present.
