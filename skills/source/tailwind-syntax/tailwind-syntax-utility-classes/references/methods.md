# Methods : Tailwind Utility Classes

Complete property mappings, value scales, and v3/v4 deltas for the ten base utility families. Use as a reference catalogue when generating class names programmatically or auditing markup.

## 1. Spacing scale (full default values)

| Token | Value | CSS calc (v4) |
|-------|-------|---------------|
| `0` | 0 | 0 |
| `0.5` | 0.125rem (2px) | `calc(var(--spacing) * 0.5)` |
| `1` | 0.25rem (4px) | `calc(var(--spacing) * 1)` |
| `1.5` | 0.375rem (6px) | `calc(var(--spacing) * 1.5)` |
| `2` | 0.5rem (8px) | `calc(var(--spacing) * 2)` |
| `2.5` | 0.625rem (10px) | |
| `3` | 0.75rem (12px) | |
| `3.5` | 0.875rem (14px) | |
| `4` | 1rem (16px) | |
| `5` | 1.25rem (20px) | |
| `6` | 1.5rem (24px) | |
| `7` | 1.75rem (28px) | |
| `8` | 2rem (32px) | |
| `9` | 2.25rem (36px) | |
| `10` | 2.5rem (40px) | |
| `11` | 2.75rem (44px) | |
| `12` | 3rem (48px) | |
| `14` | 3.5rem (56px) | |
| `16` | 4rem (64px) | |
| `20` | 5rem (80px) | |
| `24` | 6rem (96px) | |
| `28` | 7rem (112px) | |
| `32` | 8rem (128px) | |
| `36` | 9rem (144px) | |
| `40` | 10rem (160px) | |
| `44` | 11rem (176px) | |
| `48` | 12rem (192px) | |
| `52` | 13rem (208px) | |
| `56` | 14rem (224px) | |
| `60` | 15rem (240px) | |
| `64` | 16rem (256px) | |
| `72` | 18rem (288px) | |
| `80` | 20rem (320px) | |
| `96` | 24rem (384px) | |

In v4 ANY integer ≥ 0 works, so `p-17`, `p-29`, `p-100` all emit valid CSS. v3 was limited to the discrete scale above (or arbitrary brackets).

Special spacing tokens : `px` (1px), `auto`, `full`, `screen` (only for sizing, not padding).

## 2. Color palette (v4 oklch)

### 22 family names (verified at tailwindcss.com/docs/colors)

Neutrals (9) : `slate`, `gray`, `zinc`, `neutral`, `stone`, `taupe`, `mauve`, `mist`, `olive`.

Hues (13) : `red`, `orange`, `amber`, `yellow`, `lime`, `green`, `emerald`, `teal`, `cyan`, `sky`, `blue`, `indigo`, `violet`, `purple`, `fuchsia`, `pink`, `rose`.

Specials : `black`, `white`, `transparent`, `current`, `inherit`.

### Shade scale (11 steps per family)

`50` (lightest), `100`, `200`, `300`, `400`, `500`, `600`, `700`, `800`, `900`, `950` (darkest).

### Properties driven by `--color-*` namespace (v4)

- `text-<color>-<shade>` : `color`
- `bg-<color>-<shade>` : `background-color`
- `border-<color>-<shade>` : `border-color`
- `border-{t,r,b,l,x,y,s,e}-<color>-<shade>` : per-side / logical
- `outline-<color>-<shade>` : `outline-color`
- `ring-<color>-<shade>` : ring `box-shadow` color
- `ring-offset-<color>-<shade>` : ring-offset `box-shadow` color
- `divide-<color>-<shade>` : sibling-border color
- `accent-<color>-<shade>` : `accent-color` (native form controls)
- `caret-<color>-<shade>` : `caret-color` (text inputs)
- `decoration-<color>-<shade>` : `text-decoration-color`
- `fill-<color>-<shade>` : SVG `fill`
- `stroke-<color>-<shade>` : SVG `stroke`
- `placeholder-<color>-<shade>` : `::placeholder` `color`
- `shadow-<color>-<shade>` : `box-shadow` color
- `from-<color>-<shade>` / `via-<color>-<shade>` / `to-<color>-<shade>` : gradient stops

### Opacity modifier (v4 + v3.3+)

ALWAYS use slash syntax: `<utility>-<color>-<shade>/<percentage>`.

Examples : `bg-red-500/50`, `text-white/[0.85]`, `border-slate-200/30`, `ring-blue-500/40`.

The integer scale : 0, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50, 55, 60, 65, 70, 75, 80, 85, 90, 95, 100. Plus arbitrary `/[<value>]`.

### v4 REMOVED utility families

- `bg-opacity-*`
- `text-opacity-*`
- `border-opacity-*`
- `divide-opacity-*`
- `ring-opacity-*`
- `placeholder-opacity-*`

ALWAYS migrate to slash syntax.

## 3. Typography full reference

### Font size

See SKILL.md section 3 for the full table. Special syntax `text-<size>/<leading>` pairs font-size with a specific line-height : `text-sm/6` → font-size 14px + line-height 1.5rem.

### Font weight

| Class | `font-weight` |
|-------|----------------|
| `font-thin` | 100 |
| `font-extralight` | 200 |
| `font-light` | 300 |
| `font-normal` | 400 |
| `font-medium` | 500 |
| `font-semibold` | 600 |
| `font-bold` | 700 |
| `font-extrabold` | 800 |
| `font-black` | 900 |

### Tracking (letter-spacing)

| Class | Value |
|-------|-------|
| `tracking-tighter` | -0.05em |
| `tracking-tight` | -0.025em |
| `tracking-normal` | 0 |
| `tracking-wide` | 0.025em |
| `tracking-wider` | 0.05em |
| `tracking-widest` | 0.1em |

### Leading (line-height standalone)

| Class | Value |
|-------|-------|
| `leading-none` | 1 |
| `leading-tight` | 1.25 |
| `leading-snug` | 1.375 |
| `leading-normal` | 1.5 |
| `leading-relaxed` | 1.625 |
| `leading-loose` | 2 |
| `leading-<n>` | uses spacing scale (e.g., `leading-6` = 1.5rem) |
| `leading-[<value>]` | arbitrary |

### Family

`font-sans`, `font-serif`, `font-mono`, plus custom families registered in `@theme { --font-display: ... }`.

### Alignment

`text-left`, `text-center`, `text-right`, `text-justify`, `text-start`, `text-end`.

### Decoration

`underline`, `overline`, `line-through`, `no-underline` (line). Style : `decoration-solid`, `decoration-double`, `decoration-dotted`, `decoration-dashed`, `decoration-wavy`. Thickness : `decoration-auto`, `decoration-from-font`, `decoration-0`, `decoration-1`, `decoration-2`, `decoration-4`, `decoration-8`. Underline offset : `underline-offset-auto`, `underline-offset-<n>`.

### Whitespace

`whitespace-normal`, `whitespace-nowrap`, `whitespace-pre`, `whitespace-pre-line`, `whitespace-pre-wrap`, `whitespace-break-spaces`.

### Word break

`break-normal`, `break-words`, `break-all`, `break-keep`.

### Text overflow (v4 rename)

- v4 : `text-ellipsis`, `text-clip`
- v3 : `overflow-ellipsis`, `overflow-clip`

Both versions support `truncate` (the three-property combo).

## 4. Layout

### Display

`block`, `inline-block`, `inline`, `flex`, `inline-flex`, `grid`, `inline-grid`, `table`, `inline-table`, `table-caption`, `table-cell`, `table-column`, `table-column-group`, `table-footer-group`, `table-header-group`, `table-row`, `table-row-group`, `flow-root`, `contents`, `list-item`, `hidden`.

### Position

`static`, `fixed`, `absolute`, `relative`, `sticky`. Offsets use the spacing scale : `top-0` ... `top-96`, `top-auto`, `top-full`, `top-1/2`, `top-[<value>]`, `-top-<n>` (negative).

### Inset shorthand

`inset-0`, `inset-x-0`, `inset-y-0`, `inset-auto`, `inset-<n>`, `inset-[<value>]`. v4 also accepts logical inset : `start-<n>`, `end-<n>`.

### Z-index

`z-0`, `z-10`, `z-20`, `z-30`, `z-40`, `z-50`, `z-auto`. v4 dynamic : any integer (`z-15`, `z-100`, negative `-z-10`).

### Overflow

`overflow-auto`, `overflow-hidden`, `overflow-clip`, `overflow-visible`, `overflow-scroll`. Per-axis : `overflow-x-*`, `overflow-y-*`.

### Visibility

`visible`, `invisible`, `collapse`.

## 5. Flexbox full reference

### Direction

`flex-row`, `flex-row-reverse`, `flex-col`, `flex-col-reverse`.

### Wrap

`flex-wrap`, `flex-wrap-reverse`, `flex-nowrap`.

### Flex shorthand

`flex-1` (`1 1 0%`), `flex-auto` (`1 1 auto`), `flex-initial` (`0 1 auto`), `flex-none` (`none`).

Arbitrary : `flex-<n>`, `flex-<fraction>` (e.g., `flex-1/2`), `flex-[<value>]`, `flex-(--var)`.

### Grow

v4 : `grow` (`flex-grow: 1`), `grow-0`, `grow-<n>` (any integer in v4), `grow-[<value>]`.

v3 : `flex-grow`, `flex-grow-0`.

### Shrink

v4 : `shrink` (`flex-shrink: 1`), `shrink-0`, `shrink-<n>` (any integer in v4), `shrink-[<value>]`.

v3 : `flex-shrink`, `flex-shrink-0`.

### Basis

`basis-<n>`, `basis-<fraction>`, `basis-full`, `basis-auto`, `basis-[<value>]`, `basis-(--var)`.

### Order

`order-<n>` (1..12 default scale, v4 any integer), `order-first` (-9999), `order-last` (9999), `order-none` (0), `-order-<n>`.

### Justify content / items / self

`justify-start`, `justify-end`, `justify-center`, `justify-between`, `justify-around`, `justify-evenly`, `justify-stretch`, `justify-normal`.

`justify-items-start`, `justify-items-end`, `justify-items-center`, `justify-items-stretch`, `justify-items-normal` (grid only).

`justify-self-auto`, `justify-self-start`, `justify-self-end`, `justify-self-center`, `justify-self-stretch` (grid only).

### Align content / items / self

`content-start`, `content-end`, `content-center`, `content-between`, `content-around`, `content-evenly`, `content-stretch`, `content-baseline`, `content-normal`.

`items-start`, `items-end`, `items-center`, `items-baseline`, `items-stretch`, `items-last-baseline`.

`self-auto`, `self-start`, `self-end`, `self-center`, `self-stretch`, `self-baseline`, `self-last-baseline`.

### Place (combined)

`place-content-*` shorthand for `align-content + justify-content`.
`place-items-*` shorthand for `align-items + justify-items`.
`place-self-*` shorthand for `align-self + justify-self`.

## 6. Grid full reference

### Template columns / rows

`grid-cols-<n>` (1..12 default; v4 any integer), `grid-cols-none`, `grid-cols-subgrid`, `grid-cols-[<value>]`, `grid-cols-(--var)`.

`grid-rows-<n>`, `grid-rows-none`, `grid-rows-subgrid`, `grid-rows-[<value>]`.

### Span / start / end

`col-span-<n>`, `col-span-full`, `col-span-[<value>]`.
`col-start-<n>`, `col-start-auto`, `col-end-<n>`, `col-end-auto`.

Same family for rows : `row-span-<n>`, `row-start-<n>`, `row-end-<n>`.

### Auto flow

`grid-flow-row`, `grid-flow-col`, `grid-flow-dense`, `grid-flow-row-dense`, `grid-flow-col-dense`.

### Auto columns / rows

`auto-cols-auto`, `auto-cols-min`, `auto-cols-max`, `auto-cols-fr`, `auto-cols-[<value>]`.

Same for rows : `auto-rows-*`.

### Gap

`gap-<n>`, `gap-x-<n>`, `gap-y-<n>`, `gap-[<value>]`.

## 7. Sizing

### Width

`w-<n>` (spacing scale), `w-<fraction>` (e.g., `w-1/2`, `w-1/3`, `w-2/3`, `w-1/4`, `w-3/4`, `w-1/5`, ..., `w-11/12`), `w-full`, `w-screen`, `w-svw`, `w-lvw`, `w-dvw`, `w-min`, `w-max`, `w-fit`, `w-auto`, `w-[<value>]`.

### Height

`h-<n>`, `h-<fraction>`, `h-full`, `h-screen`, `h-svh`, `h-lvh`, `h-dvh`, `h-min`, `h-max`, `h-fit`, `h-auto`, `h-[<value>]`.

### Size (v3.4+ shorthand)

`size-<n>` : sets both `width` and `height` to the same value. `size-10` = `w-10 h-10`.

### Min / Max

`min-w-<n>`, `max-w-<n>`, `min-h-<n>`, `max-h-<n>`. Plus named max-w containers : `max-w-xs` (20rem), `max-w-sm` (24rem), `max-w-md` (28rem), `max-w-lg` (32rem), `max-w-xl` (36rem), `max-w-2xl` (42rem), `max-w-3xl` (48rem), `max-w-4xl` (56rem), `max-w-5xl` (64rem), `max-w-6xl` (72rem), `max-w-7xl` (80rem), `max-w-prose` (65ch), `max-w-screen-sm` (40rem), `max-w-screen-md` (48rem), `max-w-screen-lg` (64rem), `max-w-screen-xl` (80rem), `max-w-screen-2xl` (96rem), `max-w-none` (none), `max-w-full` (100%), `max-w-min`, `max-w-max`, `max-w-fit`.

### Aspect ratio

`aspect-auto`, `aspect-square` (1/1), `aspect-video` (16/9), `aspect-[<ratio>]` (e.g., `aspect-[4/3]`, `aspect-[1.618/1]`).

## 8. Border

### Width

`border` (1px), `border-0`, `border-2`, `border-4`, `border-8`, `border-<n>` (v4 any integer), `border-[<value>]`.

Per-side : `border-t-<n>`, `border-r-<n>`, `border-b-<n>`, `border-l-<n>`. Per-axis : `border-x-<n>`, `border-y-<n>`. Logical : `border-s-<n>`, `border-e-<n>`.

### Style

`border-solid`, `border-dashed`, `border-dotted`, `border-double`, `border-hidden`, `border-none`.

### Radius

`rounded`, `rounded-none`, `rounded-xs`, `rounded-sm`, `rounded-md`, `rounded-lg`, `rounded-xl`, `rounded-2xl`, `rounded-3xl`, `rounded-4xl` (v4), `rounded-full`, `rounded-[<value>]`.

Per-side : `rounded-t-*`, `rounded-r-*`, `rounded-b-*`, `rounded-l-*`. Per-corner : `rounded-tl-*`, `rounded-tr-*`, `rounded-br-*`, `rounded-bl-*`. Logical : `rounded-ss-*`, `rounded-se-*`, `rounded-ee-*`, `rounded-es-*`.

NOTE : v4 inserted `rounded-xs` below `rounded-sm`; sizes shifted. Same trap as the shadow scale.

### Divide

`divide-x-<n>`, `divide-y-<n>`, `divide-x-reverse`, `divide-y-reverse`, `divide-<color>-<shade>`, `divide-solid`, `divide-dashed`, etc.

### Outline

`outline`, `outline-0`, `outline-1`, `outline-2`, `outline-4`, `outline-8`, `outline-<n>`, `outline-none`, `outline-hidden` (v4 rename of `outline-none` for the legacy "visually hidden but accessible" pattern), `outline-dashed`, `outline-dotted`, `outline-double`. Offset : `outline-offset-<n>`.

### Ring

`ring`, `ring-<n>` (v4 default 1px, v3 default 3px). `ring-inset`. `ring-offset-<n>`. `ring-<color>-<shade>`.

## 9. Background

### Color

All `bg-<color>-<shade>` with optional `/<opacity>` (see section 2).

### Image

`bg-[url('/img/hero.png')]`, `bg-(image:--my-img)`, `bg-none`.

### Gradient (v4 names)

`bg-linear-to-r`, `bg-linear-to-tr`, `bg-linear-<deg>` (e.g., `bg-linear-45`), `bg-conic-*`, `bg-radial-*`. See [tailwind-syntax-gradients] for full coverage.

v3 used `bg-gradient-to-r` and related; all renamed in v4.

### Size

`bg-auto`, `bg-cover`, `bg-contain`, `bg-[length:200px_100px]`.

### Position

`bg-top`, `bg-top-left`, `bg-top-right`, `bg-center`, `bg-left`, `bg-right`, `bg-bottom`, `bg-bottom-left`, `bg-bottom-right`, `bg-[position:right_top]`.

### Repeat

`bg-repeat`, `bg-no-repeat`, `bg-repeat-x`, `bg-repeat-y`, `bg-repeat-round`, `bg-repeat-space`.

### Attachment

`bg-fixed`, `bg-local`, `bg-scroll`.

### Clip and origin

`bg-clip-border`, `bg-clip-padding`, `bg-clip-content`, `bg-clip-text`.

`bg-origin-border`, `bg-origin-padding`, `bg-origin-content`.

## 10. Effects full reference

### Box shadow scale (v4)

| Class | CSS value (v4 defaults) |
|-------|------------------------|
| `shadow-2xs` | `0 1px rgb(0 0 0 / 0.05)` |
| `shadow-xs` | `0 1px 2px 0 rgb(0 0 0 / 0.05)` |
| `shadow-sm` | `0 1px 3px 0 rgb(0 0 0 / 0.1), 0 1px 2px -1px rgb(0 0 0 / 0.1)` |
| `shadow-md` | `0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1)` |
| `shadow-lg` | `0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1)` |
| `shadow-xl` | `0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1)` |
| `shadow-2xl` | `0 25px 50px -12px rgb(0 0 0 / 0.25)` |
| `shadow-none` | `0 0 #0000` |

### Inset shadow (v4-only)

`inset-shadow-2xs`, `inset-shadow-xs`, `inset-shadow-sm`, `inset-shadow-none`, `inset-shadow-[<value>]`, `inset-shadow-<color>-<shade>`.

### Drop shadow

`drop-shadow-2xs`, `drop-shadow-xs`, `drop-shadow-sm`, `drop-shadow-md`, `drop-shadow-lg`, `drop-shadow-xl`, `drop-shadow-2xl`, `drop-shadow-none`, `drop-shadow-<color>-<shade>`.

### Blur

`blur-xs`, `blur-sm`, `blur-md`, `blur-lg`, `blur-xl`, `blur-2xl`, `blur-3xl`, `blur-none`, `blur-[<value>]`.

### Backdrop blur

Same scale, prefixed `backdrop-blur-*`.

### Opacity

`opacity-0`, `opacity-5`, `opacity-10`, ..., `opacity-100` (steps of 5). v4 dynamic : any integer 0-100. Arbitrary : `opacity-[0.37]`.

### Mix-blend / bg-blend

`mix-blend-normal`, `mix-blend-multiply`, `mix-blend-screen`, `mix-blend-overlay`, `mix-blend-darken`, `mix-blend-lighten`, `mix-blend-color-dodge`, `mix-blend-color-burn`, `mix-blend-hard-light`, `mix-blend-soft-light`, `mix-blend-difference`, `mix-blend-exclusion`, `mix-blend-hue`, `mix-blend-saturation`, `mix-blend-color`, `mix-blend-luminosity`, `mix-blend-plus-darker`, `mix-blend-plus-lighter`.

Same set with `bg-blend-*` prefix for background-blend-mode.

### Filter family

`brightness-<n>`, `contrast-<n>`, `grayscale-<n>`, `hue-rotate-<n>`, `invert-<n>`, `saturate-<n>`, `sepia-<n>`.

Same set with `backdrop-*` prefix for backdrop-filter.

## v3 vs v4 delta summary

| Concern | v3 | v4 |
|---------|----|----|
| Opacity per channel | `bg-opacity-50` etc. | REMOVED ; use `bg-red-500/50` |
| Shadow scale | `shadow-sm`, `shadow`, ..., `shadow-2xl` | INSERT `shadow-2xs`, `shadow-xs` before existing sizes |
| Same shift | n/a | applies to `blur-*`, `rounded-*`, `drop-shadow-*`, `backdrop-blur-*` |
| Default border color | `gray-200` | `currentColor` |
| Default ring width | 3px | 1px |
| `flex-shrink-*`, `flex-grow-*` | yes | RENAMED to `shrink-*`, `grow-*` |
| `overflow-ellipsis` | yes | RENAMED to `text-ellipsis` |
| `bg-gradient-to-r` | yes | RENAMED to `bg-linear-to-r` |
| `outline-none` (visually) | yes | RENAMED to `outline-hidden`; `outline-none` now zeros the outline |
| Important position | `!flex` | `flex!` |
| Dynamic spacing | scale-limited | any integer |
| Dynamic grid-cols | 1..12 default | any integer |
| Color palette | 22 families (legacy) | 22 families in oklch wide-gamut |

ALWAYS run `npx @tailwindcss/upgrade` to handle most renames automatically; see [tailwind-impl-migration-v3-v4].

## Sources

- https://tailwindcss.com/docs/padding
- https://tailwindcss.com/docs/colors
- https://tailwindcss.com/docs/font-size
- https://tailwindcss.com/docs/flex
- https://tailwindcss.com/docs/grid-template-columns
- https://tailwindcss.com/docs/box-shadow
- https://tailwindcss.com/docs/upgrade-guide
- https://v3.tailwindcss.com/docs

Verified 2026-05-19.
