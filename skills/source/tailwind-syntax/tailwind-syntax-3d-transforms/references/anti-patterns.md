# Tailwind CSS 3D Transforms: Anti-Patterns (v4-only)

Real failure modes that produce silent flat output or visual glitches.

## AP-1: Using v4 3D Utilities in a v3.4 Project

```html
<!-- Project on Tailwind v3.4 -->
<div class="perspective-near transform-3d rotate-y-180 translate-z-8 backface-hidden">
  v3 project trying v4 3D utilities
</div>
```

### Why this fails

NONE of `perspective-*`, `transform-3d`, `rotate-x-*`, `rotate-y-*`, `rotate-z-*`, `translate-z-*`, `backface-*` exist in Tailwind v3.4. The classes land in the DOM and produce zero CSS rules ; the element stays flat without errors.

### Fix : upgrade to v4 OR use arbitrary properties as a v3 workaround

ALWAYS upgrade to v4 for any project that needs 3D effects. The v3 workaround is verbose and ad-hoc :

```html
<!-- v3 workaround using arbitrary properties -->
<div class="
  [perspective:300px]
  [transform-style:preserve-3d]
  [transform:rotateY(180deg)_translateZ(2rem)]
  [backface-visibility:hidden]
">
  v3 with raw arbitrary CSS
</div>
```

This compiles but loses every Tailwind-aware feature (responsive variants, hover, theming). Migrate to v4 instead.

Source : https://tailwindcss.com/blog/tailwindcss-v4 (3D transforms section).

## AP-2: Missing `perspective-*` on the Parent

```html
<!-- BROKEN : rotation has no depth -->
<div>
  <div class="transform-3d rotate-y-45 translate-z-8 bg-blue-500 p-6">
    Rotates but looks flat. No depth distortion.
  </div>
</div>
```

### Why this fails

3D rotations only produce visual depth when an ancestor declares `perspective`. Without it, the CSS computes `transform: rotateY(45deg) translateZ(2rem)` but the rendering engine applies it as a flat 2D projection. The element ends up looking sheared or scaled, not rotated in space.

### Fix : ALWAYS add `perspective-*` to a parent

```html
<div class="perspective-near">
  <div class="transform-3d rotate-y-45 translate-z-8 bg-blue-500 p-6">
    Now rotates with proper depth.
  </div>
</div>
```

The perspective utility belongs on the PARENT of the rotated element, not on the rotated element itself.

Source : https://tailwindcss.com/docs/perspective.

## AP-3: Applying `perspective-*` to the Rotated Element Itself

```html
<!-- BROKEN : perspective on the same element -->
<div class="perspective-near transform-3d rotate-y-45 translate-z-8 bg-blue-500 p-6">
  Perspective set on the element being rotated.
</div>
```

### Why this fails

The CSS `perspective` property establishes a 3D viewing context for the element's CHILDREN. Setting it on the same element that is being rotated has no effect on its own rotation : the CSS engine uses the ancestor's perspective (which is none here), not the rotated element's own.

### Fix : separate perspective parent from rotated child

```html
<div class="perspective-near">
  <div class="transform-3d rotate-y-45 translate-z-8 bg-blue-500 p-6">
    Parent owns perspective ; child does the rotation.
  </div>
</div>
```

The two responsibilities (depth context + transform) belong on different elements.

Source : https://tailwindcss.com/docs/perspective.

## AP-4: Missing `transform-3d` on the Stack Parent

```html
<!-- BROKEN : translate-z collapses -->
<div class="perspective-near">
  <div class="relative h-32 w-32">
    <div class="absolute inset-0 translate-z-8 bg-red-500">Front</div>
    <div class="absolute inset-0 -translate-z-8 bg-blue-500">Back</div>
  </div>
</div>
```

### Why this fails

`transform-style` defaults to `flat`, which collapses every child's transform onto the parent's 2D plane. `translate-z-8` and `-translate-z-8` both render as 0 ; the two children overlap exactly with no depth separation.

### Fix : ALWAYS add `transform-3d` to the container holding the layered children

```html
<div class="perspective-near">
  <div class="relative h-32 w-32 transform-3d">
    <div class="absolute inset-0 translate-z-8 bg-red-500">Front</div>
    <div class="absolute inset-0 -translate-z-8 bg-blue-500">Back</div>
  </div>
</div>
```

Source : https://tailwindcss.com/docs/transform-style.

## AP-5: Forgetting `backface-hidden` on Card-Flip Faces

```html
<!-- BROKEN : both faces visible simultaneously -->
<div class="group perspective-near">
  <div class="relative transform-3d transition-transform group-hover:rotate-y-180">
    <div class="absolute inset-0 bg-blue-500">Front</div>
    <div class="absolute inset-0 rotate-y-180 bg-violet-500">Back</div>
  </div>
</div>
```

### Why this fails

By default `backface-visibility` is `visible`. During the rotation the back face is visible THROUGH the front (and vice versa), producing a confusing visual where text appears mirrored or both colors blend.

### Fix : add `backface-hidden` to BOTH faces

```html
<div class="group perspective-near">
  <div class="relative transform-3d transition-transform group-hover:rotate-y-180">
    <div class="absolute inset-0 bg-blue-500 backface-hidden">Front</div>
    <div class="absolute inset-0 rotate-y-180 bg-violet-500 backface-hidden">Back</div>
  </div>
</div>
```

ALWAYS apply `backface-hidden` to BOTH faces of a flip. Applying it to only one produces a half-visible state mid-rotation.

Source : https://tailwindcss.com/docs/transform-style (backface utilities).

## AP-6: Confusing `rotate-z-*` With Legacy `rotate-*`

```html
<!-- WORKS but mixes paradigms in a 3D scene -->
<div class="perspective-near">
  <div class="transform-3d rotate-12 translate-z-4">2D rotate in 3D scene</div>
</div>
```

### Why this is a smell

Legacy `rotate-12` compiles to `transform: rotate(12deg)`, which is equivalent to `rotateZ(12deg)`. In a flat 2D context they are identical. Inside a 3D scene, mixing `rotate-12` (2D) with `rotate-y-45` (3D) makes the code harder to read : a future maintainer cannot tell at a glance which axis the rotation targets.

### Fix : ALWAYS use the explicit axis form in 3D scenes

```html
<div class="perspective-near">
  <div class="transform-3d rotate-z-12 translate-z-4">Explicit Z axis</div>
</div>
```

Document a project-wide convention : if any element in a tree uses `rotate-x-*` or `rotate-y-*`, every rotation in that tree should be `rotate-x/y/z-*`.

## AP-7: Default Rotation Scale Gotchas

```html
<!-- BROKEN : 30 is not in the default scale -->
<div class="perspective-near">
  <div class="transform-3d rotate-y-30 translate-z-4">Rotate 30 degrees</div>
</div>
```

### Why this fails

The default rotation scale is `0, 1, 2, 3, 6, 12, 45, 90, 180`. Values like `30`, `60`, `120`, `240` are NOT generated. The class lands in the DOM but no CSS rule exists, and the element does not rotate.

### Fix : use arbitrary OR extend the theme

```html
<!-- FIX A : arbitrary value (one-off use) -->
<div class="rotate-y-[30deg]">30 degrees via arbitrary</div>

<!-- FIX B : extend the theme (repeated use) -->
<!-- @theme { --rotate-30: 30deg; --rotate-60: 60deg; } -->
<div class="rotate-y-30">Theme-extended 30 degrees</div>
```

For carousels with 6 panels spaced 60deg apart, arbitrary is the right tool ; for a project-wide hex-grid pattern, extend the theme once.

Source : https://tailwindcss.com/docs/rotate.

## AP-8: Setting `transform-3d` on the Element Being Rotated Instead of Its Parent

```html
<!-- BROKEN : transform-3d on the rotated element does nothing useful -->
<div class="perspective-near">
  <div class="transform-3d rotate-y-45 translate-z-4 bg-blue-500">
    Solo element with no 3D children
  </div>
</div>
```

### Why this is a smell

`transform-3d` (`transform-style: preserve-3d`) only matters when the element has CHILDREN that need to render in their own 3D plane. A solo rotated element does not need `transform-3d` because it has no children to preserve.

Setting it on a solo element is harmless but signals confusion : if it has no purpose here, the reader wonders why it is there.

### Fix : only add `transform-3d` to PARENTS of layered 3D children

```html
<!-- Solo rotation : no transform-3d needed -->
<div class="perspective-near">
  <div class="rotate-y-45 translate-z-4 bg-blue-500">Solo rotation</div>
</div>

<!-- Layered children : transform-3d required on the parent of the layers -->
<div class="perspective-near">
  <div class="relative h-32 w-32 transform-3d">
    <div class="absolute inset-0 translate-z-8 bg-red-500">Front</div>
    <div class="absolute inset-0 -translate-z-8 bg-blue-500">Back</div>
  </div>
</div>
```

Source : https://tailwindcss.com/docs/transform-style.

## AP-9: Animating Through 180deg With `transition-transform` on a Single Element

```html
<!-- BROKEN : flip looks wrong mid-rotation -->
<div class="perspective-near">
  <div class="
    h-32 w-32 bg-blue-500 transition-transform duration-700
    hover:rotate-y-180
  ">
    Single element flip ; back is just the front reversed
  </div>
</div>
```

### Why this fails

A single element rotating 180deg around Y shows the SAME content reversed (mirror image of the front). Without a back face, the user sees mirrored text, mirrored icons, and a confusing visual.

### Fix : two-face structure with `backface-hidden`

```html
<div class="group perspective-near">
  <div class="relative h-32 w-32 transition-transform duration-700 transform-3d group-hover:rotate-y-180">
    <div class="absolute inset-0 bg-blue-500 backface-hidden">Front</div>
    <div class="absolute inset-0 bg-violet-500 rotate-y-180 backface-hidden">Back</div>
  </div>
</div>
```

ALWAYS use the two-face pattern for card flips ; a single rotated element is wrong unless you specifically want the mirrored effect.

## AP-10: Reaching for 3D When 2D Suffices

```html
<!-- ANTI-PATTERN : 3D used for a simple icon rotation -->
<div class="perspective-near">
  <button class="transform-3d hover:rotate-z-45">
    <svg>...</svg>
  </button>
</div>
```

### Why this is a smell

A simple Z-axis rotation has no depth component and produces identical visual output to the legacy 2D `rotate-45`. The 3D stack (`perspective-*` + `transform-3d`) adds layout cost and reader confusion for zero visual benefit.

### Fix : use 2D utilities when the rotation has no depth component

```html
<button class="hover:rotate-45 transition-transform">
  <svg>...</svg>
</button>
```

Reserve 3D utilities for scenes that actually need depth : card flips, cubes, carousels, hover-lift, parallax. For chevrons, spinners, accordion arrows, sort indicators : plain 2D `rotate-*` is the right tool.

Source : https://tailwindcss.com/docs/rotate.
