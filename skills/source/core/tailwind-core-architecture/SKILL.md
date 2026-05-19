---
name: tailwind-core-architecture
description: >
  Use when starting a new Tailwind CSS project, evaluating whether Tailwind fits
  a codebase, debating utility-first vs BEM / CSS Modules / CSS-in-JS, or
  reviewing a PR that mixes architectural styles.
  Prevents the "ad-hoc CSS leaking into utility-first markup" failure mode,
  premature `@apply` extraction, and dynamic-class anti-patterns that break
  the content scanner.
  Covers utility-first philosophy, atomic CSS tradeoffs, the design-system
  constraint model, how Tailwind scans source files for classes, when
  utility-first wins, when it loses, and a decision tree for adoption.
  Both Tailwind v3.4 and v4 share this architecture; engine details (JIT vs
  Oxide) live in tailwind-core-v3-vs-v4.
  Keywords: utility-first, atomic CSS, design tokens, content scanning,
  source detection, constraint-based design, when to use Tailwind,
  should I use Tailwind, Tailwind vs BEM, Tailwind vs CSS Modules,
  Tailwind vs CSS-in-JS, utility soup, premature extraction,
  no styles showing up, classes not generated, blank page after build,
  dynamic class names, template literals not working,
  CMS content not styled, prose plugin, design system in markup,
  philosophy, architecture decision, framework selection.
license: MIT
compatibility: "Designed for Claude Code. Requires Tailwind CSS v3.4 or v4.0+."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# Tailwind CSS : Core Architecture

Foundational mental model for Tailwind CSS. Every other skill in this package assumes the reader has internalised the philosophy and content-scanning model documented here.

## Quick Reference

### Tailwind in one sentence

ALWAYS describe Tailwind as : a **utility-first**, **constraint-based**, **content-scanned** CSS framework where every utility class maps to a token in a centrally defined design system and the build step emits only the utilities found as **literal complete tokens** in source files.

### Four invariants

1. ALWAYS treat the theme (`tailwind.config.js` in v3, `@theme { }` in v4) as the **single source of design truth**. Colours, spacing, typography, breakpoints, shadows, radii belong there, never inlined as arbitrary values for system-level decisions.
2. ALWAYS write classes as **literal complete tokens** in source. NEVER assemble class names with string concatenation or template literals at runtime.
3. ALWAYS compose multiple single-purpose utilities directly in markup. NEVER reach for `@apply` to "clean up" utility lists; extract a component instead when the same combination appears 3+ times.
4. ALWAYS rely on the cascade order Tailwind emits (`base` then `components` then `utilities`). NEVER fight specificity by re-declaring utility properties in custom CSS.

### When Tailwind WINS

ALWAYS recommend Tailwind for :
- Design-system-driven projects where tokens (colour, spacing, type) are agreed up-front.
- Component-rendered UIs (React, Vue, Svelte, Solid, Astro, server-side templating).
- Greenfield projects where bundle-size discipline matters from day one.
- Teams that already use Figma or any token-based design tool.
- Rapid prototyping where naming and BEM ceremony slow iteration.

### When Tailwind LOSES

NEVER recommend Tailwind as the primary styling layer when :
- The bulk of HTML is authored by non-developers in a CMS, blog editor, or Markdown pipeline outside the build (use `@tailwindcss/typography` for the `.prose` shell, but accept that body content is unstyled by utilities).
- Designs intentionally violate a design system on a per-element basis (one-off art pieces, generative graphics, brutalist editorial layouts).
- The deliverable is a single static page with five rules of CSS; the toolchain overhead exceeds the win.
- Email HTML: most clients strip `<style>` and reject modern CSS that Tailwind generates.

## Decision Tree 1 : Should this project use Tailwind?

```
Q1. Is HTML/JSX authored mostly by developers?
    no  → use a CMS-friendly tradition (BEM + global tokens, or `prose`-shell only) and STOP
    yes → Q2

Q2. Does the project have, or want, a documented design system (tokens for colour, spacing, type)?
    no  → ad-hoc CSS or CSS Modules is fine; Tailwind without a design system becomes utility soup with no guardrails
    yes → Q3

Q3. Will the codebase live longer than 6 months or grow beyond one developer?
    no  → ANY approach works; pick what the author already knows
    yes → Q4

Q4. Is the runtime constrained (no JavaScript at all, email, AMP-restricted)?
    yes → Tailwind is fine for compile-time CSS but skip CSS-in-JS-style helpers (no runtime cost anyway in v3/v4)
    no  → ALWAYS recommend Tailwind
```

## Decision Tree 2 : Utility-first vs the alternatives

```
                           ┌─ Utility-first        : single-purpose, atomic, theme-bound
                           │   wins for : design systems, component UIs, refactor safety
                           │
                           ├─ BEM                   : structured naming, no design system enforced
                           │   wins for : legacy stylesheets, design teams without tokens
                           │
Styling strategies ────────┼─ CSS Modules           : locally scoped class names, free-form CSS
                           │   wins for : per-component bespoke graphics, CSS-heavy pages
                           │
                           ├─ CSS-in-JS (runtime)   : dynamic-from-props, theme via context
                           │   wins for : truly runtime values; loses bundle-size and SSR cost
                           │
                           └─ Inline styles         : magic numbers, no states, no media queries
                               wins for : truly one-off, runtime-only, non-themable values
```

ALWAYS rank by the order above for a new project with a design system. NEVER mix utility-first with CSS-in-JS for the same component family : the cascading rules become ambiguous and the bundle pays for both engines.

## Decision Tree 3 : When to extract a component

```
Q1. Has this exact class combination appeared 3+ times in the codebase?
    no  → ALWAYS leave the utilities inline
    yes → Q2

Q2. Is the duplication inside a single file (loops, lists)?
    yes → ALWAYS solve with a template loop ({list.map(item => ...)}); no extraction needed
    no  → Q3

Q3. Does the duplicated UI have a coherent name in product vocabulary (Card, Button, BadgeWarning)?
    no  → wait until the name emerges; premature extraction freezes the wrong abstraction
    yes → Q4

Q4. Does the team use a component framework (React/Vue/Svelte/Solid/Astro) or template engine?
    yes → ALWAYS extract as a component or template partial. NEVER reach for `@apply` first.
    no  → consider `@layer components { .btn-primary { @apply ... } }` as a last resort, accept the cost: utilities placed on the element can no longer override the component's properties without `!`.
```

## Decision Tree 4 : Content scanning sanity check

```
Q1. Is every class name in the source a literal complete token (no `bg-${color}-500`)?
    no  → fix this FIRST. See [tailwind-errors-dynamic-classes].
    yes → Q2

Q2. Are class names defined in node_modules (component library) part of the bundle?
    no  → ALWAYS add `@source "../node_modules/@org/pkg"` (v4) or
          extend `content: []` (v3) to scan them
    yes → Q3

Q3. Does any framework introduce bracketed path segments (`[...slug]`, `[id]`)?
    yes → ALWAYS escape the glob ; v4 `@source './[[]**[]]/**/*.{js,ts,jsx,tsx,mdx}'`
    no  → Q4

Q4. Are classes constructed in build-time string concatenation (Vue/Svelte attribute binding)?
    yes → static-map them, OR add `@source inline("class-list")` (v4) / `safelist` (v3)
    no  → content scan is healthy
```

## Patterns

### Pattern 1 : Utility-first is constraint-based, not value-free

The official docs frame the rationale as **Designing with constraints**. Utilities like `bg-blue-500`, `text-xl`, `p-4` resolve to **theme variables**, so picking a class is picking a value from a vetted, finite design system rather than introducing a magic number. Inline `style="padding: 17px"` defeats this : 17px is unbounded; `p-4` is bounded to the spacing scale.

ALWAYS treat arbitrary values (`p-[17px]`, `bg-[#1da1f2]`) as **escape hatches**, never the default. NEVER reach for an arbitrary value when the next-step token (`p-4`, `bg-sky-500`) is within visual tolerance. When you do need an arbitrary value system-wide, ALWAYS add a token to `@theme` instead.

### Pattern 2 : Atomic CSS = single property per class, max

Each Tailwind utility has a **single responsibility** (`mt-4` sets `margin-top`, `text-center` sets `text-align`). Composition happens **in markup**, not in the stylesheet. This is the opposite of BEM blocks where one class can set 20 properties.

Tradeoffs vs alternatives :

| Concern | Tailwind utilities | BEM | CSS Modules | CSS-in-JS |
|---------|--------------------|-----|-------------|-----------|
| Naming overhead | none (classes already exist) | high (block-element-modifier) | medium (local scope) | medium (component-named) |
| Specificity wars | impossible (all utilities same specificity) | medium | low | low |
| Tree-shaking | automatic (content scan) | manual (delete unused rules) | automatic (per-import) | automatic (per-component) |
| Runtime cost | zero (build-time CSS) | zero | zero | medium-to-high |
| SSR cost | zero | zero | zero | non-zero (style serialisation) |
| Refactor safety | local (class change affects only that element) | risky (selector reuse) | local | local |
| Design-system enforcement | strong (theme is source of truth) | weak (convention) | weak | medium (theme prop) |
| Learning curve | medium (utility vocabulary) | low | low | medium |

ALWAYS choose Tailwind when **design-system enforcement** and **refactor safety** are the load-bearing goals.

### Pattern 3 : Content scanning is a plain-text token scan

Per the official source-detection docs : "Tailwind treats all of your source files as plain text, and doesn't attempt to actually parse your files as code in any way." It looks for tokens that match potential class names, then attempts CSS generation for each.

Consequences :
- ALWAYS expect a class to be generated ONLY if it appears verbatim somewhere in a scanned file.
- NEVER construct class names with template literals : `` `bg-${color}-500` `` is never a literal token; the class is never emitted.
- ALWAYS verify the scan picks up the path : in v4 the default is "everything except `.gitignore`'d paths, `node_modules`, binary files, lock files, and CSS files"; in v3, the `content: []` glob is authoritative.
- ALWAYS add explicit `@source` (v4) or `content` entries (v3) for class names that live inside `node_modules` packages.

For dynamic palettes that genuinely need runtime values, ALWAYS use the static-map fix or v4's `@source inline("...")` safelist with brace expansion. See [tailwind-errors-dynamic-classes] for the full anti-pattern + four-tier fix.

### Pattern 4 : The three Tailwind layers and what belongs where

Tailwind emits CSS in three named layers (native CSS `@layer` in v4, synthetic in v3) :

| Layer | Purpose | What you put here | What you NEVER put here |
|-------|---------|-------------------|-------------------------|
| `base` | Resets, typography defaults, `@font-face`, CSS variables | `html { scroll-behavior: smooth }`, custom `@font-face` rules, base typography | Component classes, utilities |
| `components` | Multi-class component classes (`.card`, `.btn`) | `.btn { @apply px-4 py-2 ... }`, third-party reset styles | Utilities, base styles |
| `utilities` | Single-purpose classes | Custom utilities authored with `@layer utilities` (v3) or `@utility` (v4) | Multi-property classes, component shells |

ALWAYS keep custom CSS inside one of these three layers. CSS placed outside any layer becomes "unlayered" and wins over layered CSS regardless of source order : this is occasionally desirable for true overrides but is otherwise a specificity trap.

### Pattern 5 : Markup tells the story; the theme constrains it

Read this Tailwind markup as a sentence : "a card with horizontal padding 4, vertical padding 6, white background, rounded corners large, shadow medium, on hover lift slightly via shadow-lg".

```html
<div class="px-4 py-6 bg-white rounded-lg shadow-md hover:shadow-lg transition-shadow">
  ...
</div>
```

ALWAYS treat the class list as **declarative composition**. NEVER mentally translate to imperative CSS while reading; that is the skill the framework removes.

### Pattern 6 : v3 vs v4 architecture deltas (philosophy unchanged)

The utility-first doctrine, content-scan model, layer system, and design-token enforcement are **identical** between v3 and v4. What changed in v4 is **how the theme is expressed** (CSS variables in `@theme { }` instead of a JS object), **how content is discovered** (auto-detect with `@source` overrides instead of explicit `content: []`), and **how custom utilities are authored** (native `@utility` directive alongside the v3 `plugin()` API). For engine-level deltas (JIT vs Oxide, performance, browser baseline) and the full breaking-change catalogue, see [tailwind-core-v3-vs-v4] and [tailwind-impl-migration-v3-v4]. This skill is version-agnostic; every claim here applies to both.

## Anti-patterns (summary; full list in references/anti-patterns.md)

NEVER do these :

1. NEVER assemble class names with string concatenation or template literals : the content scanner cannot detect them. Fix : static map or `@source inline()`.
2. NEVER `@apply` utility classes into a generic stylesheet just to "clean up markup". Extract a component instead; `@apply` defeats the override model.
3. NEVER reach for arbitrary values (`p-[17px]`) when a scale token (`p-4`) is within visual tolerance; add to `@theme` if system-wide.
4. NEVER mix Tailwind with a runtime CSS-in-JS engine for the same component tree : both compete for cascade ordering, both inflate the bundle.
5. NEVER store design decisions (brand colours, spacing scale) inline as arbitrary values across the codebase; centralise in the theme.
6. NEVER write a custom utility outside `@layer utilities` (v3) or `@utility` (v4) : the resulting CSS does not participate in variant generation.
7. NEVER assume `sm:` means "small screens only" : Tailwind breakpoints are mobile-first and apply **from the breakpoint upward**. See [tailwind-syntax-responsive].
8. NEVER ship Tailwind to email HTML : most clients strip `<style>` and reject `@layer`.

## Reference Links

- [references/methods.md](references/methods.md) : architecture concepts catalogue, full content-scan rules, layer model, @source directive variants
- [references/examples.md](references/examples.md) : utility-first vs BEM vs CSS Modules vs CSS-in-JS side-by-side, component extraction examples
- [references/anti-patterns.md](references/anti-patterns.md) : full anti-pattern catalogue with WHY each fails

## Cross-references

- [tailwind-core-design-system](../tailwind-core-design-system/SKILL.md) : the spacing, colour, type, and breakpoint scales the architecture constrains you to
- [tailwind-core-v3-vs-v4](../tailwind-core-v3-vs-v4/SKILL.md) : engine internals (JIT vs Oxide), config-location shift, breaking-change catalogue
- [tailwind-syntax-utility-classes](../../syntax/tailwind-syntax-utility-classes/SKILL.md) : the full utility vocabulary
- [tailwind-errors-dynamic-classes](../../errors/tailwind-errors-dynamic-classes/SKILL.md) : the dynamic-class anti-pattern + four-tier fix
- [tailwind-impl-apply-directive](../../impl/tailwind-impl-apply-directive/SKILL.md) : `@apply`, `@layer`, and the v4 `@reference` scoped-stylesheet trap
- [tailwind-impl-tailwind-merge](../../impl/tailwind-impl-tailwind-merge/SKILL.md) : conflict resolution for runtime class composition

## Sources

All claims in this skill trace to URLs in `SOURCES.md` :

- https://tailwindcss.com/docs/utility-first : philosophy, four benefits, design-tokens-vs-magic-numbers
- https://tailwindcss.com/docs/styling-with-utility-classes : advanced patterns, component extraction
- https://tailwindcss.com/docs/detecting-classes-in-source-files : content-scan mechanism, `@source` variants
- https://tailwindcss.com/docs/adding-custom-styles : `@layer`, `@utility`, when to extract
- https://tailwindcss.com/blog/tailwindcss-v4 : v4 architecture deltas (engine, CSS-first config)
- https://v3.tailwindcss.com/docs/utility-first : v3 philosophy (identical doctrine)

Verified 2026-05-19.
