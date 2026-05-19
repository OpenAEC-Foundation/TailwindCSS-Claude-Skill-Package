# Tailwind CSS Variants: Anti-Patterns

Failure modes mined from the v3 to v4 upgrade guide and the Tailwind issue tracker.

## AP-1: Reading v4 Variant Stack Like a v3 Stack

```html
<!-- AUTHOR INTENDED : pad the first list item's top to 0 -->
<!-- v3 muscle memory : -->
<ul class="py-4 first:*:pt-0">
  <li>One</li>
  <li>Two</li>
</ul>
<!-- WORKS in v3. SILENTLY BREAKS in v4 because the stacking order flipped. -->
```

### Why this fails

v3 read stacked variants right-to-left, so `first:*:pt-0` was : direct children (`*`), then first child (`first`), then apply `pt-0`. v4 reads left-to-right : `first:*:pt-0` now means : if the styled element matches `:first-child`, look at its direct children, then apply `pt-0` to those. The compiled selector is `:first-child > *`, which targets the children of the first item, not the first item itself.

### Fix : flip stacks with structural-variant stacking

```html
<!-- v4 -->
<ul class="py-4 *:first:pt-0 *:last:pb-0">
  <li>One</li>
  <li>Two</li>
</ul>
```

Migration table for every common pattern :

| v3 | v4 |
|----|----|
| `first:*:pt-0` | `*:first:pt-0` |
| `last:*:pb-0` | `*:last:pb-0` |
| `odd:*:bg-white` | `*:odd:bg-white` |
| `hover:[&_p]:underline` | `[&_p]:hover:underline` |
| `[&_h2]:font-bold first:[&_h2]:text-blue-500` | `first:[&_h2]:font-bold` (same in v4) |

ALWAYS run `npx @tailwindcss/upgrade` first ; the tool handles the documented common cases. Manually inspect any handwritten stacks involving `*:`, `**:`, or arbitrary `[&...]`.

Source : https://tailwindcss.com/docs/upgrade-guide section 14.

## AP-2: Using Leading Bang `!flex` in v4

```html
<!-- BROKEN in v4 -->
<div class="!flex !bg-red-500">Important rules</div>
<!-- v4 compiler does not recognise leading-bang as the important modifier. -->
<!-- The classes are emitted as ordinary utilities ; no `!important` is applied. -->
```

### Why this fails

v4 moved the important modifier from leading to trailing position to align with the proposed CSS spec. The leading-bang form is no longer parsed as `!important`.

### Fix : trailing bang

```html
<!-- v4 -->
<div class="flex! bg-red-500! hover:bg-red-600!">Important rules</div>
```

The upgrade tool handles most cases but cannot rewrite bang-inside-arbitrary-variant patterns ; review those by hand.

Source : https://tailwindcss.com/docs/upgrade-guide (Important modifier section).

## AP-3: Hover Variant Silently Stops Working on Touch Devices

```html
<!-- v3 : hover styles applied unconditionally -->
<button class="bg-blue-600 hover:bg-blue-800">Tap or hover</button>
<!-- After migrating to v4, the same markup applies hover styles ONLY on devices
     that support real hover. Touch devices that fire :hover on tap no longer
     trigger the darker state. -->
```

### Why this fails

v4 wraps every `hover:*` utility in `@media (hover: hover)`. The rationale is to prevent touch-tap from leaving sticky hover styles on mobile, but apps that intentionally USED the v3 behaviour for tap-feedback regress.

### Fix : restore v3 behaviour with `@custom-variant`

```css
/* In main CSS */
@import "tailwindcss";

@custom-variant hover (&:hover);
```

After this directive, `hover:` no longer gates on the media query. ALWAYS apply at the project level if you depended on the v3 behaviour.

Source : https://tailwindcss.com/docs/upgrade-guide (Hover styles section).

## AP-4: Ambiguous Unnamed Groups

```html
<!-- BROKEN : two unnamed `.group` ancestors ; group-hover binds to nearest -->
<article class="group">
  <button class="group">
    <span class="group-hover:underline">Underlines on which hover?</span>
  </button>
</article>
<!-- The span sees TWO `.group` ancestors. group-hover binds to the NEAREST one
     (the button), so hovering the article alone does NOT underline. Surprising
     half the time. -->
```

### Why this fails

Tailwind's `group-hover:` selector compiles to `.group:hover &`. CSS resolves `:hover` against the nearest matching ancestor, so any nested `.group` shadows the outer one.

### Fix : ALWAYS use named groups when nesting

```html
<article class="group/card hover:shadow-lg">
  <button class="group/edit">
    <span class="group-hover/card:text-blue-600">Hovering the card</span>
    <span class="group-hover/edit:underline">Hovering the button</span>
  </button>
</article>
```

The same rule applies to peers (`peer/email`, `peer-invalid/email:`).

Source : https://tailwindcss.com/docs/hover-focus-and-other-states (Differentiating nested groups section).

## AP-5: Using v4-only Variants in a v3 Project

```html
<!-- v3 project -->
<button class="opacity-50 not-hover:opacity-100">v3</button>
<label class="has-checked:bg-indigo-50">v3</label>
<div class="in-[role=tablist]:font-bold">v3</div>
<!-- All three emit unknown-utility errors in v3 and silently produce no CSS. -->
```

### Why this fails

`not-*`, `has-*`, `in-*`, `**:`, `nth-N` shortcuts (without arbitrary syntax), `nth-of-type-N`, `nth-last-N`, `details-content:`, `pointer-fine:`, `pointer-coarse:`, `inert:`, `not-supports-*:`, and `starting:` are ALL v4 additions. They do not exist in v3 and produce neither errors at build time nor effective CSS at runtime.

### Fix : either upgrade to v4 or use the v3 arbitrary-variant equivalents

| v4 shortcut | v3 arbitrary equivalent |
|-------------|---------------------------|
| `not-hover:opacity-100` | `[&:not(:hover)]:opacity-100` |
| `has-checked:bg-indigo-50` | `[&:has(:checked)]:bg-indigo-50` |
| `in-[role=tablist]:font-bold` | `[[role=tablist]_&]:font-bold` |
| `**:data-avatar:size-12` | `[&_[data-avatar]]:size-12` (descendants) |
| `nth-3:underline` | `[&:nth-child(3)]:underline` |
| `not-supports-[display:grid]:flex` | not available natively ; use `@supports not (...)` in custom CSS |

Source : https://tailwindcss.com/docs/upgrade-guide (Variants section).

## AP-6: Registering Custom Variants With `addVariant()` in v4

```js
// v4 tailwind.config.js (loaded via @config "./tailwind.config.js")
const plugin = require('tailwindcss/plugin')

module.exports = {
  plugins: [
    plugin(({ addVariant }) => {
      addVariant('third', '&:nth-child(3)') // SILENTLY DOES NOTHING in v4
    }),
  ],
}
```

### Why this fails

v4 removed the `addVariant()` plugin API. JS plugins loaded via `@config` still execute, but `addVariant` calls are no-ops. The generated CSS contains no `.third\:underline` rule and the markup `<li class="third:underline">` produces no styling.

### Fix : convert each call to `@custom-variant`

```css
@import "tailwindcss";

@custom-variant third (&:nth-child(3));
@custom-variant hocus (&:hover, &:focus);
@custom-variant supports-grid {
  @supports (display: grid) { @slot; }
}
```

Source : https://tailwindcss.com/docs/upgrade-guide and https://tailwindcss.com/docs/functions-and-directives.

## AP-7: Forgetting `content-['']` on `before:` / `after:` in v4

```html
<!-- BROKEN in v4 : the pseudo-element renders but has no box -->
<div class="relative before:absolute before:inset-0 before:bg-black/50">
  Overlay
</div>
```

### Why this fails

CSS requires `content: ''` to render generated pseudo-elements. v3 applied a default `content: ''` to `before:*` / `after:*` automatically. v4 removed the implicit default, so any pseudo-element without an explicit `content-*` utility is invisible.

### Fix : add `before:content-[''] ` (or supply content explicitly)

```html
<!-- v4 -->
<div class="relative before:content-[''] before:absolute before:inset-0 before:bg-black/50">
  Overlay
</div>
```

Source : https://tailwindcss.com/docs/upgrade-guide (Empty content default section).

## AP-8: Stacking `dark:` Inside `group-` Without Considering Order

```html
<!-- BROKEN intent : dark mode AND group-hover -->
<a class="group">
  <span class="group-hover:dark:text-white">Title</span>
</a>
<!-- This works in v3 (right-to-left) AND v4 (left-to-right) because both
     `dark:` and `group-hover:` are media/attribute conditions that compose
     commutatively. BUT readers expect dark-mode-then-hover ordering. -->
```

### Why this is a smell

While the compiled selector for `group-hover:dark:` and `dark:group-hover:` is equivalent (both wrap `.group:hover &` inside `@media (prefers-color-scheme: dark)`), inconsistent ordering across a codebase becomes ungreppable. Pick ONE convention.

### Fix : adopt a project-wide stacking convention

shadcn/ui convention : `dark:` LAST (closest to the utility) so search-and-replace queries like `dark:bg-` find every dark-mode background.

```html
<a class="group">
  <span class="group-hover:dark:text-white">Title</span>
</a>
```

Document the convention in the project's STYLEGUIDE.md or equivalent.

## AP-9: Arbitrary Variant Selector Spaces

```html
<!-- BROKEN : space inside [&...] does not resolve -->
<div class="[& p]:text-red-500">Descendant red</div>
<!-- Tailwind class-name parser stops at the first space, treating " p]" as a
     separate class. -->
```

### Why this fails

A literal space in the class name terminates the token. Tailwind uses `_` as the in-arbitrary space replacement.

### Fix : use `_` for spaces in arbitrary variant selectors

```html
<div class="[&_p]:text-red-500">Descendant red</div>
<!-- Compiles to : & p { color: red; } -->
```

For multi-word arbitrary values that genuinely need a space (e.g. `grid-cols-[1fr 2fr]`), v4 changed the convention : use `_` instead.

```html
<!-- v3 -->
<div class="grid-cols-[max-content,1fr,max-content]">v3 commas</div>

<!-- v4 -->
<div class="grid-cols-[max-content_1fr_max-content]">v4 underscores</div>
```

Source : https://tailwindcss.com/docs/upgrade-guide (Arbitrary values section).

## AP-10: Expecting `dark:` to Toggle Without a Strategy

```html
<!-- BROKEN : dev expects dark:bg-slate-900 to fire when a JS toggle adds .dark to <html> -->
<body class="bg-white dark:bg-slate-900">
```

### Why this fails

In both v3 and v4, the default `dark:` strategy is `prefers-color-scheme: dark` (OS-level). Adding `class="dark"` to `<html>` does NOTHING unless the dark strategy is explicitly switched.

### Fix : declare the strategy

v3 (`tailwind.config.js`) :

```js
module.exports = { darkMode: 'class' }
// or, since v3.4.1 :
module.exports = { darkMode: ['class', '[data-theme="dark"]'] }
```

v4 (main CSS) :

```css
@import "tailwindcss";
@custom-variant dark (&:where(.dark, .dark *));
```

After the strategy is set, the JS toggle works :

```js
document.documentElement.classList.toggle('dark', userPrefersDark)
```

Source : https://tailwindcss.com/docs/dark-mode and https://v3.tailwindcss.com/docs/dark-mode.
