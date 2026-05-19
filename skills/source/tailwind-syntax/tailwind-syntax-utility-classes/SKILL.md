---
name: tailwind-syntax-utility-classes
description: >
  Use when writing Tailwind markup and you need to recall or verify the
  vocabulary of base utility families (spacing, color, typography, layout,
  flexbox, grid, sizing, border, background, effects). Also use when
  diagnosing why a class name does not produce expected styling, or when
  porting from a different framework and looking up the Tailwind
  equivalent.
  Prevents incorrect class names that the content scanner silently
  ignores, the v4 shadow-scale shift confusion where `shadow-sm` is
  smaller than v3's `shadow-sm` because `shadow-xs` was inserted, and
  the "I thought sm: meant small screens only" responsive trap (see
  [tailwind-syntax-responsive] for the full responsive treatment).
  Covers : the canonical class vocabulary for the ten base utility
  families with v3 and v4 differences flagged inline; the value scales
  (spacing 0..96, type xs..9xl, shadow 2xs..2xl, color 50..950, etc.);
  v4 dynamic spacing utilities that accept any integer; v4 opacity
  modifier syntax (`bg-red-500/50`); the official 22-family color
  palette in v4 oklch.
  Keywords: utility classes, padding, margin, p-4, px-2, m-auto, gap-4,
  bg-red-500, text-white, text-lg, font-bold, flex, grid, grid-cols-3,
  flex-1, justify-center, items-center, w-full, h-screen, rounded-md,
  shadow-md, shadow-xs, opacity-50, what class do I need,
  Tailwind cheat sheet, how do I center, how do I make 3 columns,
  blank screen no styling, class not generated, shadow looks different
  after upgrade, shadow scale changed, why is shadow smaller now,
  default color palette, how many shades does Tailwind have,
  bg opacity, color/50, slate gray zinc.
license: MIT
compatibility: "Designed for Claude Code. Requires Tailwind CSS v3.4 or v4.0+."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# Tailwind CSS : Base Utility Classes

The canonical vocabulary. Every utility documented here is a literal token Tailwind generates from theme tokens. ALWAYS write classes as complete literals; see [tailwind-core-architecture] for why partial token construction breaks the content scanner.

## Quick Reference

### The ten base utility families

| Family | Examples | Coverage in this skill |
|--------|----------|------------------------|
| Spacing | `p-4`, `px-2`, `m-auto`, `mt-8`, `space-y-4`, `gap-6` | section 1 |
| Color | `bg-red-500`, `text-white`, `border-slate-200`, `ring-blue-500/50` | section 2 |
| Typography | `text-lg`, `font-bold`, `tracking-tight`, `leading-relaxed` | section 3 |
| Layout (display + position) | `flex`, `grid`, `block`, `hidden`, `absolute`, `top-0`, `z-10` | section 4 |
| Flexbox | `flex-row`, `justify-center`, `items-center`, `flex-1`, `shrink-0` | section 5 |
| Grid | `grid-cols-3`, `col-span-2`, `grid-rows-2`, `grid-flow-col`, `auto-cols-fr` | section 6 |
| Sizing | `w-full`, `h-screen`, `min-w-0`, `max-h-96`, `size-10` | section 7 |
| Border | `border`, `border-2`, `rounded-md`, `divide-y` | section 8 |
| Background | `bg-white`, `bg-[url('/img.png')]`, `bg-cover`, `bg-center` | section 9 |
| Effects | `shadow-md`, `opacity-75`, `mix-blend-multiply`, `backdrop-blur-sm` | section 10 |

### Three invariants

1. ALWAYS write class names as complete literal tokens. NEVER assemble them dynamically.
2. ALWAYS read responsive prefixes (`sm:`, `md:`, ...) as "and up". See [tailwind-syntax-responsive].
3. ALWAYS check the v4 callouts in this skill before assuming a v3 utility name still applies.

### v4 changes affecting base utilities (callouts)

- **Shadow scale shifted (BREAKING)** : v4 inserted `shadow-2xs` and `shadow-xs`. v3 `shadow-sm` is approximately v4 `shadow-md`. See section 10.
- **Same shift applies** to `blur-*`, `rounded-*`, `drop-shadow-*`, `backdrop-blur-*` (`-xs` inserted below `-sm`).
- **Opacity utilities removed** : v4 removed `bg-opacity-50`, `text-opacity-*`, `border-opacity-*`, `divide-opacity-*`, `ring-opacity-*`, `placeholder-opacity-*`. ALWAYS use the slash syntax `bg-red-500/50` instead.
- **Gradient family renamed** : v3 `bg-gradient-to-r` becomes v4 `bg-linear-to-r`. See [tailwind-syntax-gradients].
- **Flex shorthand changes** : v3 `flex-shrink-*`, `flex-grow-*` become v4 `shrink-*`, `grow-*`.
- **Text overflow** : v3 `overflow-ellipsis` becomes v4 `text-ellipsis`.
- **Important modifier position** : v3 `!font-bold` becomes v4 `font-bold!`.
- **Dynamic spacing** : v4 generates `p-{any-integer}`, `m-{any-integer}`, `w-{any-integer}`, `grid-cols-{any-integer}` automatically; v3 was scale-limited.
- **Default color palette** : v4 ships in oklch wide-gamut with 22 family names: 9 neutrals (slate, gray, zinc, neutral, stone, taupe, mauve, mist, olive) and 13 colors (red, orange, amber, yellow, lime, green, emerald, teal, cyan, sky, blue, indigo, violet, purple, fuchsia, pink, rose) plus `black` and `white`.

## 1. Spacing

### Padding

| Class | Property | Notes |
|-------|----------|-------|
| `p-<n>` | `padding: calc(var(--spacing) * <n>)` | All sides |
| `px-<n>` / `py-<n>` | inline / block padding | Horizontal / vertical |
| `pt-<n>` / `pr-<n>` / `pb-<n>` / `pl-<n>` | single-side | Top / right / bottom / left |
| `ps-<n>` / `pe-<n>` | logical (start / end) | RTL-aware |
| `p-[<value>]` | arbitrary | `p-[37px]`, `p-[1.5em]` |
| `p-(--var)` | CSS variable | v4 only |

### Margin

| Class | Property | Notes |
|-------|----------|-------|
| `m-<n>` | `margin` | All sides |
| `mx-<n>` / `my-<n>` | horizontal / vertical | |
| `mt-<n>` / `mr-<n>` / `mb-<n>` / `ml-<n>` | single-side | |
| `ms-<n>` / `me-<n>` | logical | RTL-aware |
| `-m-<n>`, `-mt-<n>`, etc. | negative | ALWAYS prefix with `-` |
| `m-auto` | `margin: auto` | Centering trick with `mx-auto` |

### Spacing scale (`<n>` values)

`0`, `0.5`, `1`, `1.5`, `2`, `2.5`, `3`, `3.5`, `4`, `5`, `6`, `7`, `8`, `9`, `10`, `11`, `12`, `14`, `16`, `20`, `24`, `28`, `32`, `36`, `40`, `44`, `48`, `52`, `56`, `60`, `64`, `72`, `80`, `96`.

In v4 ANY integer works because spacing is `calc(var(--spacing) * <n>)` with `--spacing: 0.25rem` by default. `p-17`, `m-23`, `w-29` ALL emit valid CSS in v4.

### Gap and space-between

| Class | Property |
|-------|----------|
| `gap-<n>` | `gap` (flex + grid both) |
| `gap-x-<n>` / `gap-y-<n>` | column-gap / row-gap |
| `space-x-<n>` / `space-y-<n>` | gap between siblings (legacy; prefer `gap-*` with `flex` or `grid`) |

ALWAYS prefer `flex flex-col gap-<n>` over `space-y-<n>` per the v4 upgrade-guide recommendation; the `space-*` family uses fragile sibling selectors and changed behaviour in v4.

## 2. Color

### Color palette (v4 oklch)

22 family names :
- **Neutrals** (9) : `slate`, `gray`, `zinc`, `neutral`, `stone`, `taupe`, `mauve`, `mist`, `olive`
- **Hues** (13) : `red`, `orange`, `amber`, `yellow`, `lime`, `green`, `emerald`, `teal`, `cyan`, `sky`, `blue`, `indigo`, `violet`, `purple`, `fuchsia`, `pink`, `rose`
- Plus `black`, `white`, `transparent`, `current` (currentColor), `inherit`

Each named family has **11 shades** : `50`, `100`, `200`, `300`, `400`, `500`, `600`, `700`, `800`, `900`, `950`.

### Color utilities by property

| Class | Property |
|-------|----------|
| `text-<color>-<shade>` | `color` |
| `bg-<color>-<shade>` | `background-color` |
| `border-<color>-<shade>` | `border-color` |
| `border-{t,r,b,l}-<color>-<shade>` | per-side border colour |
| `outline-<color>-<shade>` | `outline-color` |
| `ring-<color>-<shade>` | `box-shadow` ring colour |
| `divide-<color>-<shade>` | sibling-border colour |
| `accent-<color>-<shade>` | `accent-color` (form controls) |
| `caret-<color>-<shade>` | `caret-color` (text input) |
| `decoration-<color>-<shade>` | `text-decoration-color` |
| `fill-<color>-<shade>` | SVG `fill` |
| `stroke-<color>-<shade>` | SVG `stroke` |
| `placeholder-<color>-<shade>` | `::placeholder { color }` |
| `from-<color>-<shade>` / `via-` / `to-` | gradient stops |

### Opacity modifier (v4 + v3.3+)

ALWAYS use the slash syntax :

```html
<div class="bg-red-500/50">50% opacity background</div>
<div class="text-white/[0.85]">85% with arbitrary precision</div>
```

In v4 the `bg-opacity-*`, `text-opacity-*`, etc. utility families are **REMOVED**. NEVER write `bg-red-500 bg-opacity-50` in v4 code.

### Arbitrary and variable colors

```html
<div class="bg-[#1da1f2]">arbitrary hex</div>
<div class="bg-(--brand-color)">v4 CSS variable</div>
<div class="bg-[var(--brand-color)]">v3 CSS variable</div>
```

## 3. Typography

| Class | Property |
|-------|----------|
| `text-<size>` | `font-size` + paired `line-height` |
| `text-<size>/<leading>` | `font-size` + custom `line-height` (e.g., `text-sm/6`) |
| `font-<family>` | `font-family` (defaults : `sans`, `serif`, `mono`) |
| `font-<weight>` | `font-weight` |
| `leading-<value>` | `line-height` |
| `tracking-<value>` | `letter-spacing` |
| `text-<align>` | `text-align` : `left`, `center`, `right`, `justify`, `start`, `end` |
| `uppercase`, `lowercase`, `capitalize`, `normal-case` | `text-transform` |
| `italic`, `not-italic` | `font-style` |
| `underline`, `overline`, `line-through`, `no-underline` | `text-decoration-line` |
| `decoration-<style>` | `text-decoration-style` : `solid`, `dotted`, `dashed`, `wavy`, `double` |
| `decoration-<thickness>` | `text-decoration-thickness` : `auto`, `from-font`, `0`, `1`, `2`, `4`, `8` |
| `whitespace-<value>` | `white-space` : `normal`, `nowrap`, `pre`, `pre-line`, `pre-wrap`, `break-spaces` |
| `text-ellipsis`, `text-clip` | `text-overflow` (v4; v3 used `overflow-ellipsis`) |
| `truncate` | three-in-one : `overflow: hidden`, `text-overflow: ellipsis`, `white-space: nowrap` |

### Font sizes (v4 defaults)

| Class | Font size | Line height |
|-------|-----------|-------------|
| `text-xs` | 0.75rem (12px) | calc(1 / 0.75) |
| `text-sm` | 0.875rem (14px) | calc(1.25 / 0.875) |
| `text-base` | 1rem (16px) | calc(1.5 / 1) |
| `text-lg` | 1.125rem (18px) | calc(1.75 / 1.125) |
| `text-xl` | 1.25rem (20px) | calc(1.75 / 1.25) |
| `text-2xl` | 1.5rem (24px) | calc(2 / 1.5) |
| `text-3xl` | 1.875rem (30px) | calc(2.25 / 1.875) |
| `text-4xl` | 2.25rem (36px) | calc(2.5 / 2.25) |
| `text-5xl` | 3rem (48px) | 1 |
| `text-6xl` | 3.75rem (60px) | 1 |
| `text-7xl` | 4.5rem (72px) | 1 |
| `text-8xl` | 6rem (96px) | 1 |
| `text-9xl` | 8rem (128px) | 1 |

### Font weights

`font-thin` (100), `font-extralight` (200), `font-light` (300), `font-normal` (400), `font-medium` (500), `font-semibold` (600), `font-bold` (700), `font-extrabold` (800), `font-black` (900).

### Letter spacing

`tracking-tighter` (-0.05em), `tracking-tight` (-0.025em), `tracking-normal` (0), `tracking-wide` (0.025em), `tracking-wider` (0.05em), `tracking-widest` (0.1em).

### Line height (standalone)

`leading-none` (1), `leading-tight` (1.25), `leading-snug` (1.375), `leading-normal` (1.5), `leading-relaxed` (1.625), `leading-loose` (2). Arbitrary : `leading-[1.2]`.

## 4. Layout (display + position + z-index)

| Class | Property |
|-------|----------|
| `block`, `inline-block`, `inline`, `flex`, `inline-flex`, `grid`, `inline-grid`, `table`, `inline-table`, `table-row`, `table-cell`, `flow-root`, `contents`, `hidden`, `list-item` | `display` |
| `static`, `fixed`, `absolute`, `relative`, `sticky` | `position` |
| `top-<n>` / `right-<n>` / `bottom-<n>` / `left-<n>` | per-side offset (uses spacing scale) |
| `inset-<n>` | shorthand for all four sides |
| `inset-x-<n>` / `inset-y-<n>` | horizontal / vertical |
| `z-<n>` | `z-index` : `0`, `10`, `20`, `30`, `40`, `50`, `auto` (v4 dynamic : any integer) |
| `overflow-<value>` | `auto`, `hidden`, `clip`, `visible`, `scroll` |
| `overflow-x-<value>` / `overflow-y-<value>` | per-axis |
| `visible`, `invisible`, `collapse` | `visibility` |

## 5. Flexbox

| Class | Property |
|-------|----------|
| `flex-row`, `flex-row-reverse`, `flex-col`, `flex-col-reverse` | `flex-direction` |
| `flex-wrap`, `flex-wrap-reverse`, `flex-nowrap` | `flex-wrap` |
| `flex-1`, `flex-auto`, `flex-initial`, `flex-none` | `flex` shorthand |
| `flex-<n>`, `flex-<fraction>`, `flex-[<value>]`, `flex-(--var)` | arbitrary flex values |
| `grow`, `grow-0`, `grow-<n>` (v4) | `flex-grow` (v3 used `flex-grow-*`) |
| `shrink`, `shrink-0`, `shrink-<n>` (v4) | `flex-shrink` (v3 used `flex-shrink-*`) |
| `basis-<n>`, `basis-<fraction>`, `basis-full`, `basis-auto` | `flex-basis` |
| `order-<n>`, `order-first`, `order-last`, `order-none` | `order` |
| `justify-<value>` | `justify-content` : `start`, `end`, `center`, `between`, `around`, `evenly`, `stretch`, `normal` |
| `items-<value>` | `align-items` : `start`, `end`, `center`, `baseline`, `stretch`, `last-baseline` |
| `content-<value>` | `align-content` : `start`, `end`, `center`, `between`, `around`, `evenly`, `stretch`, `normal`, `baseline` |
| `self-<value>` | `align-self` : `auto`, `start`, `end`, `center`, `stretch`, `baseline` |

## 6. Grid

| Class | Property |
|-------|----------|
| `grid-cols-<n>` | `grid-template-columns: repeat(<n>, minmax(0, 1fr))` (v4 any integer) |
| `grid-cols-none`, `grid-cols-subgrid`, `grid-cols-[<value>]` | non-numeric |
| `grid-rows-<n>`, `grid-rows-none`, `grid-rows-subgrid` | rows version |
| `col-span-<n>`, `col-span-full`, `col-start-<n>`, `col-end-<n>` | column placement |
| `row-span-<n>`, `row-span-full`, `row-start-<n>`, `row-end-<n>` | row placement |
| `grid-flow-row`, `grid-flow-col`, `grid-flow-row-dense`, `grid-flow-col-dense`, `grid-flow-dense` | `grid-auto-flow` |
| `auto-cols-auto`, `auto-cols-min`, `auto-cols-max`, `auto-cols-fr` | `grid-auto-columns` |
| `auto-rows-auto`, `auto-rows-min`, `auto-rows-max`, `auto-rows-fr` | `grid-auto-rows` |
| `place-content-<value>`, `place-items-<value>`, `place-self-<value>` | combined `*-content`, `*-items`, `*-self` |

## 7. Sizing

| Class | Property |
|-------|----------|
| `w-<n>` | `width` |
| `h-<n>` | `height` |
| `size-<n>` | shorthand : `width` + `height` (v3.4+ and v4) |
| `min-w-<n>`, `max-w-<n>`, `min-h-<n>`, `max-h-<n>` | min/max width/height |
| `w-full`, `w-screen`, `w-svw`, `w-lvw`, `w-dvw`, `w-min`, `w-max`, `w-fit`, `w-auto` | named widths |
| `h-full`, `h-screen`, `h-svh`, `h-lvh`, `h-dvh`, `h-min`, `h-max`, `h-fit`, `h-auto` | named heights |
| `w-<fraction>` (e.g., `w-1/2`, `w-2/3`, `w-5/12`) | fractional widths |
| `max-w-xs`, `max-w-sm`, ..., `max-w-7xl`, `max-w-prose`, `max-w-screen-md` | container scale |
| `aspect-square`, `aspect-video`, `aspect-[16/9]`, `aspect-auto` | `aspect-ratio` |

## 8. Border

| Class | Property |
|-------|----------|
| `border`, `border-0`, `border-2`, `border-4`, `border-8`, `border-<n>` | `border-width` |
| `border-x-<n>`, `border-y-<n>`, `border-t-<n>`, `border-r-<n>`, `border-b-<n>`, `border-l-<n>` | per-axis / side |
| `border-<color>-<shade>` | `border-color` (see section 2) |
| `border-{style}` | `border-style` : `solid`, `dashed`, `dotted`, `double`, `hidden`, `none` |
| `rounded`, `rounded-none`, `rounded-xs`, `rounded-sm`, `rounded-md`, `rounded-lg`, `rounded-xl`, `rounded-2xl`, `rounded-3xl`, `rounded-full` | `border-radius` |
| `rounded-t-*`, `rounded-r-*`, `rounded-b-*`, `rounded-l-*` | per-side radius |
| `rounded-tl-*`, `rounded-tr-*`, `rounded-br-*`, `rounded-bl-*` | per-corner |
| `divide-x-<n>`, `divide-y-<n>`, `divide-<color>` | sibling-border between flex/grid children |
| `outline`, `outline-<n>`, `outline-<style>`, `outline-<color>`, `outline-offset-<n>` | outline family |
| `ring-<n>`, `ring-<color>-<shade>`, `ring-offset-<n>`, `ring-inset` | box-shadow-based ring |

### v3 vs v4 border-color and ring-width

- v3 default border color : `gray-200`. v4 default : `currentColor`. ALWAYS add an explicit `border-<color>-<shade>` on bordered elements when migrating, OR add a shim in `@layer base`.
- v3 default `ring` width : 3px. v4 default `ring` width : 1px. ALWAYS use `ring-3` to preserve v3 behaviour, OR a shim.

See [tailwind-errors-v4-migration].

## 9. Background

| Class | Property |
|-------|----------|
| `bg-<color>-<shade>` | `background-color` (see section 2) |
| `bg-[url('/img/hero.png')]`, `bg-(image:--my-img)` | `background-image` arbitrary |
| `bg-none`, `bg-linear-to-r`, `bg-conic-*`, `bg-radial-*` | gradient utilities (see [tailwind-syntax-gradients]) |
| `bg-cover`, `bg-contain`, `bg-auto` | `background-size` |
| `bg-[length:200px_100px]` | arbitrary size with type hint |
| `bg-center`, `bg-top`, `bg-right`, etc. | `background-position` |
| `bg-repeat`, `bg-no-repeat`, `bg-repeat-x`, `bg-repeat-y`, `bg-repeat-space`, `bg-repeat-round` | `background-repeat` |
| `bg-fixed`, `bg-local`, `bg-scroll` | `background-attachment` |
| `bg-clip-border`, `bg-clip-padding`, `bg-clip-content`, `bg-clip-text` | `background-clip` |
| `bg-origin-border`, `bg-origin-padding`, `bg-origin-content` | `background-origin` |

## 10. Effects

### Shadow (the v4 scale-shift trap)

| v4 class | v3 equivalent | Notes |
|----------|---------------|-------|
| `shadow-2xs` | n/a | v4-only; smallest |
| `shadow-xs` | n/a (closest : v3 `shadow-sm`) | INSERTED |
| `shadow-sm` | v3 `shadow` (no suffix) | shifted up |
| `shadow-md` | v3 `shadow-md` (same name, slightly different value) | |
| `shadow-lg` | v3 `shadow-lg` | |
| `shadow-xl` | v3 `shadow-xl` | |
| `shadow-2xl` | v3 `shadow-2xl` | |
| `shadow-none` | `shadow-none` | unchanged |
| `shadow-[<value>]` | arbitrary | |
| `shadow-<color>-<shade>` | shadow color | colored shadows |

ALWAYS read v4 `shadow-sm` as roughly equivalent to v3's `shadow` (default). NEVER assume v4 `shadow-sm` matches v3 `shadow-sm` visually : v3's `shadow-sm` is closer to v4's `shadow-xs`.

The same shift applies to `blur-*`, `rounded-*`, `drop-shadow-*`, `backdrop-blur-*` : v4 inserted `-xs` below `-sm`.

### Inset shadow (v4-only)

`inset-shadow-2xs`, `inset-shadow-xs`, `inset-shadow-sm`, `inset-shadow-none`, `inset-shadow-[<value>]`, `inset-shadow-<color>-<shade>`. See [tailwind-syntax-modern-utilities].

### Other effects

| Class | Property |
|-------|----------|
| `opacity-<n>` | `opacity` : 0, 5, 10, 15, ..., 100; v4 dynamic any integer |
| `mix-blend-<mode>` | `mix-blend-mode` |
| `bg-blend-<mode>` | `background-blend-mode` |
| `blur-<size>`, `blur-[<value>]`, `blur-none` | `filter: blur(...)` |
| `brightness-<n>`, `contrast-<n>`, `grayscale`, `hue-rotate-<n>`, `invert`, `saturate-<n>`, `sepia` | filter family |
| `backdrop-blur-<size>` | `backdrop-filter: blur(...)` |
| `backdrop-brightness-<n>`, etc. | backdrop-filter family |
| `drop-shadow-<size>`, `drop-shadow-[<value>]`, `drop-shadow-<color>-<shade>` | `filter: drop-shadow(...)` |

## Anti-patterns (summary; full list in references/anti-patterns.md)

NEVER do these :

1. NEVER use removed v4 utilities : `bg-opacity-50`, `text-opacity-*`, `border-opacity-*`. ALWAYS use slash syntax `bg-red-500/50`.
2. NEVER assume `shadow-sm` is identical between v3 and v4. v4 inserted `shadow-xs`, shifting the entire scale.
3. NEVER write `flex-shrink-0` or `flex-grow-1` in v4 code. ALWAYS use `shrink-0` and `grow`.
4. NEVER write `bg-gradient-to-r` in v4 code. ALWAYS use `bg-linear-to-r`.
5. NEVER write `overflow-ellipsis` in v4. ALWAYS use `text-ellipsis`.
6. NEVER assume `border` alone gives a visible border in v4 : the default border color is `currentColor`, not `gray-200`. ALWAYS add an explicit `border-<color>-<shade>`.

## Reference Links

- [references/methods.md](references/methods.md) : complete property mappings for all ten families, value-scale tables, v3/v4 deltas
- [references/examples.md](references/examples.md) : real layout patterns (centered card, sidebar layout, stat grid, hero) using only base utilities
- [references/anti-patterns.md](references/anti-patterns.md) : full anti-pattern catalogue with v4-migration traps

## Cross-references

- [tailwind-core-architecture](../../tailwind-core/tailwind-core-architecture/SKILL.md) : why complete-literal tokens matter
- [tailwind-core-design-system](../../tailwind-core/tailwind-core-design-system/SKILL.md) : the theme tokens that drive these utilities
- [tailwind-core-v3-vs-v4](../../tailwind-core/tailwind-core-v3-vs-v4/SKILL.md) : full v3-to-v4 breaking-change catalogue
- [tailwind-syntax-variants](../tailwind-syntax-variants/SKILL.md) : hover/focus/dark/aria/data prefixes
- [tailwind-syntax-responsive](../tailwind-syntax-responsive/SKILL.md) : `sm:`, `md:`, container queries
- [tailwind-syntax-arbitrary-values](../tailwind-syntax-arbitrary-values/SKILL.md) : `p-[17px]`, `bg-[url(...)]`
- [tailwind-syntax-gradients](../tailwind-syntax-gradients/SKILL.md) : v4 `bg-linear-*`, `bg-conic-*`, `bg-radial-*`
- [tailwind-syntax-modern-utilities](../tailwind-syntax-modern-utilities/SKILL.md) : `inset-shadow-*`, `inset-ring-*`, `@starting-style`
- [tailwind-errors-v4-migration](../../tailwind-errors/tailwind-errors-v4-migration/SKILL.md) : shadow shift, border colour, ring width

## Sources

- https://tailwindcss.com/docs/padding (spacing utilities + dynamic scale)
- https://tailwindcss.com/docs/colors (22-family oklch palette + opacity modifier)
- https://tailwindcss.com/docs/font-size (text-* scale with line-height pairings)
- https://tailwindcss.com/docs/flex (flexbox family)
- https://tailwindcss.com/docs/grid-template-columns (grid family + dynamic grid-cols-{any})
- https://tailwindcss.com/docs/box-shadow (v4 shadow scale with inserted xs/2xs)
- https://v3.tailwindcss.com/docs (v3 baseline for comparison)
- https://tailwindcss.com/docs/upgrade-guide (rename and removal list)

Verified 2026-05-19.
