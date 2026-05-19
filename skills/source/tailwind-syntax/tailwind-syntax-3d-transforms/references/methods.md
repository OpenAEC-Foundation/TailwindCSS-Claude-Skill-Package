# Tailwind CSS 3D Transforms: Complete Reference (v4-only)

Verified 2026-05-19 against tailwindcss.com/docs/perspective, /docs/rotate, /docs/transform-style.

ALL utilities listed here are Tailwind v4.0+ exclusive. None exist in v3.4.

## `transform-style`

| Utility | CSS | Default |
|---------|-----|---------|
| `transform-3d` | `transform-style: preserve-3d;` | no (must opt in) |
| `transform-flat` | `transform-style: flat;` | yes (browser default) |

Applies to the PARENT of 3D-positioned children. Required when children use `translate-z-*` or you want them to remain in their own 3D plane.

## `perspective-*`

| Utility | CSS variable | Default px |
|---------|--------------|-----------|
| `perspective-none` | (literal) `perspective: none` | n/a |
| `perspective-dramatic` | `var(--perspective-dramatic)` | 100px |
| `perspective-near` | `var(--perspective-near)` | 300px |
| `perspective-normal` | `var(--perspective-normal)` | 500px |
| `perspective-midrange` | `var(--perspective-midrange)` | 800px |
| `perspective-distant` | `var(--perspective-distant)` | 1200px |

Arbitrary : `perspective-[800px]`, `perspective-[50vw]`.
CSS variable : `perspective-(--my-perspective)` (v4 parens shorthand) or `perspective-[var(--my-perspective)]` (brackets form).

Applies to the PARENT of the rotated element. Setting perspective on the rotated element itself is a common mistake : the depth context is created for the element's own children, not for the element itself.

## `perspective-origin-*`

| Utility | CSS |
|---------|-----|
| `perspective-origin-center` | `perspective-origin: center;` |
| `perspective-origin-top` | `perspective-origin: top;` |
| `perspective-origin-top-right` | `perspective-origin: top right;` |
| `perspective-origin-right` | `perspective-origin: right;` |
| `perspective-origin-bottom-right` | `perspective-origin: bottom right;` |
| `perspective-origin-bottom` | `perspective-origin: bottom;` |
| `perspective-origin-bottom-left` | `perspective-origin: bottom left;` |
| `perspective-origin-left` | `perspective-origin: left;` |
| `perspective-origin-top-left` | `perspective-origin: top left;` |

Arbitrary : `perspective-origin-[25%_75%]`, `perspective-origin-(--my-origin)`.

## `rotate-x-*` / `rotate-y-*` / `rotate-z-*`

### Default scale

| Suffix | Degrees |
|--------|---------|
| `0` | 0deg |
| `1` | 1deg |
| `2` | 2deg |
| `3` | 3deg |
| `6` | 6deg |
| `12` | 12deg |
| `45` | 45deg |
| `90` | 90deg |
| `180` | 180deg |

### Per-axis utilities

| Utility | CSS |
|---------|-----|
| `rotate-x-{N}` | `transform: rotateX({N}deg) ...` |
| `rotate-y-{N}` | `transform: ... rotateY({N}deg) ...` |
| `rotate-z-{N}` | `transform: ... rotateZ({N}deg) ...` |
| `-rotate-x-{N}` | negative rotation on X |
| `-rotate-y-{N}` | negative rotation on Y |
| `-rotate-z-{N}` | negative rotation on Z |

### Arbitrary

| Pattern | Result |
|---------|--------|
| `rotate-x-[37.5deg]` | exact X rotation |
| `rotate-y-[-22deg]` | negative inside brackets |
| `rotate-x-(--my-rotation)` | CSS variable (v4 parens) |
| `rotate-x-[var(--my-rotation)]` | CSS variable (brackets form) |

### Composition with legacy `rotate-*`

The 2D-only `rotate-{N}` utility (from earlier Tailwind versions) is equivalent to `rotate-z-{N}` in the 2D case. In 3D scenes ALWAYS use the explicit `rotate-z-*` form.

## `translate-z-*`

| Utility | CSS |
|---------|-----|
| `translate-z-{N}` | `transform: translateZ(calc(var(--spacing) * N))` |
| `-translate-z-{N}` | negative Z offset |
| `translate-z-px` | 1px Z offset |
| `translate-z-[120px]` | arbitrary Z offset |
| `translate-z-(--my-depth)` | CSS variable (v4 parens) |
| `translate-z-[var(--my-depth)]` | CSS variable (brackets) |

The numeric scale uses the global `--spacing` unit (0.25rem default). `translate-z-16` = `calc(0.25rem * 16)` = 4rem = 64px.

Requires the PARENT to have `transform-3d` for the offset to render with depth ; otherwise it collapses to 0.

## `translate-3d-*` (composite, optional)

Tailwind v4 also accepts `translate-3d-[x_y_z]` arbitrary syntax for setting all three axes in one declaration. The split utilities (`translate-x-*`, `translate-y-*`, `translate-z-*`) compose cleanly and are usually preferred for clarity.

## `backface-*`

| Utility | CSS |
|---------|-----|
| `backface-visible` | `backface-visibility: visible;` (browser default) |
| `backface-hidden` | `backface-visibility: hidden;` |

Apply to elements that have a "back side" the user should not see. The canonical use is the two faces of a flip-card : both faces get `backface-hidden`, the front shows when rotation is 0deg, the back (pre-rotated 180deg) shows when its rotation aligns toward the viewer.

## Required Setup Recap

| Step | Where it goes | Utility |
|------|---------------|---------|
| 1. Establish depth | parent of the 3D scene | `perspective-*` or arbitrary |
| 2. Preserve 3D rendering | parent of the rotated/translated children | `transform-3d` |
| 3. Apply transform | the element itself | `rotate-x-*`, `rotate-y-*`, `rotate-z-*`, `translate-z-*` |
| 4. Hide back side (optional) | every face of a 2-sided element | `backface-hidden` |
| 5. Move depth focal point (optional) | the perspective parent | `perspective-origin-*` |

Skipping step 1 produces flat output (rotation computes but no depth). Skipping step 2 collapses children to 2D (translate-z renders as 0). Skipping step 4 on card flips leaks the back face through the front.

## Theme Customisation

```css
@import "tailwindcss";

@theme {
  --perspective-product:    1500px;
  --perspective-portrait:    600px;
  --perspective-cinema:      900px;
}
```

Now `perspective-product`, `perspective-portrait`, `perspective-cinema` work everywhere alongside the defaults.

## Animations Compatible With 3D

| Utility | Use with 3D |
|---------|-------------|
| `transition-transform` | smooth rotation/translation interpolation |
| `transition-all` | broadest sweep ; can be expensive |
| `duration-N` | timing |
| `ease-*` | curve |
| `animate-spin` | continuous rotation (combined with `[animation-duration:Ns]` arbitrary) |
| `animate-pulse` | depth-pulsing effect with translate-z |
| Custom `@keyframes` in `@theme { --animate-* }` | full control |
