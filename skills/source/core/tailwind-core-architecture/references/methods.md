# Methods : Tailwind Core Architecture

This file catalogues the architectural concepts referenced from `SKILL.md`. It is a conceptual reference; concrete API names live in the `syntax-*` and `impl-*` skills.

## 1. The four utility-first benefits (official)

Per https://tailwindcss.com/docs/utility-first :

| # | Benefit | Mechanism |
|---|---------|-----------|
| 1 | You get things done faster | No class-naming, no file-switching, no specificity wars |
| 2 | Making changes feels safer | A utility class affects only the element where it lives |
| 3 | Maintaining old projects is easier | Find the element, change the classes; no detached stylesheet to decipher |
| 4 | Your code is more portable | Markup and styling co-located; whole UI chunks copy-paste between projects |
| 5 | Your CSS stops growing | Utility reuse means CSS size grows logarithmically, not linearly |

## 2. Three reasons utilities beat inline styles

1. **Designing with constraints** : every utility resolves to a theme token; inline `style=""` accepts any magic number.
2. **Hover, focus, and other states** : inline styles cannot target `:hover`, `:focus`, `:disabled`; Tailwind variants can.
3. **Media queries** : inline styles cannot express `@media`; Tailwind responsive variants (`sm:`, `dark:`, `motion-safe:`) can.

## 3. The constraint-based design doctrine

ALWAYS treat the theme as the single source of truth for design decisions :

- v3 : `tailwind.config.js` `theme` (replace) vs `theme.extend` (additive)
- v4 : `@theme { --token: value; }` block in the main CSS file

Theme namespaces that drive utility generation (v4) :

| Namespace | Drives utilities |
|-----------|------------------|
| `--color-*` | `bg-*`, `text-*`, `border-*`, `ring-*`, `fill-*`, `stroke-*`, `accent-*`, `caret-*`, `decoration-*`, `outline-*`, `shadow-*` |
| `--font-*` | `font-{family}` |
| `--text-*` | `text-{size}` |
| `--font-weight-*` | `font-thin` to `font-black` |
| `--tracking-*` | `tracking-*` |
| `--leading-*` | `leading-*` |
| `--breakpoint-*` | `sm:`, `md:`, `lg:`, `xl:`, `2xl:` |
| `--container-*` | `@sm:`, `@md:` container queries + `max-w-*` |
| `--spacing` | All spacing utilities (single base unit, scaled by integer) |
| `--radius-*` | `rounded-*` |
| `--shadow-*` | `shadow-*` |
| `--ease-*` | `ease-*` |
| `--animate-*` | `animate-*` |

For the full design-system taxonomy (default values, P3 oklch palette, type scale, breakpoint defaults), see [tailwind-core-design-system].

## 4. The plain-text content scanner

Per https://tailwindcss.com/docs/detecting-classes-in-source-files :

> "Tailwind treats all of your source files as plain text, and doesn't attempt to actually parse your files as code in any way."

The scanner :
1. Tokenises files looking for substrings that COULD be valid class names.
2. Attempts CSS generation for each token.
3. Discards tokens that do not map to a known utility.

Consequences ALWAYS to enforce :

- A class must appear as a **literal complete token** in a scanned file to be emitted.
- String concatenation, template literals, and runtime composition break detection.
- Partial tokens are silently ignored : `` `bg-${color}-500` `` emits nothing because no file contains the literal token `bg-red-500`.

### Default scan boundaries (v4)

Files scanned :
- All source files in the project, EXCEPT :

Files explicitly excluded :
- Paths matching `.gitignore`
- `node_modules`
- Binary files (images, video, archives)
- CSS files
- Common package-manager lock files

### v3 vs v4 scan configuration

| Concern | v3 | v4 |
|---------|----|----|
| Discovery mode | Explicit globs in `content: []` | Auto-detect with exclusion rules |
| Add an extra path | Add to `content` array | `@source "../path"` |
| Exclude a path | NOT supported natively (rely on globs) | `@source not "../path"` |
| Safelist a literal | `safelist: ['bg-red-500']` or regex | `@source inline("bg-red-500")` |
| Safelist with patterns | `pattern: /bg-(red\|blue)-(100\|500\|900)/` | Brace expansion : `@source inline("bg-{red,blue}-{100,500,900}")` |
| Disable auto-detection | n/a | `@import "tailwindcss" source(none);` |
| Custom base path | n/a | `@import "tailwindcss" source("../src");` |

## 5. The three Tailwind layers

Tailwind uses three named cascade layers in the order `base → components → utilities`. v4 uses native CSS `@layer`; v3 uses synthetic ordering.

| Layer | What belongs here | Example |
|-------|-------------------|---------|
| `base` | Resets, typography defaults, `@font-face`, global CSS variables, html/body styling | `@layer base { h1 { @apply text-4xl font-bold } }` |
| `components` | Multi-class component shells, third-party-library style overrides | `@layer components { .btn { @apply px-4 py-2 rounded } }` |
| `utilities` | Custom single-purpose classes that participate in variant generation | `@layer utilities { .scrollbar-hidden { ... } }` (v3) ; `@utility scrollbar-hidden { ... }` (v4) |

Rules :

- ALWAYS place custom CSS inside one of the three layers.
- NEVER place custom CSS outside any `@layer` block unless you specifically want it to win over layered CSS regardless of source order.
- ALWAYS prefer `@utility name { ... }` over `@layer utilities { .name { ... } }` in v4 : `@utility` participates in the variant pipeline (responsive, hover, dark) automatically and is properly overridable by user-applied utilities.

## 6. `@source` directive variants (v4 only)

Per https://tailwindcss.com/docs/detecting-classes-in-source-files :

| Directive | Purpose | Example |
|-----------|---------|---------|
| `@source "path"` | Add a path to the scan set | `@source "../node_modules/@acmecorp/ui-lib"` |
| `@source not "path"` | Exclude a path from scanning | `@source not "../src/legacy"` |
| `@source inline("classes")` | Safelist literal class strings | `@source inline("underline")` |
| `@source inline("...")` with brace expansion | Generate Cartesian product of classes | `@source inline("{hover:,focus:,}bg-red-{50,{100..900..100},950}")` |
| `@source not inline("classes")` | Explicitly exclude classes | `@source not inline("bg-yellow-500")` |
| `@import "tailwindcss" source("path")` | Set custom base path for auto-detection | `@import "tailwindcss" source("../src")` |
| `@import "tailwindcss" source(none)` | Disable auto-detection entirely | `@import "tailwindcss" source(none)` |

ALWAYS use `@source inline()` over wide-net safelists : brace expansion gives precise control and avoids bloat.

## 7. Component-extraction triggers

Per https://tailwindcss.com/docs/styling-with-utility-classes (and corroborated in the utility-first docs), three triggers warrant moving from inline utilities to an extracted artefact :

1. **Three or more occurrences** of the exact same class combination across the codebase.
2. **A coherent product name** exists for the combination (Card, Button, BadgeWarning), so the abstraction is real.
3. **The duplication crosses file boundaries** : loops within a single file solve same-file repetition without extraction.

Extraction targets, ranked by preference :

1. **Component or template partial** in your framework of choice (React/Vue/Svelte/Solid/Astro/Blade/ERB).
2. **`@layer components { .name { @apply ... } }`** (v3) or **`@utility name { ... }`** (v4) when no component framework is available or when the artefact must be reachable from outside-the-build content (CMS, markdown).
3. **Custom CSS class via `var(--token)` references** for cases where `@apply` would lose too much override flexibility.

ALWAYS prefer option 1. NEVER reach for option 2 first.

## 8. v3 vs v4 architectural delta summary (non-breaking surface)

Architectural concepts that survive identically from v3 to v4 :

- Utility-first philosophy.
- Constraint-based design system as source of truth.
- Plain-text content scanning model.
- Three-layer cascade (`base`, `components`, `utilities`).
- Variant pipeline (responsive, state, group/peer, arbitrary).
- Plugin API (`addUtilities`, `addComponents`, `addBase`, `addVariant`, `matchUtilities`, `matchVariant`).

Concepts that **change syntax** but not philosophy in v4 (deferred to companion skills) :

- Theme expression : JS object → CSS variables in `@theme { }`
- Content discovery : `content: []` glob → auto-detect + `@source`
- Engine : JIT (JS) → Oxide (Rust + Lightning CSS)
- Authoring custom utilities : `addUtilities()` plugin → native `@utility` directive
- Import : 3x `@tailwind` directives → single `@import "tailwindcss"`

For the full breaking-change catalogue, see [tailwind-impl-migration-v3-v4] and [tailwind-errors-v4-migration].

## 9. When utility-first wins (decision criteria)

ALWAYS recommend Tailwind when at least three of these are true :

- A design system exists (or is being built) with tokens for colour, spacing, type.
- HTML/JSX is authored in a component framework or templating engine.
- Refactor safety matters more than visual one-offs.
- Bundle-size discipline is a goal.
- The team values rapid iteration over naming ceremony.
- Dark mode, responsive, and state-driven styling are needed across many elements.

## 10. When utility-first loses (anti-criteria)

NEVER recommend Tailwind as the primary styling layer when any of these dominate :

- Content authored by non-developers in a CMS WYSIWYG outside the build (use `@tailwindcss/typography` as a `.prose` shell only).
- Generative/brutalist/editorial designs that intentionally break a system.
- Email HTML (most clients strip `<style>` and reject `@layer`).
- A single static page with five rules of CSS where toolchain overhead exceeds benefit.

## 11. References

- https://tailwindcss.com/docs/utility-first (philosophy)
- https://tailwindcss.com/docs/styling-with-utility-classes (advanced syntax)
- https://tailwindcss.com/docs/detecting-classes-in-source-files (content scanning)
- https://tailwindcss.com/docs/adding-custom-styles (`@layer`, `@utility`)
- https://tailwindcss.com/docs/functions-and-directives (`@source`, `@theme`, `@reference`)
- https://tailwindcss.com/blog/tailwindcss-v4 (v4 architectural changes)
- https://v3.tailwindcss.com/docs/configuration (v3 config equivalent)

Verified 2026-05-19.
