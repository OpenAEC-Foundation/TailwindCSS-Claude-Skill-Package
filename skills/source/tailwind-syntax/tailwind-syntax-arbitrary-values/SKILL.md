---
name: tailwind-syntax-arbitrary-values
description: >
  Use when needing a one-off value that is not in the theme: specific hex colors,
  arbitrary pixel sizes, calc() expressions, custom grid templates, unusual font sizes,
  specific transforms, custom selectors, runtime CSS-variable values. Use also when
  reading existing Tailwind markup and encountering `bg-[#1da1f2]`, `w-[calc(100%-2rem)]`,
  `data-[state=open]:`, `[mask-type:luminance]`, `bg-(--brand)`, `[&_p]:`, or any other
  square-bracket / parens-bracket utility, and needing to know what it compiles to.
  Use also when an arbitrary value is being misinterpreted (background-size vs
  background-position, font-size vs color) and a type hint is required.
  Prevents (a) reaching for arbitrary syntax when a theme token already exists (lazy
  arbitrary values fragment design systems), (b) dynamic class-string concatenation
  like `bg-[${color}]` that never appears as a literal token and silently generates
  no CSS, (c) using v4 parens CSS-var syntax `bg-(--brand)` in a v3 project where it
  is parsed incorrectly, (d) using literal spaces inside `[...]` which terminates the
  class-name token, (e) mistaking a type-hint prefix as the property name itself
  (`text-[color:var(--c)]` is a `color` utility, not a property assignment), and (f)
  reaching for `text-[14.5px]` when `text-base/[1.4]` size-plus-line-height syntax
  fits, or `mt-17` in v4 where dynamic integer spacing avoids arbitrary syntax entirely.
  Covers the complete arbitrary-value system: value-arbitrary `bg-[#hex]`, `w-[calc()]`,
  `text-[Npx]`, `top-[-Npx]`, modifier-arbitrary `[&:nth-child(3)]:`, `[&>*]:`, `[&_p]:`,
  variant-arbitrary `data-[state=open]:`, `aria-[expanded=true]:`, `supports-[display:grid]:`,
  arbitrary-property `[mask-type:luminance]`, `[scroll-snap-stop:always]`, type-hint
  disambiguation `bg-[length:200px_100px]`, `bg-[image:url(...)]`, `text-[color:var(--c)]`,
  `text-[length:var(--c)]`, CSS-variable shorthand `bg-(--brand)` (v4 parens) vs
  `bg-[var(--brand)]` (v3 brackets), space encoding with `_`, comma encoding with `_`
  in v4 (`grid-cols-[max-content_1fr]`), source-detection rules (classes must be
  literal whole tokens), JIT performance model (v3 JIT vs v4 Oxide), and the v4
  dynamic-integer fallback for spacing that obviates many arbitrary use cases.
  Keywords: arbitrary value, arbitrary class, brackets syntax, bracket notation,
  one-off color, bg-[hex], w-[calc], text-[px], custom value, custom selector,
  arbitrary variant, arbitrary modifier, arbitrary property, [mask-type], data-[state],
  aria-[expanded], type hint, length color image, bg-[length:], text-[color:],
  CSS variable Tailwind, bg-(--var), v4 parens syntax, bg-[var()], v3 brackets,
  underscore space, why does my arbitrary class not work, dynamic class not generated,
  JIT compiler, just in time, Oxide engine, performance, literal token, class detection,
  source scanning, mt-17, dynamic spacing.
license: MIT
compatibility: "Designed for Claude Code. Requires Tailwind CSS v3.4 or v4.0+."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# Tailwind CSS Syntax: Arbitrary Values

Arbitrary values are Tailwind's escape hatch: any utility can accept a one-off value via square-bracket syntax (or parens for CSS variables in v4). The syntax is identical in v3 and v4 ; the only difference is the v4 parens shorthand for CSS variables and the v4 Oxide engine that generates the CSS roughly 5x faster. Mastering arbitrary values means knowing WHEN to reach for them (rarely) and WHEN to use a theme token (almost always).

## Quick Reference

### Five arbitrary-value patterns

| Pattern : example : compiles to |
|--|
| Value-arbitrary : `bg-[#1da1f2]` : `background-color: #1da1f2` |
| Modifier-arbitrary : `[&:nth-child(3)]:underline` : custom selector wraps the utility |
| Variant-arbitrary : `data-[state=open]:rotate-180` : `[data-state="open"] { rotate: 180deg }` |
| Arbitrary property : `[mask-type:luminance]` : `mask-type: luminance` (no utility namespace at all) |
| Type-hinted arbitrary : `bg-[length:200px_100px]` : disambiguates ambiguous values |

### Top 3 most-used patterns

```html
<!-- 1. One-off color or size (works v3 + v4) -->
<button class="bg-[#1da1f2] text-[14.5px] w-[calc(100%-2rem)]">Custom values</button>

<!-- 2. CSS variable for runtime theming -->
<!-- v4 parens shorthand -->
<div class="bg-(--brand-bg) text-(--brand-fg)">v4</div>
<!-- v3 brackets equivalent -->
<div class="bg-[var(--brand-bg)] text-[var(--brand-fg)]">v3</div>

<!-- 3. Arbitrary variant for one-off selector -->
<ul class="[&_li]:list-disc [&>li:first-child]:font-bold">
  <li>First (bold)</li>
  <li>Second</li>
</ul>
```

### Type hints when ambiguous

| Hint | When needed | Example |
|------|-------------|---------|
| `length:` | distinguish from color/keyword on `text-*`, `bg-*` with CSS var | `text-[length:var(--my-size)]` |
| `color:` | distinguish from length on `text-*`, `bg-*` with CSS var | `text-[color:var(--my-color)]` |
| `image:` | distinguish from length/color on `bg-*` | `bg-[image:url('/hero.png')]` |
| `family-name:` | distinguish on `font-*` with CSS var | `font-[family-name:var(--my-font)]` |
| `position:` / `size:` | distinguish on `bg-*` | `bg-[position:50%_50%]`, `bg-[size:cover]` |

## Decision Trees

### Should I use arbitrary syntax at all?

```
need a value outside the default theme?
├── value used in 2+ places?
│   └── ALWAYS extend the theme (v4 @theme or v3 tailwind.config.js) ; NEVER duplicate arbitrary values
├── value used exactly once + unlikely to repeat?
│   └── arbitrary value is fine
├── value comes from runtime data (user picks color, server sends size)?
│   └── ALWAYS use inline style `style={{...}}` for fully dynamic values, NEVER arbitrary
├── value is an integer multiple of base spacing (Tailwind v4)?
│   └── ALWAYS use dynamic integer utility `mt-17`, `gap-43` ; NEVER `mt-[68px]` when on v4
└── value is a CSS variable from your theme?
    ├── v4 : ALWAYS use parens shorthand `bg-(--brand)` (shorter)
    └── v3 : ALWAYS use brackets with `var()` `bg-[var(--brand)]`
```

### Picking the right arbitrary-value flavour

```
what am I customising?
├── the value of an existing utility (color, size, calc()) : VALUE-arbitrary
│   └── `bg-[#hex]`, `w-[calc()]`, `text-[Npx]`, `rotate-[7.5deg]`
├── a CSS property Tailwind does not ship a utility for : ARBITRARY PROPERTY
│   └── `[mask-type:luminance]`, `[scroll-snap-stop:always]`, `[contain:layout]`
├── the selector the utility applies to : MODIFIER-arbitrary
│   └── `[&_p]:text-red-500`, `[&>*]:p-4`, `[&.dragging]:cursor-grabbing`
├── a custom attribute or media match : VARIANT-arbitrary
│   └── `data-[state=open]:rotate-180`, `aria-[sort=ascending]:bg-blue-50`, `supports-[display:grid]:grid`
└── a value that is ambiguous (string could parse multiple ways) : TYPE-HINTED arbitrary
    └── `bg-[length:200px_100px]`, `text-[color:var(--c)]`, `text-[length:var(--c)]`
```

### Why is my arbitrary class not generating CSS?

```
class is in the DOM but no CSS rule exists?
├── built via string concatenation (`bg-[${color}]` or `bg-[#${hex}]`)?
│   └── ROOT CAUSE : Tailwind scans source as literal tokens. Concatenation NEVER produces
│       a complete literal class. Fix : static map or inline `style`.
├── value contains an unencoded space (`grid-cols-[1fr 2fr]`)?
│   └── ROOT CAUSE : space terminates the class-name token. Fix : `grid-cols-[1fr_2fr]`.
├── value contains a CSS variable without proper syntax?
│   ├── v3 : MUST be `bg-[var(--c)]` (full var() form inside brackets)
│   └── v4 : either `bg-[var(--c)]` (still works) or `bg-(--c)` (shorthand)
├── using v4 parens syntax `bg-(--c)` in v3 project?
│   └── ROOT CAUSE : v3 does NOT parse parens-arbitrary. Fix : use `bg-[var(--c)]` in v3.
├── ambiguous value (length vs color from CSS var)?
│   └── ROOT CAUSE : Tailwind cannot disambiguate at compile time. Fix : type hint
│       `text-[color:var(--c)]` or `text-(color:--c)` (v4).
└── class only appears in production after a build?
    └── ROOT CAUSE : content scanning missed the source file. Fix : adjust content glob
        (v3) or add `@source "./path"` (v4).
```

## Patterns

### Pattern 1: Value-arbitrary

The single most common form. Square brackets wrap a CSS value the utility would otherwise refuse.

```html
<!-- Colors -->
<div class="bg-[#1da1f2]">Twitter blue</div>
<div class="text-[rgb(20,30,40)]">Custom rgb</div>
<div class="border-[oklch(0.65_0.196_254)]">Custom oklch</div>

<!-- Sizes -->
<div class="w-[calc(100%-2rem)]">Width via calc</div>
<div class="h-[80vh]">Viewport-relative</div>
<div class="text-[14.5px]">Sub-pixel font size</div>
<div class="text-[14.5px]/[1.3]">Size + line-height in one</div>
<div class="top-[-113px]">Negative top (auto-detected as negative)</div>

<!-- Grids -->
<div class="grid grid-cols-[24rem_2.5rem_minmax(0,1fr)]">Custom 3-column layout</div>

<!-- Transforms / filters -->
<div class="rotate-[7.5deg] blur-[2.5px] hue-rotate-[15deg]">Custom rotations</div>
```

### Pattern 2: Modifier-arbitrary (custom selector)

The `[&...]:utility` form gives the utility a custom selector. `&` is the styled element ; `_` represents a space in the selector.

```html
<!-- Direct child selector -->
<ul class="[&>li]:list-disc [&>li]:ml-4">
  <li>One</li>
  <li>Two</li>
</ul>

<!-- Descendant selector (use _ for the space) -->
<article class="[&_p]:text-slate-600 [&_h2]:text-2xl [&_a]:text-blue-600">
  <h2>Heading</h2>
  <p>Body</p>
  <a>Link</a>
</article>

<!-- Sibling and attribute selectors -->
<input class="peer [&~p]:hidden focus:[&~p]:block" />

<!-- Pseudo-class shortcut for things not in the built-in list -->
<li class="[&:nth-child(-n+3)]:font-bold">First 3 items bold</li>
<li class="[&::-webkit-scrollbar]:hidden">Hide webkit scrollbar</li>

<!-- Complex selector combining multiple parts -->
<div class="[&>[data-active]+span]:text-blue-600">
  <button data-active>...</button>
  <span>Sibling of active gets blue</span>
</div>
```

### Pattern 3: Variant-arbitrary (custom attribute or query)

Built-in variants like `aria-checked:` and `data-active:` are shortcuts. Arbitrary variants accept any value or any selector.

```html
<!-- Custom data attribute value -->
<div data-state="open" class="data-[state=open]:rotate-180 data-[state=closed]:rotate-0">
  Chevron
</div>

<!-- Custom aria attribute value -->
<th aria-sort="ascending" class="aria-[sort=ascending]:bg-blue-50">
  Name
</th>

<!-- Browser feature detection -->
<div class="grid-cols-3 grid supports-[display:grid]:gap-4 not-supports-[backdrop-filter]:bg-white">
  Grid + fallback
</div>

<!-- Custom at-rule -->
<div class="[@media(prefers-reduced-data:reduce)]:bg-none">
  Saves bandwidth
</div>
<div class="[@container(width>=600px)]:grid">
  Container-query escape hatch
</div>
```

### Pattern 4: Arbitrary property (no utility namespace)

When Tailwind ships no utility for a CSS property at all, use `[property:value]`.

```html
<!-- CSS properties without Tailwind utilities -->
<div class="[mask-type:luminance] hover:[mask-type:alpha]">SVG mask switching</div>
<section class="[scroll-snap-stop:always] [scroll-snap-align:center]">Snap target</section>
<div class="[contain:layout] [content-visibility:auto]">Optimisation hints</div>
<div class="[view-transition-name:hero]">View Transitions API</div>
<input class="[appearance:textfield] [-moz-appearance:textfield]">Strip number arrows</input>
```

These compile to a direct CSS declaration with no class-name prefix : the `[mask-type:luminance]` utility produces `mask-type: luminance;`.

### Pattern 5: Type hints (disambiguation)

```html
<!-- Ambiguous : 200px_100px could be size or position -->
<div class="bg-[length:200px_100px]">Background-size</div>
<div class="bg-[position:200px_100px]">Background-position</div>

<!-- Ambiguous : a CSS var could be color or length -->
<div class="text-[color:var(--accent)]">Color utility</div>
<div class="text-[length:var(--size)]">Font-size utility</div>

<!-- v4 parens shorthand with type hint -->
<div class="text-(color:--accent)">v4 color</div>
<div class="text-(length:--size)">v4 size</div>

<!-- Always-needed type hints -->
<div class="bg-[image:url('/hero.png')]">Force image interpretation</div>
<div class="font-[family-name:var(--my-font)]">Force family-name interpretation</div>
```

Without the hint, Tailwind would either guess wrong or refuse to compile.

### Pattern 6: CSS variables (v3 vs v4)

```html
<!-- v3 : ALWAYS brackets with full var() form -->
<div class="bg-[var(--brand-bg)] text-[var(--brand-fg)]">v3</div>

<!-- v4 : EITHER brackets with var() (still works) OR parens shorthand -->
<div class="bg-[var(--brand-bg)] text-[var(--brand-fg)]">v4 explicit</div>
<div class="bg-(--brand-bg) text-(--brand-fg)">v4 shorthand (preferred)</div>

<!-- v4 parens + type hint for ambiguous CSS vars -->
<div class="text-(color:--accent)">v4 color from CSS var</div>
<div class="text-(length:--size)">v4 size from CSS var</div>
```

The v4 parens shorthand IS NOT available in v3. Mixing in a v3 project breaks silently.

## Performance: JIT and Class Detection

Both v3 and v4 use Just-In-Time compilation : Tailwind scans source files for class names and generates CSS only for the classes that appear. ALWAYS write class names as **complete literal tokens** in source. NEVER concatenate, interpolate, or build them dynamically at template-render time.

```jsx
// BROKEN : never generates CSS
<div className={`bg-[${color}]`} />

// FIXED : the literal class string appears in source
const STYLE_MAP = { primary: 'bg-[#1da1f2]', secondary: 'bg-[#888]' }
<div className={STYLE_MAP[variant]} />

// FIXED for truly dynamic values : use inline style
<div style={{ backgroundColor: color }} />
```

| Version | Engine | Full build | Incremental |
|---------|--------|-----------|-------------|
| v3.4 JIT | JavaScript-based JIT | baseline | baseline |
| v4.0+ Oxide | Rust + Lightning CSS | ~5x faster | ~100x faster |

The performance gain comes from Oxide ; the source-detection RULES are identical.

## Reference Links

- `references/methods.md` : complete arbitrary-value syntax reference (every flavour, every type hint, v3 vs v4 differences).
- `references/examples.md` : worked examples for value/modifier/variant/property/type-hinted/CSS-variable forms across both versions.
- `references/anti-patterns.md` : the dynamic-class-string trap, missing type hints, v4-parens-in-v3 mistakes, spaces inside brackets, reaching for arbitrary when a theme token or v4 dynamic integer fits.

## Verified Sources

- https://tailwindcss.com/docs/styling-with-utility-classes (arbitrary values, type hints, v4 parens shorthand)
- https://tailwindcss.com/docs/adding-custom-styles (arbitrary values + JIT class-detection rules)
- https://tailwindcss.com/blog/just-in-time-the-next-generation-of-tailwind-css (v3 JIT origin)
- https://tailwindcss.com/blog/tailwindcss-v4 (v4 Oxide engine + parens shorthand)
- https://tailwindcss.com/docs/detecting-classes-in-source-files (literal-token scan rule)

Last verified : 2026-05-19 (Tailwind CSS v4.x current).
