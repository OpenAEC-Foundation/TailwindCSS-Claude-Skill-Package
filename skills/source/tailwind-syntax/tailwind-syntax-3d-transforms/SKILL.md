---
name: tailwind-syntax-3d-transforms
description: >
  Use when building 3D effects in Tailwind v4: card-flip animations, 3D carousels,
  product showcases, hover-reveal effects, parallax-like depth, or any layout that
  requires elements to rotate or translate along the Z axis. Use also when an
  existing v3 project is migrating to v4 and adopting depth effects for the first
  time, or when reading v4 markup that contains `transform-3d`, `perspective-*`,
  `rotate-x-*`, `rotate-y-*`, `rotate-z-*`, `translate-z-*`, or `backface-hidden`
  and needing to know what each utility does and what its parent must declare.
  Prevents (a) applying 3D rotations without declaring `perspective-*` on the parent,
  which renders flat with no depth, (b) forgetting `transform-3d` on the parent of
  a 3D scene, which collapses children into 2D space, (c) leaving the back of a
  card-flip element visible during the flip because `backface-hidden` was omitted,
  (d) reaching for these utilities in a Tailwind v3.4 project where NONE of the
  3D-axis utilities exist (they compile to nothing and the layout silently stays
  flat), (e) confusing the v3 single-axis `rotate-45` with v4 `rotate-z-45` (they
  produce identical CSS but v4 also offers `rotate-x-45` and `rotate-y-45` for
  3D scenes), and (f) putting `perspective-*` on the same element being rotated
  (perspective MUST live on the parent of the rotated element).
  Covers the complete v4 3D transform system: `transform-3d` vs `transform-flat`
  for the rendering context, the six `perspective-*` keyword utilities
  (perspective-none, perspective-dramatic 100px, perspective-near 300px,
  perspective-normal 500px, perspective-midrange 800px, perspective-distant 1200px),
  arbitrary `perspective-[800px]` and CSS-var `perspective-(--my-perspective)`,
  the three rotation axes `rotate-x-*`, `rotate-y-*`, `rotate-z-*` with the default
  number scale (0, 1, 2, 3, 6, 12, 45, 90, 180), negative `-rotate-y-30` and
  arbitrary `rotate-x-[45deg]` forms, `translate-z-*` for Z-axis offset (matches
  the spacing scale), `backface-visible` / `backface-hidden` for two-sided
  flipping cards, perspective-origin utilities, and the complete card-flip plus
  3D-carousel example patterns. ALL utilities here are Tailwind v4-only.
  Keywords: 3D transform, transform-3d, transform-style, preserve-3d, perspective,
  perspective-dramatic, perspective-near, perspective-distant, rotate-x, rotate-y,
  rotate-z, translate-z, z-axis, backface-hidden, backface-visible, card flip,
  flip card, 3D carousel, depth effect, product showcase, hover reveal, parallax,
  why does my rotation look flat, no depth, 2D fallback, v4 only utility,
  3D rendering context, perspective-origin, rotate around X axis, rotate around Y axis,
  Tailwind v4 features, Tailwind 4 3D, modern utilities.
license: MIT
compatibility: "Designed for Claude Code. Requires Tailwind CSS v4.0+."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# Tailwind CSS Syntax: 3D Transforms (v4-only)

Tailwind v4 introduces a complete 3D transform system: `perspective-*` to establish depth, `transform-3d` to opt children into a 3D rendering context, `rotate-x/y/z-*` for axis-aware rotations, `translate-z-*` for Z-axis offsets, and `backface-hidden` / `backface-visible` for two-sided elements. ALL of these utilities are v4-ONLY ; using them in a Tailwind v3.4 project produces no CSS and the layout stays flat.

## Quick Reference

### The 3D transform stack

| Layer : utility : where it goes : default |
|--|
| Depth : `perspective-*` : the PARENT of the 3D scene : `perspective: none` (no depth) |
| Rendering context : `transform-3d` : the PARENT of the rotated elements : `transform-style: flat` (default) |
| Rotation : `rotate-x-*`, `rotate-y-*`, `rotate-z-*` : the rotated element itself : 0 |
| Z-axis offset : `translate-z-*`, `-translate-z-*` : the offset element itself : 0 |
| Two-sided : `backface-visible` / `backface-hidden` : the element with a back face : `visible` |
| Origin : `perspective-origin-*` : the parent that has `perspective-*` : center |

### Top 3 patterns

```html
<!-- 1. Card flip on hover -->
<div class="group [perspective:1000px]">
  <div class="relative transition-transform duration-700 transform-3d group-hover:rotate-y-180">
    <div class="absolute inset-0 backface-hidden">Front</div>
    <div class="absolute inset-0 rotate-y-180 backface-hidden">Back</div>
  </div>
</div>

<!-- 2. 3D cube face -->
<div class="perspective-distant">
  <div class="relative transform-3d rotate-x-12 rotate-y-45">
    <div class="absolute translate-z-12 bg-blue-500">Front</div>
    <div class="absolute -translate-z-12 bg-blue-700">Back</div>
  </div>
</div>

<!-- 3. Subtle hover depth -->
<article class="perspective-normal">
  <div class="transition-transform hover:rotate-x-6 hover:translate-z-2">
    Lifts toward viewer on hover
  </div>
</article>
```

### Perspective keyword scale (default values)

| Keyword | CSS value | Effect |
|---------|-----------|--------|
| `perspective-none` | `perspective: none` | Disable depth |
| `perspective-dramatic` | `100px` | Extreme distortion ; very close camera |
| `perspective-near` | `300px` | Strong depth ; close camera |
| `perspective-normal` | `500px` | Standard depth |
| `perspective-midrange` | `800px` | Subtle depth |
| `perspective-distant` | `1200px` | Minimal depth ; far camera |

## Decision Trees

### Setting up a 3D scene (REQUIRED order)

```
need 3D effect?
├── STEP 1 : PARENT gets a perspective utility (perspective-distant, perspective-[800px], or perspective-(--my-p))
│   └── omit -> rotations appear flat (no depth)
├── STEP 2 : PARENT (same element OR a wrapper inside) gets transform-3d
│   └── omit -> children with translate-z-* collapse into 2D space
├── STEP 3 : CHILD gets the rotation/translation (rotate-x-*, rotate-y-*, rotate-z-*, translate-z-*)
│   └── omit perspective on ancestor -> CSS computes but renders flat
└── STEP 4 : if element has a back face (card flip), add backface-hidden to BOTH faces
    └── omit -> back of element shows through during/after rotation
```

### Picking a perspective value

```
how dramatic should the depth be?
├── product showcase (subtle camera-like depth) : `perspective-midrange` (800px) or `perspective-distant` (1200px)
├── card-flip animation (clear but not extreme) : `perspective-near` (300px) or `perspective-[1000px]`
├── 3D cube / carousel / extreme effect : `perspective-dramatic` (100px) or smaller arbitrary
├── responsive change : `perspective-midrange md:perspective-distant`
├── theme-driven : define `--perspective-brand: 1200px` in @theme, then `perspective-(--perspective-brand)`
└── disable depth conditionally : `perspective-none` in dark mode / on mobile / etc.
```

### Choosing `rotate-z-*` vs the legacy `rotate-*`

```
Tailwind v4 project with 3D usage?
├── only 2D rotations in this element : `rotate-45` (the legacy 2D utility still works)
└── element lives inside a transform-3d scene : ALWAYS prefer `rotate-z-45`
    └── reason : explicit axis avoids ambiguity ; legacy `rotate-*` becomes equivalent to rotate-z but the explicit form documents intent
```

### Picking `transform-3d` vs `transform-flat`

```
does this element contain children that need to render in 3D space?
├── yes (cube faces, parallax layers, 3D-cards stack) : `transform-3d`
├── no (single 2D element that happens to rotate-x) : default `transform-flat` is fine
└── responsive : `transform-3d md:transform-flat` to disable 3D on larger screens
```

## Patterns

### Pattern 1: Card flip on hover

The single most common 3D pattern. Requires perspective on the OUTER wrapper, `transform-3d` and the rotation on the INNER container, and `backface-hidden` on each face.

```html
<div class="group h-64 w-48 perspective-near">
  <div class="
    relative h-full w-full transition-transform duration-700 transform-3d
    group-hover:rotate-y-180
  ">
    <!-- Front face -->
    <div class="
      absolute inset-0 flex items-center justify-center
      rounded-lg bg-blue-500 text-white
      backface-hidden
    ">
      Front
    </div>
    <!-- Back face : pre-rotated 180 around Y, hidden by default -->
    <div class="
      absolute inset-0 flex items-center justify-center
      rounded-lg bg-blue-700 text-white
      rotate-y-180 backface-hidden
    ">
      Back
    </div>
  </div>
</div>
```

ALWAYS apply `backface-hidden` to BOTH faces. Without it, the back face shows through the front during the flip.

### Pattern 2: 3D cube

```html
<div class="perspective-distant grid place-items-center h-96">
  <div class="
    relative transform-3d
    h-32 w-32
    rotate-x-12 rotate-y-45
    animate-spin [animation-duration:8s]
  ">
    <div class="absolute inset-0 bg-red-500/80    translate-z-16">Front</div>
    <div class="absolute inset-0 bg-blue-500/80   -translate-z-16">Back</div>
    <div class="absolute inset-0 bg-green-500/80  rotate-y-90 translate-z-16">Right</div>
    <div class="absolute inset-0 bg-yellow-500/80 rotate-y-90 -translate-z-16">Left</div>
    <div class="absolute inset-0 bg-purple-500/80 rotate-x-90 translate-z-16">Top</div>
    <div class="absolute inset-0 bg-pink-500/80   rotate-x-90 -translate-z-16">Bottom</div>
  </div>
</div>
```

Each face is positioned along the Z axis (`translate-z-16` is the half-size of the cube : 32 / 2 in 0.25rem units = 4rem = 64px from centre).

### Pattern 3: Subtle hover lift

```html
<article class="perspective-midrange">
  <div class="
    rounded-xl bg-white p-6 shadow
    transition-transform duration-300
    hover:rotate-x-6 hover:translate-z-2 hover:shadow-2xl
  ">
    Card lifts toward viewer on hover.
  </div>
</article>
```

`perspective-midrange` (800px) keeps the effect subtle. `rotate-x-6` tilts the card 6 degrees ; `translate-z-2` pulls it 0.5rem toward the camera.

### Pattern 4: Animated 3D carousel

```html
<div class="perspective-distant overflow-hidden">
  <ul class="
    flex transform-3d
    animate-spin [animation-duration:20s]
    [transform-origin:center_center_-300px]
  ">
    <li class="rotate-y-0   translate-z-72 bg-red-500    p-6">Item 1</li>
    <li class="rotate-y-60  translate-z-72 bg-blue-500   p-6">Item 2</li>
    <li class="rotate-y-120 translate-z-72 bg-green-500  p-6">Item 3</li>
    <li class="rotate-y-180 translate-z-72 bg-yellow-500 p-6">Item 4</li>
    <li class="-rotate-y-120 translate-z-72 bg-purple-500 p-6">Item 5</li>
    <li class="-rotate-y-60  translate-z-72 bg-pink-500   p-6">Item 6</li>
  </ul>
</div>
```

`rotate-y-60` (and friends) are not in the default scale ; if not extending the theme, swap to arbitrary `rotate-y-[60deg]`. The default rotation scale (0, 1, 2, 3, 6, 12, 45, 90, 180) covers degrees ; for arbitrary multiples of 30 or 60 the arbitrary form is required.

### Pattern 5: Theme-driven perspective

```css
@import "tailwindcss";

@theme {
  --perspective-product:  1500px;
  --perspective-portrait:  600px;
}
```

```html
<section class="perspective-(--perspective-product)">
  <article class="transform-3d hover:rotate-y-12 hover:translate-z-3">...</article>
</section>
```

Now the project's two named perspectives are first-class utilities, swappable per breakpoint or per theme.

### Pattern 6: Conditional 3D (disable on mobile)

```html
<div class="perspective-none md:perspective-near">
  <div class="md:transform-3d transition-transform md:hover:rotate-y-180">
    Flips only on md+ screens.
  </div>
</div>
```

`perspective-none` removes the depth context ; the rotation still applies but renders flat (effectively a 2D mirror). On `md+`, the depth re-engages and the flip looks 3D.

## Reference Links

- `references/methods.md` : complete API (every perspective keyword + default value, every rotation axis + scale, transform-3d, transform-flat, translate-z, backface-*, perspective-origin).
- `references/examples.md` : worked examples for card flip, 3D cube, hover lift, carousel, theme-driven perspectives, responsive disable.
- `references/anti-patterns.md` : v3 silent failure, missing perspective parent, missing transform-3d, missing backface-hidden, applying perspective to the rotated element instead of its parent, default rotation scale gotchas.

## Verified Sources

- https://tailwindcss.com/docs/perspective (perspective utilities + keyword scale)
- https://tailwindcss.com/docs/rotate (rotate-x-*, rotate-y-*, rotate-z-* utilities)
- https://tailwindcss.com/docs/transform-style (transform-3d vs transform-flat, backface utilities, translate-z confirmation)
- https://tailwindcss.com/blog/tailwindcss-v4 (3D transform announcement + Oxide engine)

Last verified : 2026-05-19 (Tailwind CSS v4.x current).
