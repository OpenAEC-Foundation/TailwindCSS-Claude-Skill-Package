---
name: tailwind-syntax-variants
description: >
  Use when applying conditional styles in Tailwind: hover, focus, active, dark mode,
  responsive breakpoints, group-* and peer-* combinators, aria-* and data-* attribute
  variants, has-* and not-* selectors, position-in-parent variants (first, last, *,
  nth-*), or arbitrary [&...] variants. Use also when stacking multiple variants
  (`md:hover:dark:bg-blue-500`) and the resulting selector does not match the intended
  element. Use also when migrating variant chains from Tailwind v3 to v4: the stacking
  order flipped from right-to-left to left-to-right and is the #1 silent breakage
  during upgrade.
  Prevents the common mistakes of (a) reading a v3 stack like `first:*:pt-0` and
  expecting the same compiled selector in v4 (it is now `*:first:pt-0`), (b) writing
  hover variants that mysteriously stop working on touch devices in v4 (hover is now
  gated on `@media (hover: hover)`), (c) writing `!flex` important syntax that fails
  silently in v4 (the bang moved to the trailing position: `flex!`), (d) using
  unscoped `group-hover:` when a named group `group/sidebar` is required for nested
  group hierarchies, (e) trying `not-hover:opacity-75` in v3 (`not-*` is v4-only), and
  (f) registering custom variants via `addVariant()` in a v4 project (use `@custom-variant`
  instead).
  Covers every variant family in v3.4 and v4: pseudo-class (hover, focus, focus-within,
  focus-visible, active, visited, target, first, last, odd, even, nth-*, first-of-type,
  empty, disabled, enabled, checked, indeterminate, required, valid, invalid, autofill,
  read-only, open, inert), pseudo-element (before, after, placeholder, file, marker,
  selection, first-line, first-letter, backdrop, details-content, popover-open),
  media-query (sm, md, lg, xl, 2xl, max-*, min-[Npx]:, max-[Npx]:, dark, light,
  motion-safe, motion-reduce, contrast-more, contrast-less, forced-colors, portrait,
  landscape, print, screen, ltr, rtl), feature-query (supports-*, not-supports-*),
  attribute (aria-* and aria-[...], data-* and data-[...]), combinator (group-*,
  peer-*, named groups `group/name`, named peers `peer/name`, has-*, not-*, in-*,
  starting:*, group-has-*, peer-has-*), position-in-parent (*, **, *:first, *:last,
  *:odd, *:even, nth-N, nth-[3n+1]), and arbitrary variants ([&:nth-child(3)],
  [&_p], [@media(...)], [@supports(...)]). Includes the v4 stacking-order flip with
  migration examples, the `@custom-variant` directive (v4) vs `addVariant()` plugin
  (v3), the trailing-bang important syntax change, and the hover-on-touch gate.
  Keywords: variant, hover, focus, active, dark mode, group-hover, peer-checked,
  named group, group/sidebar, peer/name, aria-checked, data-state, has-checked,
  not-hover, in-focus, starting, first child, last child, nth-child, odd even,
  before after, placeholder, file input, marker, selection, backdrop, motion-reduce,
  print only, rtl, arbitrary variant, stacking order, left to right, right to left,
  variant order flipped, why does my hover not work on mobile, important modifier,
  bang flex, flex bang, custom variant, addVariant, @custom-variant, supports query,
  feature query, why does first:*: not work in v4.
license: MIT
compatibility: "Designed for Claude Code. Requires Tailwind CSS v3.4 or v4.0+."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# Tailwind CSS Syntax: Variants

Variants are the conditional prefixes that turn a flat utility (`bg-blue-500`) into a state-aware rule (`hover:bg-blue-500`, `md:dark:hover:bg-blue-500`). Tailwind defines around 100 built-in variants spread across ten families plus arbitrary `[&...]` escape hatches. Master this and you master 90% of real-world Tailwind markup.

## Quick Reference

### Variant families at a glance

| Family : signature : v3 : v4 |
|--|
| Pseudo-class : `hover:`, `focus:`, `active:` : same : same (but `hover:` gated on `@media (hover: hover)`) |
| Pseudo-element : `before:`, `placeholder:` : same : same (+ `details-content:`, recognises `popover-open`) |
| Media query : `sm:`, `dark:`, `motion-reduce:`, `print:` : same : same (custom breakpoints via `--breakpoint-*` instead of `theme.screens`) |
| Feature query : `supports-[display:grid]:` : same : same + `not-supports-*:` |
| Attribute : `aria-checked:`, `data-[state=open]:` : same : same |
| Combinator : `group-*`, `peer-*`, named `group/sidebar` : same : same + `has-*`, `not-*`, `in-*`, `peer-has-*`, `group-has-*` |
| Position-in-parent : `first:`, `last:`, `odd:`, `*:` : same | same + parameterised `nth-[3n+1]:`, `nth-last-*:`, `**` (deep descendants) |
| Arbitrary : `[&:nth-child(3)]:`, `[@media(...)]:` : same : same (parens for CSS-vars : `[&_p]:bg-(--brand)`) |
| Important modifier : leading `!flex` (v3) : same (v3) : trailing `flex!` (v4) |
| Custom variant : `addVariant()` plugin (v3) : JS plugin | `@custom-variant name (&:selector);` CSS directive |

### Top 3 most-used patterns

```html
<!-- 1. State + responsive + dark stack (works v3 + v4) -->
<button class="bg-blue-600 hover:bg-blue-700 dark:bg-blue-500 dark:hover:bg-blue-400 md:px-8">
  Save
</button>

<!-- 2. Named group for nested cards (works v3 + v4) -->
<article class="group/card hover:shadow-lg">
  <h3 class="text-slate-900 group-hover/card:text-blue-600">Title</h3>
  <button class="group/edit invisible group-hover/card:visible">
    <span class="group-hover/edit:underline">Edit</span>
  </button>
</article>

<!-- 3. Arbitrary variant for one-off selectors -->
<ul class="[&_li]:list-disc [&_li:last-child]:mb-0">
  <li>One</li><li>Two</li>
</ul>
```

## Decision Trees

### Picking the right variant family

```
state I care about?
├── interaction state (hover, focus, active, visited, target)
│   └── pseudo-class : `hover:`, `focus:`, ...
├── form/input state (checked, indeterminate, disabled, required, invalid, read-only)
│   └── pseudo-class : `checked:`, `disabled:`, `invalid:`, ...
├── position in parent (first child, last child, odd, every-3rd)
│   ├── known position : `first:`, `last:`, `odd:`, `even:`
│   └── arbitrary position : `nth-[3n+1]:` (v4) or `[&:nth-child(3n+1)]:` (v3 + v4)
├── parent or sibling state?
│   ├── one level of nesting : `group-hover:`, `peer-checked:`
│   ├── multiple nested groups : NAMED `group/sidebar` + `group-hover/sidebar:`
│   └── ancestor-with-attribute without `group` class : `in-[role=tabpanel]:` (v4)
├── presence of descendant with a specific state?
│   └── `has-[input:checked]:` (v4), or `group-has-checked:` if going up first
├── negate any other variant?
│   ├── v4 : `not-hover:`, `not-supports-grid:`
│   └── v3 : NOT available ; use arbitrary `[&:not(:hover)]:`
├── HTML attribute value match (aria-* or data-*)?
│   ├── known value : `aria-checked:`, `data-active:`
│   └── arbitrary : `aria-[sort=ascending]:`, `data-[state=open]:`
├── browser capability test?
│   └── `supports-[display:grid]:`, `not-supports-[backdrop-filter]:` (v4)
├── viewport size or device preference?
│   ├── breakpoint : `sm:`, `md:`, `max-md:`, `min-[400px]:`
│   ├── color scheme : `dark:`, `light:`
│   ├── motion : `motion-reduce:`, `motion-safe:`
│   ├── contrast : `contrast-more:`, `contrast-less:`
│   ├── pointer : `pointer-fine:`, `pointer-coarse:` (v4)
│   ├── reading direction : `ltr:`, `rtl:`
│   └── medium : `print:`, `screen:`
└── nothing built-in fits?
    └── arbitrary variant : `[&:nth-child(odd)]:`, `[@media(prefers-reduced-data:reduce)]:`
```

### Picking direction for stacking variants (CRITICAL)

```
Tailwind major version?
├── v3.4 : read RIGHT-TO-LEFT
│   └── `first:*:pt-0` means : direct children (`*`), then first one (`first`)
│       └── selector : `:first-child > *` (incorrect intuition warning)
└── v4.0+ : read LEFT-TO-RIGHT
    └── `*:first:pt-0` means : direct children (`*`), then first one (`first`)
        └── selector : `> :first-child` (matches CSS reading order)
```

The order **flipped** between v3 and v4. The upgrade tool `npx @tailwindcss/upgrade` catches most cases, but bespoke stacks involving `*:`, structural variants, or arbitrary `[&...]` need manual review.

### Picking how to add a custom variant

```
project Tailwind major version?
├── v3.4 (JavaScript plugin)
│   └── `plugin(({ addVariant }) => { addVariant('third', '&:nth-child(3)') })`
└── v4.0+ (CSS directive)
    └── `@custom-variant third (&:nth-child(3));`
```

`addVariant()` from v3 is REMOVED in v4. If you encounter a v3 plugin shipping `addVariant` calls, convert each one into a `@custom-variant` directive during migration.

## Patterns

### Pattern 1: Stacking variants (v3 vs v4)

This is the most consequential v3-to-v4 change. NEVER mix the two mental models.

| v3.4 right-to-left | v4.0+ left-to-right |
|--------------------|----------------------|
| ```html
<!-- v3 syntax -->
<ul class="py-4 first:*:pt-0 last:*:pb-0">
  <li>One</li>
  <li>Two</li>
</ul>
<!-- Read right-to-left :
     pt-0 -> apply to first -> then to direct children (*)
     Compiles to : ul > :first-child { padding-top: 0; }
-->
``` | ```html
<!-- v4 syntax -->
<ul class="py-4 *:first:pt-0 *:last:pb-0">
  <li>One</li>
  <li>Two</li>
</ul>
<!-- Read left-to-right :
     direct children (*) -> first one -> pt-0
     Compiles to : ul > :first-child { padding-top: 0; }
-->
``` |

Both produce the same CSS, but the WRITTEN order is reversed. The migration tool covers `first:*:`, `last:*:`, `odd:*:`, `even:*:`. Manually inspect any handwritten stacks containing `*:`, `**:`, or arbitrary `[&...]` after a long-chained variant.

### Pattern 2: Named groups for nested hierarchies

Unnamed `group-hover:` always targets the NEAREST `.group` ancestor. When components nest groups, this becomes ambiguous. Named groups make the binding explicit.

```html
<!-- BROKEN : both inner and outer use unnamed group ; group-hover ambiguity -->
<article class="group">
  <button class="group">
    <span class="group-hover:underline">Which group?</span>
  </button>
</article>

<!-- FIXED : named groups -->
<article class="group/card hover:shadow-lg">
  <button class="group/edit">
    <span class="group-hover/edit:underline">Hovering the button</span>
    <span class="group-hover/card:text-blue-600">Hovering the card</span>
  </button>
</article>
```

The same pattern applies to peers : `<input class="peer/email">` then `<p class="peer-invalid/email:text-red-500">`.

### Pattern 3: Attribute variants (aria-* and data-*)

```html
<!-- ARIA boolean states (built-in shortcuts) -->
<div aria-checked="true" class="aria-checked:bg-sky-100">Checkbox row</div>
<button aria-pressed="true" class="aria-pressed:ring-2">Toggle</button>
<button aria-disabled="true" class="aria-disabled:opacity-50">Disabled-styled</button>

<!-- ARIA arbitrary values (any attribute value) -->
<th aria-sort="ascending" class="aria-[sort=ascending]:bg-[url('/down-arrow.svg')]">Name</th>

<!-- data-* known shortcuts (active, state, etc.) -->
<div data-active class="data-active:border-purple-500">Selected</div>

<!-- data-* arbitrary values (most common with Radix UI / Headless UI / state machines) -->
<div data-state="open" class="data-[state=open]:rotate-180">Chevron</div>
<div data-side="bottom" class="data-[side=bottom]:slide-in-from-top-2">Tooltip</div>
```

These are essential when integrating Radix UI, Headless UI, or any framework that sets `data-state="open|closed"`, `data-side="top|right|bottom|left"`, or similar attributes.

### Pattern 4: Position-in-parent variants

```html
<!-- Direct children (* prefix) : style only DIRECT children -->
<ul class="*:rounded-full *:border *:px-2 *:py-1">
  <li>Pill</li>
  <li>Pill</li>
</ul>

<!-- Deep descendants (** prefix) : v4 only -->
<ul class="**:data-avatar:size-12">
  <li><img data-avatar src="..." /></li>
  <li><img data-avatar src="..." /></li>
</ul>

<!-- Structural pseudo-classes -->
<ul>
  <li class="first:pt-0 last:pb-0 only:py-0">...</li>
  <li class="odd:bg-white even:bg-slate-50">...</li>
</ul>

<!-- nth-child shortcuts (v4) -->
<div class="nth-3:underline">Third</div>
<div class="nth-[3n+1]:underline">Every 3rd starting at 1</div>
<div class="nth-last-2:underline">Second from end</div>
<div class="nth-of-type-1:font-bold">First of its type</div>
```

### Pattern 5: Has, Not, In selectors (v4)

```html
<!-- has-* : style a parent based on a descendant's state -->
<label class="has-checked:bg-indigo-50 has-checked:ring-2">
  <input type="radio" name="plan" />
  Standard plan
</label>

<!-- not-* : negate any variant -->
<button class="opacity-75 not-hover:opacity-50">
  Default 50%, hover 75%
</button>
<div class="not-supports-[display:grid]:flex">
  Falls back to flex when grid not supported
</div>

<!-- in-* : implicit group (no `.group` class required on ancestor) -->
<button class="text-slate-500 in-[role=tablist]:text-slate-900">
  Highlights when inside a tablist
</button>
```

All three are v4-only. In v3, replace with arbitrary variants : `has-checked:` becomes `[&:has(:checked)]:`, `not-hover:` becomes `[&:not(:hover)]:`, `in-[role=tablist]:` becomes `[[role=tablist]_&]:`.

### Pattern 6: Custom variants (v3 vs v4)

| v3.4 : JavaScript plugin | v4.0+ : CSS `@custom-variant` |
|---------------------------|------------------------------|
| ```js
// tailwind.config.js
const plugin = require('tailwindcss/plugin')
module.exports = {
  plugins: [
    plugin(function ({ addVariant }) {
      addVariant('third', '&:nth-child(3)')
      addVariant('hocus', ['&:hover', '&:focus'])
      addVariant('supports-grid', '@supports (display: grid)')
    }),
  ],
}
``` | ```css
@import "tailwindcss";

@custom-variant third (&:nth-child(3));
@custom-variant hocus (&:hover, &:focus);

@custom-variant supports-grid {
  @supports (display: grid) { @slot; }
}
``` |

Then use `third:bg-blue-500`, `hocus:underline`, `supports-grid:grid` in markup. The two syntaxes produce equivalent compiled output.

### Pattern 7: Arbitrary variants (escape hatch, both versions)

When no built-in variant fits, the `[&...]` arbitrary syntax accepts any CSS selector or at-rule.

```html
<!-- Custom class selector -->
<li class="[&.is-dragging]:cursor-grabbing">Drag-aware</li>

<!-- Descendant selector -->
<article class="[&_p]:text-slate-600 [&_h2]:text-2xl">
  Auto-styles every p and h2 inside
</article>

<!-- Sibling selector -->
<input class="peer [&~p]:hidden focus:[&~p]:block" />

<!-- Custom at-rule -->
<div class="[@media(prefers-reduced-data:reduce)]:bg-none">No bg-image on slow networks</div>
<div class="[@container(width>=600px)]:grid">Container-query escape hatch</div>

<!-- Stacked with built-in variants -->
<li class="[&.is-dragging]:active:cursor-grabbing">Stacks fine</li>
```

The `&` token represents the styled element. Use `_` to mean a space in CSS selectors (`[&_p]` => `& p`).

## Reference Links

- `references/methods.md` : complete variant catalog (every pseudo-class, pseudo-element, media query, feature query, attribute variant, combinator) with v3 vs v4 availability columns.
- `references/examples.md` : working code examples for every family + arbitrary variants + nested groups + Radix data-state patterns.
- `references/anti-patterns.md` : the stacking-order flip mistake, hover-on-touch surprise, leading-vs-trailing bang, ambiguous groups, v4-only variants in v3 projects.

## Verified Sources

- https://tailwindcss.com/docs/hover-focus-and-other-states (v4 variant system + `@custom-variant`)
- https://v3.tailwindcss.com/docs/hover-focus-and-other-states (v3 variant system + `addVariant()`)
- https://tailwindcss.com/docs/upgrade-guide (section 14, stacking-order flip + trailing bang + hover gate)

Last verified : 2026-05-19 (Tailwind CSS v4.x current).
