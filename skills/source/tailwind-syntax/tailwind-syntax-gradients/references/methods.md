# Gradients : Complete API Reference

Verified against https://tailwindcss.com/docs/background-image (v4) and
https://v3.tailwindcss.com/docs/background-image (v3), 2026-05-19.

## 1. Linear Gradient Utilities

### Direction (v4 : `bg-linear-*`, v3 : `bg-gradient-*`)

| v4 | v3 | CSS |
|----|----|-----|
| `bg-linear-to-t` | `bg-gradient-to-t` | `linear-gradient(to top, ...)` |
| `bg-linear-to-tr` | `bg-gradient-to-tr` | `linear-gradient(to top right, ...)` |
| `bg-linear-to-r` | `bg-gradient-to-r` | `linear-gradient(to right, ...)` |
| `bg-linear-to-br` | `bg-gradient-to-br` | `linear-gradient(to bottom right, ...)` |
| `bg-linear-to-b` | `bg-gradient-to-b` | `linear-gradient(to bottom, ...)` |
| `bg-linear-to-bl` | `bg-gradient-to-bl` | `linear-gradient(to bottom left, ...)` |
| `bg-linear-to-l` | `bg-gradient-to-l` | `linear-gradient(to left, ...)` |
| `bg-linear-to-tl` | `bg-gradient-to-tl` | `linear-gradient(to top left, ...)` |

### Angle (v4 only)

| Pattern | CSS |
|---------|-----|
| `bg-linear-0` | `linear-gradient(0deg in oklab, ...)` |
| `bg-linear-45` | `linear-gradient(45deg in oklab, ...)` |
| `bg-linear-90` | `linear-gradient(90deg in oklab, ...)` |
| `bg-linear-135` | `linear-gradient(135deg in oklab, ...)` |
| `bg-linear-180` | `linear-gradient(180deg in oklab, ...)` |
| `-bg-linear-45` | `linear-gradient(-45deg in oklab, ...)` |
| `bg-linear-(--angle)` | reads CSS variable |
| `bg-linear-[<value>]` | full arbitrary linear-gradient |

v3 has no native angle utility. Use arbitrary :

```html
<div class="bg-[linear-gradient(45deg,theme(colors.blue.500),theme(colors.purple.500))]"></div>
```

## 2. Radial Gradient Utilities (v4 only)

| Pattern | CSS |
|---------|-----|
| `bg-radial` | `radial-gradient(in oklab, ...)` |
| `bg-radial-[at_top_left]` | `radial-gradient(at top left in oklab, ...)` |
| `bg-radial-[at_25%_25%]` | offset position |
| `bg-radial-[circle]` | `radial-gradient(circle in oklab, ...)` |
| `bg-radial-[circle_at_top]` | shape + position |
| `bg-radial-[ellipse_closest-side]` | shape + size keyword |
| `bg-radial-[circle_at_50%_50%_closest-side]` | all three |
| `bg-radial-(--my-shape)` | CSS variable |

CSS shape keywords inside the arbitrary brackets : `circle`, `ellipse`.
Size keywords : `closest-side`, `closest-corner`, `farthest-side`,
`farthest-corner` (default).

v3 radial requires arbitrary :

```html
<div class="bg-[radial-gradient(circle_at_25%_25%,#fff,#000)]"></div>
```

## 3. Conic Gradient Utilities (v4 only)

| Pattern | CSS |
|---------|-----|
| `bg-conic` | `conic-gradient(in oklab, ...)` |
| `bg-conic-0` | `conic-gradient(from 0deg in oklab, ...)` |
| `bg-conic-90` | `conic-gradient(from 90deg in oklab, ...)` |
| `bg-conic-180` | `conic-gradient(from 180deg in oklab, ...)` |
| `-bg-conic-45` | negative starting angle |
| `bg-conic-[from_45deg_at_50%_50%]` | full arbitrary |
| `bg-conic-(--my-conic)` | CSS variable |

v3 conic requires arbitrary :

```html
<div class="bg-[conic-gradient(from_45deg,#f00,#0f0,#00f,#f00)]"></div>
```

## 4. Color Stop Utilities

### Colour values

| Utility | CSS variable | Default |
|---------|--------------|---------|
| `from-<color>` | `--tw-gradient-from` | none (transparent if absent) |
| `via-<color>` | `--tw-gradient-via` | absent by default ; two-stop gradient |
| `to-<color>` | `--tw-gradient-to` | transparent (variant of from-color) |

Both v3 and v4 share this surface. Slash-modifier opacity also works :

```html
<div class="bg-linear-to-r from-blue-500/80 to-purple-500/50"></div>
```

### Stop positions (v4 only)

| Utility | CSS variable |
|---------|--------------|
| `from-{N%}` | `--tw-gradient-from-position` |
| `via-{N%}` | `--tw-gradient-via-position` |
| `to-{N%}` | `--tw-gradient-to-position` |
| `-from-{N%}` | negative start |
| `-via-{N%}` | negative via position |
| `-to-{N%}` | negative end |

Positions accept any percentage from `-100%` to `100%+` (the CSS spec
allows values outside the box).

```html
<div class="bg-linear-to-r
  from-blue-500 from-10%
  via-purple-500 via-50%
  to-pink-500 to-90%"></div>
```

v3 stop positions require arbitrary :

```html
<div class="bg-[linear-gradient(to_right,theme(colors.blue.500)_10%,theme(colors.pink.500)_90%)]"></div>
```

## 5. Interpolation Modifiers (v4)

Syntax : append `/{mode}` to any gradient utility.

| Modifier | Colour space | Visual effect |
|----------|--------------|---------------|
| `/oklab` | OKLab | DEFAULT ; perceptually uniform |
| `/oklch` | OKLCh | Polar OKLab ; vivid hue paths |
| `/srgb` | sRGB | Legacy default of plain CSS ; can muddy midpoints |
| `/hsl` | HSL | Polar ; vivid rotation around hue wheel |
| `/longer` | hue (longer arc) | Walks full hue wheel |
| `/shorter` | hue (shorter arc) | Smooth two-colour rotation |
| `/increasing` | hue increasing | Forward rotation |
| `/decreasing` | hue decreasing | Reverse rotation |

```html
<div class="bg-linear-to-r/oklch from-red-500 to-green-500"></div>
<div class="bg-conic/[in_hsl_longer_hue] from-red-600 to-red-600"></div>
```

Compound forms via arbitrary inside `[]` :
`bg-conic/[in_hsl_longer_hue]` -> `conic-gradient(in hsl longer hue, ...)`.

## 6. Arbitrary Values and CSS Variables

| Pattern | Use |
|---------|-----|
| `bg-linear-[25deg,red_5%,yellow_60%,lime_90%,teal]` | Full gradient string |
| `bg-radial-[at_25%_25%]` | Just position |
| `bg-conic-[from_45deg_at_50%_50%]` | Conic with from + at |
| `bg-linear-(--my-gradient)` | Read whole gradient from CSS var |
| `from-(--my-color)` | Read from-color from CSS var |
| `from-[#1da1f2]` | Arbitrary colour |
| `from-[50%]` | Arbitrary position |

Underscores inside `[]` convert to spaces. Commas remain commas.

## 7. Removing a Gradient

| Utility | Effect |
|---------|--------|
| `bg-none` | `background-image: none` ; clears any gradient or image |

```html
<div class="bg-linear-to-r from-blue-500 to-purple-500 hover:bg-none">
  Removes gradient on hover.
</div>
```

## 8. Customising Gradient Colour Palette

The colour utilities `from-<color>` / `via-<color>` / `to-<color>` accept
every colour configured in the theme. To add a custom colour :

### v3 : `tailwind.config.js`

```js
module.exports = {
  theme: {
    extend: {
      colors: {
        brand: { DEFAULT: '#1da1f2', dark: '#0d8ddf' },
      },
    },
  },
}
```

```html
<div class="bg-gradient-to-r from-brand to-brand-dark"></div>
```

### v4 : `@theme` in CSS

```css
@import "tailwindcss";

@theme {
  --color-brand: #1da1f2;
  --color-brand-dark: #0d8ddf;
}
```

```html
<div class="bg-linear-to-r from-brand to-brand-dark"></div>
```

## 9. Browser Support

| Feature | Min Safari | Min Chrome | Min Firefox |
|---------|-----------|-----------|------------|
| Linear gradients | All modern | All modern | All modern |
| Radial gradients | All modern | All modern | All modern |
| Conic gradients | 12.1 | 69 | 83 |
| `color-mix()` interpolation | 16.4 | 111 | 113 |
| OKLab / OKLch | 15.4 | 111 | 113 |

v4's baseline (Safari 16.4+, Chrome 111+, Firefox 128+) covers all of
these. The interpolation modifier compiles to `color-mix()` calls or
`linear-gradient(... in oklch ...)` syntax depending on browser support.

## 10. The `--tw-gradient-*` Custom Properties (advanced)

Tailwind sets these CSS variables internally :

| Variable | Default | Set by |
|----------|---------|--------|
| `--tw-gradient-from` | transparent | `from-{color}` |
| `--tw-gradient-via` | absent | `via-{color}` |
| `--tw-gradient-to` | transparent | `to-{color}` |
| `--tw-gradient-from-position` | 0% | `from-{N%}` |
| `--tw-gradient-via-position` | 50% | `via-{N%}` |
| `--tw-gradient-to-position` | 100% | `to-{N%}` |
| `--tw-gradient-stops` | composed string | the utility itself |

NEVER read or write these variables in user code. They are internal and
subject to change. Use the public utilities only.

## 11. v3 to v4 Migration Mapping

| v3 | v4 |
|----|----|
| `bg-gradient-to-t` | `bg-linear-to-t` |
| `bg-gradient-to-tr` | `bg-linear-to-tr` |
| `bg-gradient-to-r` | `bg-linear-to-r` |
| `bg-gradient-to-br` | `bg-linear-to-br` |
| `bg-gradient-to-b` | `bg-linear-to-b` |
| `bg-gradient-to-bl` | `bg-linear-to-bl` |
| `bg-gradient-to-l` | `bg-linear-to-l` |
| `bg-gradient-to-tl` | `bg-linear-to-tl` |

The `npx @tailwindcss/upgrade` codemod handles every direction in one
pass. Source : https://tailwindcss.com/blog/tailwindcss-v4 ("we've renamed
`bg-gradient-*` to `bg-linear-*`").
