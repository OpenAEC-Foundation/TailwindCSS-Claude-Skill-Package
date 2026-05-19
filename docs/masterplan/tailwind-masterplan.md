# Masterplan : Tailwind CSS

> Status : Phase 1 raw : pre-research
> Generated : 2026-05-19

## Scope

- Technology : Tailwind CSS
- Versions : v3.4.x (stable, JIT, JS-config) AND v4.0.x (Oxide engine in Rust, CSS-first `@theme` config)
- Languages : CSS, HTML, JavaScript
- Prefix : tailwind
- License : MIT

## Mission

Deterministic Claude skills covering Tailwind CSS utility-first methodology, v3-to-v4 migration, configuration patterns, variant system, responsive design, dark mode, plugin authorship, JIT engine, build integration, and the tailwind-merge interop pattern needed for shadcn/ui contexts.

## Identified Topics (raw, pre-research)

### Core
- core-architecture : utility-first philosophy, Oxide engine vs JIT, design tokens, content scanning
- core-design-system : spacing scale, color scale (default + P3 in v4), type scale, breakpoints
- core-v3-vs-v4 : breaking changes overview, engine differences, config-location swap, default behaviors

### Syntax
- syntax-utility-classes : base utility patterns (spacing, color, typography, layout, flexbox, grid)
- syntax-variants : hover/focus/active/disabled, group-*, peer-*, aria-*, data-*, supports-*
- syntax-responsive : mobile-first breakpoints, sm/md/lg/xl/2xl, container queries (`@container` in v4)
- syntax-dark-mode : `dark:` variant, class strategy vs media strategy, v4 `@variant` selector
- syntax-arbitrary-values : `bg-[#1da1f2]`, `w-[calc(100%-2rem)]`, modifier-arbitrary, type hints
- syntax-state-modifiers : first/last/odd/even, before/after, placeholder, file, marker, selection
- syntax-pseudo-elements : `::before`, `::after`, content utilities

### Implementation
- impl-config-v3 : `tailwind.config.js` (theme, extend, content, plugins, presets)
- impl-config-v4 : CSS-first `@theme`, `@source`, `@plugin`, `@utility`, `@variant` directives
- impl-build-vite : `@tailwindcss/vite` plugin (v4) or PostCSS (v3), HMR, content config
- impl-build-postcss : `postcss.config.js`, autoprefixer pairing, production purge
- impl-build-cli : standalone `tailwindcss` CLI for non-Node builds
- impl-build-nextjs : App Router integration, RSC compatibility, fonts pipeline
- impl-build-astro : `@astrojs/tailwind` vs Vite plugin, scoped vs global
- impl-plugins-official : `@tailwindcss/typography`, `@tailwindcss/forms`, `@tailwindcss/container-queries`, `@tailwindcss/aspect-ratio`
- impl-plugins-custom : `plugin()` API, `addUtilities`, `addComponents`, `matchUtilities`, theme access
- impl-apply-directive : `@apply` use cases vs anti-patterns, component classes, `@layer` placement
- impl-tailwind-merge : `tailwind-merge` for class deduplication (shadcn/ui pattern), `clsx` interop
- impl-migration-v3-v4 : automated upgrade tool, manual breaking changes, dual-version strategy

### Errors
- errors-utility-soup : long class lists, when to extract components vs accept utility-soup, lint patterns
- errors-build-failures : content path misses, JIT not picking classes, dynamic class detection failure
- errors-purge-issues : production CSS missing classes, safelist usage, dynamic class anti-patterns
- errors-specificity : `@apply` ordering, `@layer` conflicts, important-modifier `!`, plugin-order
- errors-v4-migration : common upgrade pitfalls, deprecated utilities, color-format changes

### Agents
- agents-validator : cross-skill consistency checker, classname linter, anti-pattern detector

## Estimated Skill Count

| Category | Estimated |
|----------|-----------|
| core | 3 |
| syntax | 7 |
| impl | 13 |
| errors | 5 |
| agents | 1 |
| **Total** | **~29** |

Final count adjusted after Phase 2 research (merges/drops/splits in Refinement Decisions table).

## Cross-Package Boundaries

- **shadcn/ui** (companion pkg) : Tailwind is required dependency. Skills here MUST stay tech-agnostic for utility patterns. shadcn-specific class composition lives in shadcn pkg, but `tailwind-merge` usage lives here (utility-level tool).
- **frontend-design** (companion pkg) : design-system thinking + visual aesthetics live there. Token-level scale definitions live here.
- **Vite** (companion pkg) : Vite-plugin setup mentioned in impl-build-vite, but deep Vite config patterns live in Vite pkg. Skill here covers Tailwind side only.

## Open Questions for Phase 2

1. Does v4.0 still support `tailwind.config.js` for back-compat, or CSS-first only?
2. Container queries : v4 built-in vs v3 plugin behavior differences?
3. `@apply` deprecation status in v4?
4. Color format defaults : v4 OKLCH vs v3 HSL/RGB?
5. Plugin API stability v3 → v4?
6. Migration tool coverage : what does it auto-fix vs manual?

## Next : Phase 2 Deep Research

Dispatch research-agent to produce `docs/research/vooronderzoek-tailwind.md` (>=2000 words, WebFetch-verified against SOURCES.md URLs, Newly Discovered Sub-Topics section).
