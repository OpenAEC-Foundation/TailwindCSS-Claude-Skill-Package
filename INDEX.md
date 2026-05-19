# INDEX : Tailwind CSS Skill Package Catalog

## Overview

30 deterministic Claude AI skills for Tailwind CSS v3.4 and v4.0+ across 5 categories.

## Summary

| Category | Skills | Focus |
|----------|:------:|-------|
| **core** | 3 | Architecture, design system, v3-vs-v4 model |
| **syntax** | 10 | Utility classes, variants, responsive, dark mode, arbitrary values, v4-only features |
| **impl** | 11 | Configuration (v3/v4), build integrations, plugins, @apply, tailwind-merge, migration |
| **errors** | 5 | Utility-soup, build failures, dynamic classes, v4 migration traps, specificity |
| **agents** | 1 | Cross-skill validator |
| **Total** | **30** | |

## Core Skills (3)

| Skill | Path | Use when |
|-------|------|----------|
| `tailwind-core-architecture` | [skills/source/tailwind-core/tailwind-core-architecture](skills/source/tailwind-core/tailwind-core-architecture) | Starting a new Tailwind project, evaluating utility-first fit, debating utility-first vs BEM / CSS Modules / CSS-in-JS, or reviewing a PR that mixes architectural styles |
| `tailwind-core-design-system` | [skills/source/tailwind-core/tailwind-core-design-system](skills/source/tailwind-core/tailwind-core-design-system) | Designing or picking values: spacing units, color tokens, font sizes, breakpoints, opacity, P3 oklch vs rgb/hsl |
| `tailwind-core-v3-vs-v4` | [skills/source/tailwind-core/tailwind-core-v3-vs-v4](skills/source/tailwind-core/tailwind-core-v3-vs-v4) | Picking a Tailwind version, deciding whether to upgrade, debugging visual regressions after v3 to v4 bump |

## Syntax Skills (10)

| Skill | Path | Use when |
|-------|------|----------|
| `tailwind-syntax-utility-classes` | [skills/source/tailwind-syntax/tailwind-syntax-utility-classes](skills/source/tailwind-syntax/tailwind-syntax-utility-classes) | Writing Tailwind markup, recalling base utility families (spacing, color, typography, layout, flexbox, grid, sizing, border, background, effects) |
| `tailwind-syntax-variants` | [skills/source/tailwind-syntax/tailwind-syntax-variants](skills/source/tailwind-syntax/tailwind-syntax-variants) | Applying conditional styles: hover, focus, active, dark, responsive, group-*, peer-*, aria-*, data-*, has-*, not-*, position-in-parent |
| `tailwind-syntax-responsive` | [skills/source/tailwind-syntax/tailwind-syntax-responsive](skills/source/tailwind-syntax/tailwind-syntax-responsive) | Building responsive layouts: viewport breakpoints, container queries (@container in v4 built-in vs v3 plugin), max-*, arbitrary breakpoints |
| `tailwind-syntax-dark-mode` | [skills/source/tailwind-syntax/tailwind-syntax-dark-mode](skills/source/tailwind-syntax/tailwind-syntax-dark-mode) | Adding, debugging, or migrating dark-mode: media/class/selector strategies, v3 darkMode vs v4 @custom-variant dark |
| `tailwind-syntax-arbitrary-values` | [skills/source/tailwind-syntax/tailwind-syntax-arbitrary-values](skills/source/tailwind-syntax/tailwind-syntax-arbitrary-values) | Writing one-off utility values: `bg-[#1da1f2]`, `w-[calc(100%-2rem)]`, modifier-arbitrary, type hints, CSS variables |
| `tailwind-syntax-state-modifiers` | [skills/source/tailwind-syntax/tailwind-syntax-state-modifiers](skills/source/tailwind-syntax/tailwind-syntax-state-modifiers) | Styling element states, structural positions, pseudo-elements, motion-preference, form states |
| `tailwind-syntax-functional-utilities` | [skills/source/tailwind-syntax/tailwind-syntax-functional-utilities](skills/source/tailwind-syntax/tailwind-syntax-functional-utilities) | (v4 only) Authoring custom utilities with `@utility` directive using `--value()` / `--modifier()` / `--alpha()` / `--spacing()` |
| `tailwind-syntax-3d-transforms` | [skills/source/tailwind-syntax/tailwind-syntax-3d-transforms](skills/source/tailwind-syntax/tailwind-syntax-3d-transforms) | (v4 only) Building 3D scenes with `rotate-x/y/z`, `translate-z`, `perspective-*`, `transform-3d`, backface-visibility |
| `tailwind-syntax-gradients` | [skills/source/tailwind-syntax/tailwind-syntax-gradients](skills/source/tailwind-syntax/tailwind-syntax-gradients) | Building linear/conic/radial gradients, interpolation colour space (oklab/oklch/srgb), v4 `bg-linear-*` / `bg-conic-*` / `bg-radial-*` |
| `tailwind-syntax-modern-utilities` | [skills/source/tailwind-syntax/tailwind-syntax-modern-utilities](skills/source/tailwind-syntax/tailwind-syntax-modern-utilities) | (v4 only) Modern CSS utilities: @starting-style + starting:, transition-discrete, field-sizing-content, scheme-*, font-stretch-*, inset-shadow-*, inset-ring-* |

## Implementation Skills (11)

| Skill | Path | Use when |
|-------|------|----------|
| `tailwind-impl-config-v3` | [skills/source/tailwind-impl/tailwind-impl-config-v3](skills/source/tailwind-impl/tailwind-impl-config-v3) | Configuring `tailwind.config.js` v3: content, theme/extend, presets, plugins, darkMode, corePlugins, safelist, prefix |
| `tailwind-impl-config-v4` | [skills/source/tailwind-impl/tailwind-impl-config-v4](skills/source/tailwind-impl/tailwind-impl-config-v4) | (v4 only) CSS-first config: @import tailwindcss, @theme, @source, @plugin, @utility, @variant, @custom-variant, @reference, @config |
| `tailwind-impl-build-vite` | [skills/source/tailwind-impl/tailwind-impl-build-vite](skills/source/tailwind-impl/tailwind-impl-build-vite) | Wiring Tailwind into a Vite project: v4 @tailwindcss/vite plugin vs v3 PostCSS, HMR, monorepo content paths |
| `tailwind-impl-build-postcss-cli` | [skills/source/tailwind-impl/tailwind-impl-build-postcss-cli](skills/source/tailwind-impl/tailwind-impl-build-postcss-cli) | Non-Vite builds: PostCSS plugin, standalone CLI binary, Rails/PHP/Go projects |
| `tailwind-impl-build-nextjs` | [skills/source/tailwind-impl/tailwind-impl-build-nextjs](skills/source/tailwind-impl/tailwind-impl-build-nextjs) | Installing Tailwind in Next.js (App Router + Pages Router), next/font integration, Turbopack quirks |
| `tailwind-impl-build-frameworks` | [skills/source/tailwind-impl/tailwind-impl-build-frameworks](skills/source/tailwind-impl/tailwind-impl-build-frameworks) | Wiring Tailwind into Astro, Remix, Nuxt, or SvelteKit per Tailwind version |
| `tailwind-impl-plugins-official` | [skills/source/tailwind-impl/tailwind-impl-plugins-official](skills/source/tailwind-impl/tailwind-impl-plugins-official) | Installing/configuring official plugins: @tailwindcss/typography, /forms, /container-queries (built-in v4), /aspect-ratio (deprecated v4) |
| `tailwind-impl-plugins-custom` | [skills/source/tailwind-impl/tailwind-impl-plugins-custom](skills/source/tailwind-impl/tailwind-impl-plugins-custom) | Authoring a custom plugin: plugin() API, addUtilities/addComponents/addVariant/matchUtilities, or v4 @utility directive |
| `tailwind-impl-apply-directive` | [skills/source/tailwind-impl/tailwind-impl-apply-directive](skills/source/tailwind-impl/tailwind-impl-apply-directive) | Using @apply with theme tokens, organizing CSS via @layer base/components/utilities, fixing "Cannot apply unknown utility class" in scoped styles via @reference |
| `tailwind-impl-tailwind-merge` | [skills/source/tailwind-impl/tailwind-impl-tailwind-merge](skills/source/tailwind-impl/tailwind-impl-tailwind-merge) | Deduplicating conflicting Tailwind classes deterministically (twMerge), pairing with clsx + cva (shadcn/ui pattern) |
| `tailwind-impl-migration-v3-v4` | [skills/source/tailwind-impl/tailwind-impl-migration-v3-v4](skills/source/tailwind-impl/tailwind-impl-migration-v3-v4) | Migrating from v3 to v4 using npx @tailwindcss/upgrade plus manual breaking-change catalogue |

## Error Skills (5)

| Skill | Path | Use when |
|-------|------|----------|
| `tailwind-errors-utility-soup` | [skills/source/tailwind-errors/tailwind-errors-utility-soup](skills/source/tailwind-errors/tailwind-errors-utility-soup) | An element has 20+ utility classes on one line, reviewers complain about "utility soup", or deciding between inline utilities, component extraction, or @apply |
| `tailwind-errors-build-failures` | [skills/source/tailwind-errors/tailwind-errors-build-failures](skills/source/tailwind-errors/tailwind-errors-build-failures) | Tailwind build completes but CSS is empty/stale, classes appear in markup but no styles render, HMR shows old build, monorepo packages ship unstyled |
| `tailwind-errors-dynamic-classes` | [skills/source/tailwind-errors/tailwind-errors-dynamic-classes](skills/source/tailwind-errors/tailwind-errors-dynamic-classes) | Classes work in dev but disappear in production, or `bg-${color}-500` template literals / server-injected / CMS-driven class names render but produce no style |
| `tailwind-errors-v4-migration` | [skills/source/tailwind-errors/tailwind-errors-v4-migration](skills/source/tailwind-errors/tailwind-errors-v4-migration) | Upgrading v3 to v4 and hitting visual regressions, build errors after import switch, or behavior changes the codemod missed |
| `tailwind-errors-specificity` | [skills/source/tailwind-errors/tailwind-errors-specificity](skills/source/tailwind-errors/tailwind-errors-specificity) | A utility class is in CSS but does NOT visually override a component class, @apply output behaves differently than the utility, custom utilities ignore variants, or !important syntax in v3 vs v4 |

## Agent Skills (1)

| Skill | Path | Use when |
|-------|------|----------|
| `tailwind-agents-validator` | [skills/source/tailwind-agents/tailwind-agents-validator](skills/source/tailwind-agents/tailwind-agents-validator) | Reviewing a Tailwind CSS codebase for cross-rule violations, performing pre-PR audit, validating a v3-to-v4 migration result, or running periodic codebase health checks |

## Dependency Graph

```
tailwind-core-architecture
        |
tailwind-core-design-system
        |
tailwind-core-v3-vs-v4
        |
+-------+-------+
|               |
syntax-*        impl-*  (depend on core)
                |
            errors-*  (depend on impl)
                |
            agents-validator  (depends on ALL)
```

## Companion Cross-Package Skills

- **shadcn/ui** : component composition layer on top of Tailwind ([shadcn-ui-Claude-Skill-Package](https://github.com/OpenAEC-Foundation/shadcn-ui-Claude-Skill-Package))
- **frontend-design** : visual aesthetics + design system thinking ([Frontend-Design-Claude-Skill-Package](https://github.com/OpenAEC-Foundation/Frontend-Design-Claude-Skill-Package))
- **Vite** : build tool integration ([Vite-Claude-Skill-Package](https://github.com/OpenAEC-Foundation/Vite-Claude-Skill-Package))

`tailwind-impl-tailwind-merge` is the bridge between this package and the shadcn package.

## Discovery

- npm-agentskills manifest : [package.json](package.json) `agents.skills[]`
- OpenAI Codex : [agents/openai.yaml](agents/openai.yaml)
- GitHub topic : `agentskills`

## Verified Sources

All skills verify code against the sources listed in [SOURCES.md](SOURCES.md). Last verified : 2026-05-19.
