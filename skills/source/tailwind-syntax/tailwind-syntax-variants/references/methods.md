# Tailwind CSS Variants: Complete Catalog

Verified 2026-05-19 against tailwindcss.com/docs/hover-focus-and-other-states (v4) and v3.tailwindcss.com/docs/hover-focus-and-other-states (v3).

## Pseudo-class Variants

| Variant | Compiles to | v3 | v4 |
|---------|-------------|----|----|
| `hover:*` | `&:hover` | yes (always) | yes (gated on `@media (hover: hover)`) |
| `focus:*` | `&:focus` | yes | yes |
| `focus-within:*` | `&:focus-within` | yes | yes |
| `focus-visible:*` | `&:focus-visible` | yes | yes |
| `active:*` | `&:active` | yes | yes |
| `visited:*` | `&:visited` | yes | yes |
| `target:*` | `&:target` | yes | yes |
| `first:*` | `&:first-child` | yes | yes |
| `last:*` | `&:last-child` | yes | yes |
| `only:*` | `&:only-child` | yes | yes |
| `odd:*` | `&:nth-child(odd)` | yes | yes |
| `even:*` | `&:nth-child(even)` | yes | yes |
| `nth-N:*` | `&:nth-child(N)` | no (use arbitrary) | yes |
| `nth-[expr]:*` | `&:nth-child(expr)` | yes (arbitrary only) | yes |
| `nth-last-N:*` | `&:nth-last-child(N)` | no | yes |
| `nth-of-type-N:*` | `&:nth-of-type(N)` | no | yes |
| `nth-last-of-type-N:*` | `&:nth-last-of-type(N)` | no | yes |
| `first-of-type:*` | `&:first-of-type` | yes | yes |
| `last-of-type:*` | `&:last-of-type` | yes | yes |
| `only-of-type:*` | `&:only-of-type` | yes | yes |
| `empty:*` | `&:empty` | yes | yes |
| `disabled:*` | `&:disabled` | yes | yes |
| `enabled:*` | `&:enabled` | yes | yes |
| `checked:*` | `&:checked` | yes | yes |
| `indeterminate:*` | `&:indeterminate` | yes | yes |
| `default:*` | `&:default` | yes | yes |
| `required:*` | `&:required` | yes | yes |
| `valid:*` | `&:valid` | yes | yes |
| `invalid:*` | `&:invalid` | yes | yes |
| `in-range:*` | `&:in-range` | yes | yes |
| `out-of-range:*` | `&:out-of-range` | yes | yes |
| `placeholder-shown:*` | `&:placeholder-shown` | yes | yes |
| `autofill:*` | `&:autofill` | yes | yes |
| `read-only:*` | `&:read-only` | yes | yes |
| `open:*` | `&:is([open], :popover-open)` | yes (details only) | yes (details + popover) |
| `inert:*` | `&[inert], &:where([inert] *)` | no | yes |

## Pseudo-element Variants

| Variant | Compiles to | v3 | v4 |
|---------|-------------|----|----|
| `before:*` | `&::before` (requires `content-['']` default in v4 if no content set) | yes | yes |
| `after:*` | `&::after` (same) | yes | yes |
| `placeholder:*` | `&::placeholder` | yes | yes |
| `file:*` | `&::file-selector-button` | yes | yes |
| `marker:*` | `&::marker, & *::marker` | yes | yes |
| `selection:*` | `&::selection, & *::selection` | yes | yes |
| `first-line:*` | `&::first-line` | yes | yes |
| `first-letter:*` | `&::first-letter` | yes | yes |
| `backdrop:*` | `&::backdrop` | yes | yes |
| `details-content:*` | `&::details-content` | no | yes |

## Media-query Variants

### Breakpoints (mobile-first, both versions)

| Variant | Min-width | CSS |
|---------|-----------|-----|
| `sm:*` | 40rem (640px) | `@media (width >= 40rem)` |
| `md:*` | 48rem (768px) | `@media (width >= 48rem)` |
| `lg:*` | 64rem (1024px) | `@media (width >= 64rem)` |
| `xl:*` | 80rem (1280px) | `@media (width >= 80rem)` |
| `2xl:*` | 96rem (1536px) | `@media (width >= 96rem)` |
| `max-sm:*` ... `max-2xl:*` | upper-bound counterparts | `@media (width < 40rem)` etc. |
| `min-[Npx]:*` | arbitrary | `@media (width >= Npx)` |
| `max-[Npx]:*` | arbitrary | `@media (width < Npx)` |

### Preferences

| Variant | CSS | v3 | v4 |
|---------|-----|----|----|
| `dark:*` | `@media (prefers-color-scheme: dark)` (overridable to class/attribute) | yes | yes |
| `light:*` | `@media (prefers-color-scheme: light)` | yes | yes |
| `motion-safe:*` | `@media (prefers-reduced-motion: no-preference)` | yes | yes |
| `motion-reduce:*` | `@media (prefers-reduced-motion: reduce)` | yes | yes |
| `contrast-more:*` | `@media (prefers-contrast: more)` | yes | yes |
| `contrast-less:*` | `@media (prefers-contrast: less)` | yes | yes |
| `forced-colors:*` | `@media (forced-colors: active)` | yes | yes |
| `pointer-fine:*` | `@media (pointer: fine)` | no | yes |
| `pointer-coarse:*` | `@media (pointer: coarse)` | no | yes |
| `portrait:*` | `@media (orientation: portrait)` | yes | yes |
| `landscape:*` | `@media (orientation: landscape)` | yes | yes |
| `print:*` | `@media print` | yes | yes |
| `screen:*` | `@media screen` | yes | yes |
| `ltr:*` | `&:where(:not([dir=rtl] *))` | yes | yes |
| `rtl:*` | `&:where([dir=rtl] *)` | yes | yes |
| `starting:*` | `@starting-style` block | no | yes |

## Feature-query Variants

| Variant | Compiles to | v3 | v4 |
|---------|-------------|----|----|
| `supports-[CSS]:*` | `@supports (CSS)` | yes | yes |
| `not-supports-[CSS]:*` | `@supports not (CSS)` | no | yes |

Examples : `supports-[display:grid]:grid`, `supports-[backdrop-filter]:backdrop-blur-md`, `not-supports-[backdrop-filter]:bg-white/95`.

## Attribute Variants

### ARIA shortcuts

| Variant | Compiles to | v3 | v4 |
|---------|-------------|----|----|
| `aria-checked:*` | `&[aria-checked="true"]` | yes | yes |
| `aria-disabled:*` | `&[aria-disabled="true"]` | yes | yes |
| `aria-expanded:*` | `&[aria-expanded="true"]` | yes | yes |
| `aria-hidden:*` | `&[aria-hidden="true"]` | yes | yes |
| `aria-pressed:*` | `&[aria-pressed="true"]` | yes | yes |
| `aria-readonly:*` | `&[aria-readonly="true"]` | yes | yes |
| `aria-required:*` | `&[aria-required="true"]` | yes | yes |
| `aria-selected:*` | `&[aria-selected="true"]` | yes | yes |
| `aria-busy:*` | `&[aria-busy="true"]` | yes | yes |
| `aria-modal:*` | `&[aria-modal="true"]` | yes | yes |
| `aria-[expr]:*` | `&[expr]` (arbitrary) | yes | yes |

### data-* shortcuts

| Variant | Compiles to | v3 | v4 |
|---------|-------------|----|----|
| `data-active:*` | `&[data-active]` | yes | yes |
| `data-[KEY]:*` | `&[data-KEY]` | yes | yes |
| `data-[KEY=VALUE]:*` | `&[data-KEY="VALUE"]` | yes | yes |
| `data-[KEY^=VALUE]:*` | `&[data-KEY^="VALUE"]` (starts-with) | yes | yes |
| `data-[KEY$=VALUE]:*` | `&[data-KEY$="VALUE"]` (ends-with) | yes | yes |
| `data-[KEY*=VALUE]:*` | `&[data-KEY*="VALUE"]` (contains) | yes | yes |

## Combinator Variants

| Variant | Compiles to | v3 | v4 |
|---------|-------------|----|----|
| `group-{state}:*` | `.group:state &` | yes | yes |
| `peer-{state}:*` | `.peer:state ~ &` | yes | yes |
| `group/NAME` + `group-{state}/NAME:*` | named group | yes | yes |
| `peer/NAME` + `peer-{state}/NAME:*` | named peer | yes | yes |
| `group-has-[selector]:*` | `.group:has(selector) &` | yes (arbitrary) | yes (built-in) |
| `peer-has-[selector]:*` | `.peer:has(selector) ~ &` | yes (arbitrary) | yes (built-in) |
| `group-aria-checked:*` | `.group[aria-checked="true"] &` | yes | yes |
| `group-data-[KEY=VALUE]:*` | `.group[data-KEY="VALUE"] &` | yes | yes |
| `has-[selector]:*` | `&:has(selector)` | no (use arbitrary) | yes |
| `has-checked:*` | `&:has(:checked)` | no | yes |
| `not-{variant}:*` | `&:not(:variant)` or `@media not (...)` | no | yes |
| `in-[selector]:*` | `:where(selector) &` (implicit group) | no | yes |

### `*` and `**` (direct/deep children)

| Variant | Compiles to | v3 | v4 |
|---------|-------------|----|----|
| `*:utility` | `& > * { utility }` (direct children) | yes | yes |
| `**:utility` | `& * { utility }` (all descendants) | no | yes |

## Arbitrary Variants

Any CSS selector wrapped in `[&...]:` becomes a variant. `&` is the styled element, `_` represents a space in the selector.

| Pattern | Compiles to |
|---------|-------------|
| `[&:nth-child(3)]:*` | `&:nth-child(3)` |
| `[&.is-active]:*` | `&.is-active` |
| `[&_p]:*` | `& p` (descendant) |
| `[&>p]:*` | `& > p` (direct child) |
| `[&~p]:*` | `& ~ p` (sibling) |
| `[@media(width>=900px)]:*` | `@media (width >= 900px) { & }` |
| `[@supports(...)]:*` | `@supports (...) { & }` |
| `[@container(...)]:*` | `@container (...) { & }` |

## Stacking Order (BREAKING CHANGE)

| Version | Direction | Example | Compiled selector |
|---------|-----------|---------|-------------------|
| v3.4 | right-to-left | `first:*:pt-0` | `ul > :first-child { padding-top: 0; }` |
| v4.0+ | left-to-right | `*:first:pt-0` | `ul > :first-child { padding-top: 0; }` |

Same compiled selector, OPPOSITE written order. Migrate every stack involving `*:`, `**:`, structural variants, and arbitrary `[&...]` after a long-chained variant.

## Important Modifier (BREAKING CHANGE)

| Version | Syntax | Example |
|---------|--------|---------|
| v3.4 | leading `!` | `!flex !bg-red-500 hover:!bg-blue-500` |
| v4.0+ | trailing `!` | `flex! bg-red-500! hover:bg-blue-500!` |

## Hover Variant Gate (v4 BEHAVIOUR CHANGE)

v4 compiles `hover:` inside `@media (hover: hover)`, so touch devices that fire `:hover` on tap no longer trigger sticky hover styles. Override per project :

```css
@import "tailwindcss";
@custom-variant hover (&:hover);
```

## Custom Variants

### v3.4 (JavaScript plugin)

```js
const plugin = require('tailwindcss/plugin')

module.exports = {
  plugins: [
    plugin(({ addVariant }) => {
      addVariant('third', '&:nth-child(3)')
      addVariant('hocus', ['&:hover', '&:focus'])
      addVariant('supports-grid', '@supports (display: grid)')
    }),
  ],
}
```

### v4.0+ (CSS directive)

```css
@import "tailwindcss";

@custom-variant third (&:nth-child(3));
@custom-variant hocus (&:hover, &:focus);
@custom-variant supports-grid {
  @supports (display: grid) { @slot; }
}
```

`addVariant()` from the v3 plugin API is removed in v4. Convert each call into a `@custom-variant` directive. The `@slot` placeholder is required when wrapping the variant in an at-rule.
