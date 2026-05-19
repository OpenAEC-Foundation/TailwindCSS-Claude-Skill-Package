---
name: tailwind-errors-specificity
description: >
  Use when a Tailwind utility class is in the compiled CSS but does
  NOT visually override a component class or a third-party rule,
  when @apply produces output that does not behave like the utility
  applied to a real element, when a custom utility you authored
  inside CSS does not respect responsive or hover variants, when
  plugin-emitted classes get overridden by your own classes in
  unexpected ways, or when you reach for !important and want to know
  the right syntax for v3 vs v4. Prevents the unlayered-CSS trap
  (custom CSS written outside @layer wins over any utility because
  unlayered styles defeat the cascade), the @apply !important-strip
  trap (v3 silently removes !important from utilities pulled in via
  @apply), the v3-vs-v4-important syntax flip (v3 prefix !bg-red-500,
  v4 suffix bg-red-500!), the wrong-layer trap (custom CSS in @layer
  base when you wanted utility-level priority), the plugin-order
  trap (v3 plugins array : LAST entry wins when two plugins emit the
  same class), and the same-property in @apply trap (only the LAST
  applied utility for one CSS property survives).
  Covers the three Tailwind layers (base, components, utilities) and
  their ordering, where custom CSS must live to participate, @apply
  semantics + the !important rule, the !-prefix and !-suffix
  important-modifier syntax for both v3 and v4, declaration-order
  rules inside a single class, plugin-array ordering in v3 config,
  @utility directive in v4 vs @layer utilities, @reference for scoped
  styles (Vue/Svelte/CSS modules), and how to debug "this utility is
  in CSS but I don't see it apply".
  Keywords: tailwind specificity, @layer base, @layer components,
  @layer utilities, custom utility specificity, unlayered css,
  unlayered styles, important modifier, !important tailwind, v3
  important prefix, v4 important suffix, font-bold!, !font-bold,
  apply not working, @apply important, @apply stripped, plugin order
  v3, plugins array order, last plugin wins, declaration order
  tailwind, same property apply, cascade tailwind, my class is in css
  but not applied, utility does not override component, why is my
  background color not changing, why is my padding ignored,
  third-party css winning, override mui tailwind, override bootstrap
  tailwind, scoped style apply, @reference vue svelte, @utility
  directive specificity, conflict in apply, multiple plugins same
  class, tailwind cascade order, tailwind layer order, debug tailwind
  cascade
license: MIT
compatibility: "Designed for Claude Code. Requires Tailwind CSS v3.4 or v4.0+."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# Tailwind Specificity & Cascade Failures

Tailwind delegates ordering to native CSS `@layer` plus declaration-
order rules. Conflicts trace to FIVE distinct mechanisms :

1. Layer-level priority (base < components < utilities < unlayered).
2. Class-level specificity (single-class selectors -> 0,0,1,0).
3. Declaration order WITHIN one class (last property wins).
4. `!important` modifier (overrides cascade entirely).
5. Plugin loading order (when two plugins emit the same class).

This skill is the diagnostic flow for all five.

Companion skills :

- `tailwind-syntax-apply-directive` : `@apply` deep dive
- `tailwind-impl-plugins-custom` : authoring your own plugins
- `tailwind-impl-config-v3` and `tailwind-impl-config-v4` : where to
  declare custom utilities
- `tailwind-errors-build-failures` : when the class is NOT in CSS
- `tailwind-errors-runtime` : DOM-level issues (wrong attribute,
  React class vs className)

## Quick Reference : The Layer Order

Tailwind emits a native CSS `@layer` block. Browser cascade resolves
in this order (HIGHER wins) :

```
1. Unlayered styles                  (HIGHEST -> "naked" CSS)
2. @layer utilities                  (Tailwind utilities)
3. @layer components                 (Tailwind components, prose, btn)
4. @layer base                       (Preflight reset, typography defaults)
5. Browser defaults                  (LOWEST)
```

A utility (`bg-red-500`) ALWAYS beats a component (`prose`) which
ALWAYS beats base (h1 reset). Unlayered CSS (custom rules NOT inside
`@layer`) ALWAYS wins over any Tailwind output. That last rule is
the most common source of "my utility doesn't work".

## Decision Tree : "My Utility Is Not Applied"

```
Inspect the element in DevTools. Is the utility selector visible
with a strike-through ?
  ├── YES (something overrides it)
  │   ├── Override is in :hover, :focus, :active etc
  │   │   -> Add hover:/focus: variant to your utility
  │   ├── Override is from an unlayered CSS rule (e.g. ".my-card { padding: 20px; }")
  │   │   -> Move that rule into @layer components or remove it
  │   ├── Override is a plugin-emitted component (prose-*, form-*)
  │   │   -> Your utility is correct ; the component is winning
  │   │      because Tailwind v3 plugin classes register in the
  │   │      components layer. Add !-modifier OR a more specific
  │   │      utility variant.
  │   └── Override comes from a third-party CSS file (Bootstrap, etc.)
  │       -> Apply the !-important modifier OR move third-party CSS
  │          into a lower layer.
  │
  └── NO (selector not shown at all)
      ├── Class not in compiled CSS -> see tailwind-errors-build-failures
      ├── Class is in markup but DOM attribute is `class` in JSX (not className)
      │   -> Fix the JSX (use className)
      └── Class targets a pseudo-element you cannot style (e.g. ::-webkit-scrollbar)
          -> Use vendor-specific utility or raw CSS in @layer utilities
```

## CRITICAL : Unlayered CSS Defeats Tailwind

The most common failure. Custom CSS authored OUTSIDE any `@layer`
block wins over everything Tailwind emits.

### Wrong

```css
/* app.css */
@import "tailwindcss";

.card {
  padding: 1rem;
  background: white;
}
```

In markup :

```html
<div class="card p-8 bg-zinc-100">
```

`p-8` and `bg-zinc-100` are emitted in `@layer utilities`. The `.card`
rule is UNLAYERED. Native CSS cascade : unlayered > any layered. So
padding is `1rem` (from `.card`) NOT `2rem` (from `p-8`).

### Right (v3)

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer components {
  .card {
    padding: 1rem;
    background: white;
  }
}
```

### Right (v4)

```css
@import "tailwindcss";

@layer components {
  .card {
    padding: 1rem;
    background: white;
  }
}
```

Now `.card` lives in `@layer components`. `p-8` from `@layer utilities`
WINS as expected.

## CRITICAL : @apply Strips !important by Default (v3)

```css
.my-card {
  @apply !p-4;
}
```

This INTENTIONALLY does NOT preserve the !important flag in v3. The
output is `padding: 1rem;` not `padding: 1rem !important`. Per v3
docs : "By default `!important` flags are stripped to avoid
specificity conflicts."

### Force !important (v3)

```css
.my-card {
  @apply p-4 !important;
}
```

### v4 behaviour

v4 PRESERVES the important modifier through `@apply`. If you write :

```css
.my-card {
  @apply p-4!;
}
```

The output IS `padding: 1rem !important`.

## CRITICAL : Important Modifier Syntax Flipped Between v3 and v4

| Version | Syntax | Example |
|---------|--------|---------|
| v3 | `!`-PREFIX | `!font-bold`, `!bg-red-500`, `md:!text-xl` |
| v4 | `!`-SUFFIX | `font-bold!`, `bg-red-500!`, `md:text-xl!` |

Both produce `!important` in compiled CSS. The position changed
between major versions ; mixing them is the single most common
v3-to-v4 migration trap.

### Wrong in v4

```html
<div class="!font-bold">   <!-- v3 syntax, v4 ignores -->
```

### Right in v4

```html
<div class="font-bold!">
```

### Right in v3

```html
<div class="!font-bold">
```

Note for variant chains :

- v3 : `md:hover:!text-blue-500`
- v4 : `md:hover:text-blue-500!`

## CRITICAL : Plugin Order in v3 plugins Array

When TWO plugins emit the same class name, the LAST loaded plugin
wins (because its CSS is appended later in `@layer components` /
`@layer utilities`).

```js
// tailwind.config.js
module.exports = {
  plugins: [
    require("@tailwindcss/typography"),   // emits prose, prose-* (FIRST)
    require("custom-typography-plugin"),  // emits prose with overrides (LAST -> WINS)
  ],
};
```

ALWAYS list plugins in ASCENDING priority order : least-specific
first, most-specific last. If a third-party plugin must override your
custom plugin, load the third-party plugin LATER.

## CRITICAL : Declaration Order Inside @apply

Within ONE `@apply` line, conflicting CSS properties resolve by
DECLARATION ORDER, not specificity (specificity is the same: 0,0,1,0).

```css
.btn {
  @apply px-4 py-2 px-8;   /* px-8 wins ; px-4 silently lost */
}
```

Output :

```css
.btn {
  padding-left: 2rem;
  padding-right: 2rem;
  padding-left: 2rem;   /* duplicate, harmless */
  padding-right: 2rem;
}
```

The LAST `px-*` directive applied is the surviving padding-left/right.
ALWAYS audit `@apply` lines for the same-property-twice anti-pattern.

Note : utilities targeting DIFFERENT properties survive together :

```css
.btn {
  @apply px-4 py-2 bg-blue-600 text-white;   /* all four survive */
}
```

## CRITICAL : @layer utilities vs @utility (v4)

In v4 the PREFERRED way to add custom utilities is the `@utility`
directive ; `@layer utilities` still works but has subtle differences.

```css
/* v4 preferred */
@utility tab-4 {
  tab-size: 4;
}
```

vs

```css
@layer utilities {
  .tab-4 { tab-size: 4; }
}
```

Difference :

- `@utility` participates fully in variant pipeline (`hover:tab-4`,
  `md:tab-4`) and overrides components correctly.
- `@layer utilities` emits the rule but custom utilities authored
  this way do NOT always handle responsive variants reliably for
  complex CSS.

ALWAYS prefer `@utility` in v4 for new custom utilities.

## Decision Tree : Forcing Override Without !important

```
Can the layer be adjusted ?
  ├── My custom CSS rule -> Move into @layer components (lower priority)
  │                         OR move overriding utility into @layer utilities (higher)
  │
  ├── Third-party CSS I can edit -> Wrap in @layer base or @layer components
  │
  ├── Third-party CSS I CANNOT edit -> Use !-modifier as last resort
  │                                    (font-bold! in v4, !font-bold in v3)
  │
  └── Inline style attribute on the element -> CANNOT override with class
                                               (inline style wins)
                                               -> Apply utility WITH !-modifier
                                                  OR change the markup
```

## CRITICAL : Inline style Always Wins (Almost)

```html
<div style="padding: 10px" class="p-8">
```

Padding is `10px`. Class is ignored. ONE exception : utility with
!-modifier beats inline style.

```html
<div style="padding: 10px" class="p-8!">   <!-- v4 syntax, wins -->
<div style="padding: 10px" class="!p-8">   <!-- v3 syntax, wins -->
```

ALWAYS prefer fixing the markup. The !-modifier on every class is a
code smell.

## @reference for Scoped Styles (v4)

Vue `<style scoped>`, Svelte `<style>`, CSS modules, and MDX inline
styles process CSS in ISOLATION from `app.css`. `@apply` inside them
fails with "Cannot apply unknown utility class" because they have
NO theme context.

Fix : `@reference` imports the theme without duplicating CSS output.

```vue
<style>
  @reference "../../app.css";
  h1 { @apply text-2xl font-bold; }
</style>
```

OR (for default theme only) :

```vue
<style>
  @reference "tailwindcss";
  h1 { @apply text-2xl font-bold; }
</style>
```

ALWAYS use `@reference` for scoped stylesheets that need `@apply`.
See `tailwind-syntax-apply-directive` for full coverage.

## What This Skill Does NOT Cover

- Class is NOT in compiled CSS -> see
  `tailwind-errors-build-failures`.
- @apply patterns for component composition -> see
  `tailwind-syntax-apply-directive`.
- Custom utility/component design via plugins -> see
  `tailwind-impl-plugins-custom`.
- prefix / important: config flags (v3) -> see
  `tailwind-impl-config-v3`.
- CSS variables / @theme tokens -> see `tailwind-impl-config-v4`.

## References

- `references/methods.md` : every layer + directive that affects
  specificity, full !-modifier syntax for both versions, plugin-order
  rules.
- `references/examples.md` : real cases with DevTools-level output.
- `references/anti-patterns.md` : unlayered CSS, wrong-version
  important syntax, @apply !important strip, plugin-order mistakes.

## Official Source Links

- v4 functions and directives : https://tailwindcss.com/docs/functions-and-directives
- v3 functions and directives : https://v3.tailwindcss.com/docs/functions-and-directives
- v4 styling guide (important) : https://tailwindcss.com/docs/styling-with-utility-classes
