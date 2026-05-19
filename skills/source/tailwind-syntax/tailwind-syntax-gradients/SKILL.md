---
name: tailwind-syntax-gradients
description: >
  Use when building linear, conic, or radial gradient backgrounds, picking
  an interpolation colour space (oklab default, oklch, srgb, hsl, longer-hue,
  shorter-hue), placing color stops at specific positions, or migrating a
  v3 `bg-gradient-to-r from-X to-Y` markup to v4's expanded gradient API
  with `bg-linear-*`, `bg-radial-*`, and `bg-conic-*`. Prevents the
  rename-trap (every v3 `bg-gradient-*` class is renamed to `bg-linear-*`
  in v4), the muddy-middle trap (sRGB interpolation produces a gray midpoint
  between saturated complementary colours where oklch keeps it vivid), the
  conic-without-from trap (`bg-conic` without a `from-{color}` produces an
  invisible gradient), and the v3-radial-by-arbitrary trap (v3 has no
  native bg-radial-* and forces `bg-[radial-gradient(...)]`). Covers every
  linear direction utility, the angle-based form (bg-linear-45), the
  conic and radial families, every from-/via-/to- color and position stop,
  and the seven interpolation modifiers.
  Keywords: tailwind gradient, bg-linear, bg-linear-to-r, bg-linear-45,
  bg-radial, bg-radial-[at_top_left], bg-conic, bg-conic-90, from-,
  via-, to-, from-10%, via-30%, to-90%, color stops, gradient position,
  interpolation modifier, oklab, oklch, srgb, hsl, longer-hue, shorter-hue,
  in_oklab, bg-gradient-to-r v3, bg-gradient renamed, gradient migration
  v3 to v4, conic gradient tailwind, radial gradient tailwind, vivid
  gradient, muddy gradient, gray midpoint, gradient middle gray,
  rainbow gradient, angle gradient.
license: MIT
compatibility: "Designed for Claude Code. Requires Tailwind CSS v3.4 or v4.0+."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# Tailwind CSS Gradients

v4 expanded the gradient API from v3's eight linear directions to a full
three-family system (linear, radial, conic) with seven interpolation
colour spaces, angle-based linear gradients, and percentage stop positions.
v3 stays linear-only with the legacy `bg-gradient-*` prefix.

Companion skills :

- `tailwind-syntax-utility-classes` : the broader background utility surface
- `tailwind-core-v3-vs-v4` : the v4-renamed utilities table (bg-gradient -> bg-linear)
- `tailwind-impl-migration-v3-v4` : codemod that handles the rename

## Quick Reference : v3 vs v4 Cheat Sheet

| Need | v3 | v4 |
|------|----|----|
| Linear horizontal | `bg-gradient-to-r from-blue-500 to-purple-500` | `bg-linear-to-r from-blue-500 to-purple-500` |
| Linear diagonal | `bg-gradient-to-tr from-A to-B` | `bg-linear-to-tr from-A to-B` |
| Linear at custom angle | `bg-[linear-gradient(45deg,#A,#B)]` (arbitrary) | `bg-linear-45 from-A to-B` |
| Radial | `bg-[radial-gradient(circle,#A,#B)]` (arbitrary) | `bg-radial from-A to-B` |
| Radial at position | `bg-[radial-gradient(at_top_left,#A,#B)]` (arbitrary) | `bg-radial-[at_top_left] from-A to-B` |
| Conic | `bg-[conic-gradient(#A,#B)]` (arbitrary) | `bg-conic from-A to-B` |
| Conic at angle | `bg-[conic-gradient(from_90deg,#A,#B)]` (arbitrary) | `bg-conic-90 from-A to-B` |
| Vivid interpolation | not available natively | `bg-linear-to-r/oklch from-red-500 to-blue-500` |
| Stop position 10% | `bg-[linear-gradient(to_right,#A_10%,#B)]` (arbitrary) | `bg-linear-to-r from-A from-10% to-B` |

ALWAYS run `npx @tailwindcss/upgrade` on v3-to-v4 migration : it renames
every `bg-gradient-*` to `bg-linear-*` automatically. NEVER hand-edit
the rename across a large codebase ; the codemod is exhaustive.

Source : https://tailwindcss.com/blog/tailwindcss-v4 ("we've renamed
`bg-gradient-*` to `bg-linear-*`").

## Linear Gradients : v4 Direction Utilities

| Utility | Direction |
|---------|-----------|
| `bg-linear-to-t` | bottom to top |
| `bg-linear-to-tr` | bottom-left to top-right |
| `bg-linear-to-r` | left to right |
| `bg-linear-to-br` | top-left to bottom-right |
| `bg-linear-to-b` | top to bottom |
| `bg-linear-to-bl` | top-right to bottom-left |
| `bg-linear-to-l` | right to left |
| `bg-linear-to-tl` | bottom-right to top-left |

Identical eight directions exist in v3 with `bg-gradient-to-*` prefix.

## Linear Gradients : Angle Utilities (v4-only)

```html
<div class="bg-linear-45 from-indigo-500 to-pink-500"></div>
<div class="bg-linear-90 from-red-500 to-blue-500"></div>
<div class="-bg-linear-45 from-indigo-500 to-pink-500"></div>
```

| Pattern | CSS |
|---------|-----|
| `bg-linear-{N}` | `linear-gradient({N}deg in oklab, var(--tw-gradient-stops))` |
| `-bg-linear-{N}` | negative angle (e.g. `-45deg`) |
| `bg-linear-(--my-angle)` | reads from CSS variable |
| `bg-linear-[25deg,red_5%,yellow_60%,lime_90%,teal]` | fully arbitrary |

Angle utilities REPLACE the v3 arbitrary-only pattern
`bg-[linear-gradient(45deg,#A,#B)]`. NEVER use the arbitrary form in v4
for simple angle gradients ; the utility is shorter and uses the configured
interpolation space.

## Radial Gradients (v4)

```html
<div class="bg-radial from-white to-zinc-900"></div>
<div class="bg-radial-[at_top_left] from-blue-500 to-transparent"></div>
<div class="bg-radial-[at_25%_25%] from-white to-zinc-900 to-75%"></div>
```

| Pattern | CSS shape |
|---------|-----------|
| `bg-radial` | `radial-gradient(in oklab, var(--tw-gradient-stops))` |
| `bg-radial-[at_top_left]` | `radial-gradient(at top left in oklab, ...)` |
| `bg-radial-[at_25%_25%]` | offset position |
| `bg-radial-[circle]` | force a circle shape |
| `bg-radial-[circle_at_top]` | shape + position |
| `bg-radial-[ellipse_closest-side]` | shape + size keyword |
| `bg-radial-(--my-shape)` | CSS variable |

v3 has NO native radial utility. Use arbitrary
`bg-[radial-gradient(circle,#A,#B)]` or stay on v3.

Source : https://tailwindcss.com/docs/background-image

## Conic Gradients (v4)

```html
<div class="bg-conic from-red-500 via-yellow-500 to-red-500"></div>
<div class="bg-conic-90 from-red-500 to-blue-500"></div>
<div class="bg-conic-180 from-red-500 to-blue-500"></div>
<div class="-bg-conic-45 from-red-500 to-blue-500"></div>
```

| Pattern | CSS shape |
|---------|-----------|
| `bg-conic` | `conic-gradient(in oklab, var(--tw-gradient-stops))` |
| `bg-conic-{N}` | `conic-gradient(from {N}deg in oklab, ...)` |
| `-bg-conic-{N}` | negative starting angle |
| `bg-conic-[from_45deg_at_50%_50%]` | full arbitrary |
| `bg-conic-(--my-conic)` | CSS variable |

ALWAYS include `from-{color}` AND `to-{color}` (and typically `via-{color}`)
on a `bg-conic`. NEVER use `bg-conic` with only one stop : the gradient
collapses to a single colour and the conic effect is invisible.

For a smooth full-circle rainbow :

```html
<div class="size-24 rounded-full
  bg-conic/[in_hsl_longer_hue] from-red-600 to-red-600"></div>
```

The `longer-hue` interpolation walks the entire colour wheel between two
identical endpoints.

## Color Stops : Colour Utilities

```html
<div class="bg-linear-to-r from-indigo-500 via-purple-500 to-pink-500"></div>
```

| Utility | CSS variable |
|---------|--------------|
| `from-{color}` | `--tw-gradient-from` |
| `via-{color}` | `--tw-gradient-via` |
| `to-{color}` | `--tw-gradient-to` |

ALWAYS pair `from-` with `to-`. NEVER ship a gradient with only `from-` ;
the `to-` defaults to transparent, which means the gradient fades to
nothing.

## Color Stops : Position Utilities (v4)

```html
<!-- from at 10%, via at 30%, to at 90% -->
<div class="bg-linear-to-r
  from-blue-500 from-10%
  via-purple-500 via-30%
  to-pink-500 to-90%"></div>

<!-- start the gradient OUTSIDE the box for a partial visible band -->
<div class="bg-linear-to-r
  -from-10% from-blue-500
  to-pink-500"></div>
```

| Utility | Sets |
|---------|------|
| `from-{N%}` | `--tw-gradient-from-position` |
| `via-{N%}` | `--tw-gradient-via-position` |
| `to-{N%}` | `--tw-gradient-to-position` |
| `-from-{N%}` | negative starting position (gradient begins outside box) |

v3 has NO native position utilities. Use arbitrary
`bg-[linear-gradient(to_right,#A_10%,#B)]` or stay on v3.

## Interpolation Modifiers (v4)

```html
<div class="bg-linear-to-r/srgb   from-indigo-500 to-teal-400"></div>
<div class="bg-linear-to-r/oklab  from-indigo-500 to-teal-400"></div>   <!-- default -->
<div class="bg-linear-to-r/oklch  from-indigo-500 to-teal-400"></div>
<div class="bg-linear-to-r/hsl    from-indigo-500 to-teal-400"></div>
<div class="bg-linear-to-r/longer from-red-500    to-red-500"></div>
<div class="bg-linear-to-r/shorter from-red-500   to-blue-500"></div>
<div class="bg-linear-to-r/increasing from-red-500 to-blue-500"></div>
<div class="bg-linear-to-r/decreasing from-red-500 to-blue-500"></div>
```

| Modifier | Colour space | Use when |
|----------|--------------|----------|
| `/oklab` | OKLab | DEFAULT in v4 ; perceptually uniform, no muddy midpoints |
| `/oklch` | OKLCh | Polar variant of OKLab ; produces VERY vivid hue paths |
| `/srgb` | sRGB | Legacy CSS default ; produces gray midpoints between complementaries |
| `/hsl` | HSL | Polar; vivid rotation around hue wheel |
| `/longer` | (hue-based) longer-hue arc | Rainbows, full-circle conic gradients |
| `/shorter` | (hue-based) shorter-hue arc | Smooth two-colour transitions |
| `/increasing` | hue increasing | Forward hue rotation |
| `/decreasing` | hue decreasing | Reverse hue rotation |

Source : https://tailwindcss.com/blog/tailwindcss-v4 ("Using polar color
spaces like OKLCH or HSL can lead to much more vivid gradients when the
from-* and to-* colors are far apart on the color wheel. We're using OKLAB
by default in v4.0").

NEVER use `/srgb` for a saturated red to saturated green gradient unless
you actively want the gray midpoint. The default `/oklab` and the polar
`/oklch` both keep the midpoint vivid.

## Decision Tree : Which Interpolation?

```
Are the two colours close on the hue wheel (e.g. blue and purple)?
├── YES → default /oklab works
│         (no special modifier needed)
│
└── NO  → Far apart (red and green, blue and orange)?
    ├── Smooth perceptual blend → /oklch
    ├── Vivid hue rotation through wheel → /hsl
    ├── Full circle through every hue → /longer
    └── Want legacy CSS look (gray midpoint) → /srgb (explicit opt-in)
```

ALWAYS test `/oklch` against the default `/oklab` for complementary-colour
gradients ; the difference is dramatic. NEVER ship a red-to-green gradient
on `/srgb` by accident : you get a muddy gray middle.

## Arbitrary Values

Every utility supports the `[value]` and `(--var)` escape hatches :

```html
<!-- Full arbitrary linear gradient -->
<div class="bg-linear-[25deg,red_5%,yellow_60%,lime_90%,teal]"></div>

<!-- Conic with explicit from + center + colour stops -->
<div class="bg-conic-[from_45deg_at_50%_25%,red,blue,red]"></div>

<!-- Reading angle from CSS variable -->
<div class="bg-linear-(--my-angle) from-blue-500 to-purple-500"></div>
```

Underscores convert to spaces in arbitrary values : `bg-linear-[to_right]`
emits `linear-gradient(to right, ...)`.

## v3 Pattern : Linear-Only With Arbitrary For Everything Else

```html
<!-- v3 simple linear : utility available -->
<div class="bg-gradient-to-r from-blue-500 to-purple-500"></div>

<!-- v3 angle gradient : MUST be arbitrary -->
<div class="bg-[linear-gradient(45deg,theme(colors.blue.500),theme(colors.purple.500))]"></div>

<!-- v3 radial : MUST be arbitrary -->
<div class="bg-[radial-gradient(circle,theme(colors.white),theme(colors.zinc.900))]"></div>

<!-- v3 conic : MUST be arbitrary -->
<div class="bg-[conic-gradient(from_45deg,theme(colors.red.500),theme(colors.blue.500))]"></div>

<!-- v3 stop positions : arbitrary -->
<div class="bg-[linear-gradient(to_right,theme(colors.blue.500)_10%,theme(colors.pink.500)_90%)]"></div>
```

`theme(colors.blue.500)` resolves the configured Tailwind colour at build
time in v3. v4 dropped this in favour of `var(--color-blue-500)`.

## Stacking With Variants

```html
<!-- Hover changes gradient direction -->
<div class="bg-linear-to-r hover:bg-linear-to-br
  from-blue-500 to-purple-500"></div>

<!-- Dark-mode variant changes colours -->
<div class="bg-linear-to-r from-blue-200 to-purple-200
  dark:from-blue-700 dark:to-purple-700"></div>

<!-- Responsive changes interpolation -->
<div class="bg-linear-to-r/srgb md:bg-linear-to-r/oklch
  from-red-500 to-green-500"></div>
```

The gradient utility and its colour stops are independent variants ; each
gets its own prefix when conditional.

## Reference Files

- `references/methods.md` : every utility with CSS output, default values,
  interpolation modifier list, browser baseline
- `references/examples.md` : side-by-side v3 vs v4 for every gradient
  category, real layouts (hero, button, badge, glow, conic spinner)
- `references/anti-patterns.md` : the rename trap, muddy-middle, lonely
  `from-`, conic-without-from, v3 angle-not-supported confusion

## Verified Sources

- https://tailwindcss.com/docs/background-image (v4 full API)
- https://v3.tailwindcss.com/docs/background-image (v3 baseline)
- https://tailwindcss.com/blog/tailwindcss-v4 (announcement, default oklab,
  rename, interpolation rationale)
