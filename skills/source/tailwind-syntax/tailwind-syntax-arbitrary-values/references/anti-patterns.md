# Tailwind CSS Arbitrary Values: Anti-Patterns

Real failure modes mined from the Tailwind issue tracker and the official docs.

## AP-1: Dynamic Class String Interpolation

```jsx
// BROKEN
function Badge({ color }) {
  return <span className={`bg-[${color}] text-white`}>{children}</span>
}
function Card({ hex }) {
  return <div className={`bg-[#${hex}]`}>{children}</div>
}
```

### Why this fails

Tailwind's source scanner reads files as **literal text tokens**. `bg-[${color}]` is never a complete token in source : the value comes from a runtime variable. The compiler emits no CSS rule, the DOM gets the class, and the element renders with no background.

### Fix : static class map OR inline style

```jsx
// FIX A : static map (preferred when colors enumerable)
const BG_CLASSES = {
  red: "bg-[#ef4444]",
  blue: "bg-[#3b82f6]",
  green: "bg-[#22c55e]",
}
<span className={`${BG_CLASSES[color]} text-white`}>{children}</span>

// FIX B : inline style (preferred when truly dynamic from user data)
<div style={{ backgroundColor: `#${hex}` }}>...</div>

// FIX C : v4 safelist via @source inline()
// @source inline("bg-[#{ef4444,3b82f6,22c55e}]");
```

ALWAYS prefer FIX A when the value set is known at design time. Use FIX B only when the value is genuinely user-supplied (color picker, CMS payload).

Source : https://tailwindcss.com/docs/detecting-classes-in-source-files and https://github.com/tailwindlabs/tailwindcss/issues/18136.

## AP-2: Literal Space Inside Brackets

```html
<!-- BROKEN -->
<div class="grid-cols-[1fr 2fr 1fr]">Three columns</div>
<div class="[transition: transform 0.3s ease]">Smooth</div>
```

### Why this fails

A literal space in the HTML `class` attribute terminates the class-name token. Tailwind sees `grid-cols-[1fr` as one class and `2fr` and `1fr]` as two more (none of which is valid).

### Fix : encode spaces with `_`

```html
<div class="grid-cols-[1fr_2fr_1fr]">Three columns</div>
<div class="[transition:transform_0.3s_ease]">Smooth</div>
```

Tailwind decodes `_` back to a space when generating the CSS. The single exception : underscores inside `url(...)` are NOT decoded, because they are valid in URLs.

```html
<!-- url() with literal underscore in the filename -->
<div class="bg-[url('/images/file_name.png')]">Underscore preserved</div>
```

Source : https://tailwindcss.com/docs/styling-with-utility-classes (Arbitrary values, Encoding spaces section).

## AP-3: Using v4 Parens Shorthand in a v3 Project

```html
<!-- BROKEN in v3 -->
<div class="bg-(--brand) text-(--accent)">v3 project trying v4 syntax</div>
```

### Why this fails

The parens-arbitrary form is v4-only. v3 does NOT parse `bg-(--brand)` as a CSS-variable reference ; it produces no CSS rule, and the element gets the default theme background.

### Fix : use brackets with explicit var() in v3

```html
<!-- v3 -->
<div class="bg-[var(--brand)] text-[var(--accent)]">v3</div>

<!-- v4 (either form, parens preferred) -->
<div class="bg-(--brand) text-(--accent)">v4 shorthand</div>
<div class="bg-[var(--brand)] text-[var(--accent)]">v4 explicit (still works)</div>
```

Migrate AFTER fully on v4 ; mixing during transition produces silent breakage.

Source : https://tailwindcss.com/docs/upgrade-guide (Arbitrary variables section).

## AP-4: Missing Type Hint for Ambiguous CSS Variable

```html
<!-- BROKEN : Tailwind cannot tell length from color -->
<div class="text-[var(--my-token)]">What is --my-token?</div>
```

### Why this fails

`text-*` accepts both `font-size` (length) and `color`. When the value is an inline literal, Tailwind disambiguates by parsing : `text-[14px]` is a length, `text-[#fff]` is a color. A CSS variable contains no parse signal : Tailwind cannot know which CSS property to set. Without a type hint, it defaults to one interpretation (usually color) and produces wrong CSS.

### Fix : ALWAYS add a type hint for CSS-var-driven `text-*`, `bg-*`, `font-*`

```html
<!-- v3 brackets -->
<div class="text-[color:var(--accent)]">Color</div>
<div class="text-[length:var(--display-size)]">Size</div>
<div class="font-[family-name:var(--brand-font)]">Family</div>
<div class="bg-[image:var(--hero-image)]">Background image</div>

<!-- v4 parens -->
<div class="text-(color:--accent)">Color</div>
<div class="text-(length:--display-size)">Size</div>
```

The hint precedes the value, separated by a colon. Available hints : `length`, `color`, `image`, `family-name`, `position`, `size`, `number`, `percentage`, `angle`, `url`.

Source : https://tailwindcss.com/docs/styling-with-utility-classes (Type hints section).

## AP-5: Reaching for Arbitrary When a Theme Token Already Exists

```html
<!-- BROKEN : design-system erosion -->
<div class="p-[16px] m-[8px] gap-[24px]">Triple-arbitrary</div>
```

### Why this is a smell

`p-[16px]` is identical in compiled output to `p-4` (4 * 0.25rem = 1rem = 16px). Using arbitrary values for theme-equivalent sizes :

- bypasses the design system (no longer constrained)
- defeats search-and-replace ("show me every padding-4 usage" can no longer find this)
- enlarges the CSS bundle (every unique arbitrary value emits its own rule)
- breaks design-token swapping (changing `--spacing` to `0.3rem` no longer updates these)

### Fix : ALWAYS prefer theme tokens

```html
<!-- Same compiled output, cleaner -->
<div class="p-4 m-2 gap-6">Theme tokens</div>
```

If the value is genuinely outside the scale, EXTEND the theme :

```css
@theme {
  --spacing-13: 3.25rem;   /* if used in 3+ places */
}
```

Then `p-13` works everywhere.

Source : https://tailwindcss.com/docs/theme (extending the theme).

## AP-6: Reaching for Arbitrary for Integer Spacing in v4

```html
<!-- v4 anti-pattern -->
<div class="mt-[68px] w-[116px] gap-[172px]">v4 integer arbitrary</div>
```

### Why this is suboptimal in v4

v4 generates spacing utilities for ANY positive integer automatically (`mt-17`, `w-29`, `gap-43`). They desugar to `calc(var(--spacing) * N)`. The arbitrary form bypasses this and produces a HARDCODED `68px` instead of `calc(var(--spacing) * 17)`, breaking design-token swapping.

### Fix : v4 dynamic integer

```html
<div class="mt-17 w-29 gap-43">v4 dynamic spacing</div>
```

Now `--spacing: 0.25rem` produces 68px, but a swap to `--spacing: 0.3rem` produces 81.6px automatically. Design-token-driven theming works.

This rule is v4-only. In v3 the arbitrary form is necessary for non-scale integers.

Source : https://tailwindcss.com/docs/padding (v4 dynamic spacing section).

## AP-7: Confusing Arbitrary Property With Type-Hinted Value

```html
<!-- AUTHOR INTENT : set background-color via CSS variable -->
<!-- BROKEN -->
<div class="[background-color:var(--brand)]">Background</div>

<!-- This is an arbitrary PROPERTY, which works (compiles to background-color: var(--brand)).
     But it bypasses the bg-* utility entirely : no opacity-modifier support,
     no tailwind-merge conflict-resolution, no theme-token integration. -->
```

### Why this is a smell

Arbitrary properties `[prop:value]` are an escape hatch for CSS that Tailwind does not ship a utility for. Using them where a utility EXISTS is a design-system erosion : you lose every Tailwind-aware tooling layer (twMerge, theme tokens, opacity modifier).

### Fix : use the typed utility form

```html
<!-- v3 / v4 -->
<div class="bg-[var(--brand)]">v3 syntax</div>
<div class="bg-(--brand)">v4 parens</div>

<!-- Now opacity modifier works -->
<div class="bg-(--brand)/50">Half-opacity brand</div>
```

ALWAYS prefer typed utility + type-hinted arbitrary over raw arbitrary property when a utility exists.

Source : https://tailwindcss.com/docs/styling-with-utility-classes (Arbitrary properties section).

## AP-8: Forgetting `image:` Hint for `bg-[url(...)]`

```html
<!-- WORKS in v3, AMBIGUOUS in v4 -->
<div class="bg-[url('/hero.png')]">Hero</div>
```

### Why this is fragile

v3 always interpreted `url(...)` inside `bg-[...]` as a background image. v4 is stricter : if any other CSS variable or value could match, the parser refuses. For url-only values it still works, but combining with other arbitrary values exposes the ambiguity.

### Fix : ALWAYS use the explicit `image:` hint for clarity

```html
<div class="bg-[image:url('/hero.png')] bg-cover bg-center">Hero</div>
```

The compiled output is identical, but the markup is self-documenting and survives parser updates.

Source : https://tailwindcss.com/docs/styling-with-utility-classes.

## AP-9: Treating the `&` Token as a Generic Selector Placeholder

```html
<!-- BROKEN : misuse of & -->
<div class="[&]:bg-red-500">Apply red bg</div>
<!-- This is valid syntax but pointless : [&]: compiles to `& {}`, same as the plain
     utility `bg-red-500`. The arbitrary modifier added nothing. -->

<!-- WORSE : trying to use & for a child selector -->
<div class="[&]:>li]:p-4">Style children</div>
<!-- Syntax error. & represents the styled element, not a chain start. -->
```

### Why this fails

The `&` token in `[&...]:utility` represents the styled element's own selector. Children, descendants, siblings need `& > child`, `& descendant`, `& ~ sibling` written out. Without `&` the bracket is just a CSS value, not a selector.

### Fix : write the full relative selector

```html
<!-- Children -->
<ul class="[&>li]:p-4">Direct children</ul>

<!-- Descendants (use _ for space) -->
<article class="[&_p]:text-slate-600">All paragraph descendants</article>

<!-- Sibling -->
<input class="[&~p]:hidden focus:[&~p]:block" />

<!-- Self with state -->
<button class="[&:disabled]:opacity-50">Self when disabled</button>
```

Source : https://tailwindcss.com/docs/hover-focus-and-other-states (Arbitrary variants section).

## AP-10: Putting Arbitrary Values Inside Template Literals That Survive Static Analysis

```jsx
// BROKEN : looks like a literal, but template-substitution prevents detection
const size = 14.5
<div className={`text-[${size}px]`} />
```

### Why this fails

Even though the substitution result `text-[14.5px]` is a valid Tailwind class, the LITERAL form in source is `text-[${size}px]`. Tailwind scans source for literal tokens before runtime evaluation ; the interpolation marker `${...}` breaks the literal match.

### Fix : either inline the literal or use inline style

```jsx
// FIX A : literal class string
<div className="text-[14.5px]" />

// FIX B : if the value is dynamic, inline style (NOT a class)
<div style={{ fontSize: `${size}px` }} />

// FIX C : enumerated map of literal classes
const SIZE_CLASSES = {
  small: "text-[12.5px]",
  medium: "text-[14.5px]",
  large: "text-[16.5px]",
}
<div className={SIZE_CLASSES[sizeKey]} />
```

ALWAYS use inline `style` for fully dynamic values. NEVER hope Tailwind detects through interpolation.

Source : https://tailwindcss.com/docs/detecting-classes-in-source-files.
