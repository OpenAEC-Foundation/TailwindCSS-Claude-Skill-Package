# Tailwind CSS 3D Transforms: Worked Examples (v4-only)

Verified 2026-05-19 against tailwindcss.com/docs/perspective, /docs/rotate, /docs/transform-style.

## Example 1: Card flip on hover

```html
<div class="group h-64 w-48 perspective-near">
  <div class="
    relative h-full w-full
    transition-transform duration-700 ease-in-out
    transform-3d
    group-hover:rotate-y-180
  ">
    <!-- Front face -->
    <div class="
      absolute inset-0 flex items-center justify-center
      rounded-xl bg-gradient-to-br from-blue-500 to-blue-700
      text-white font-semibold
      backface-hidden
      shadow-xl
    ">
      Front
    </div>

    <!-- Back face : pre-rotated 180 around Y -->
    <div class="
      absolute inset-0 flex items-center justify-center
      rounded-xl bg-gradient-to-br from-violet-500 to-violet-700
      text-white font-semibold
      rotate-y-180 backface-hidden
      shadow-xl
    ">
      Back
    </div>
  </div>
</div>
```

Hovering the outer `group` rotates the middle wrapper 180deg around Y. Because both faces use `backface-hidden`, only the side currently facing the camera is visible.

## Example 2: 3D cube (six faces, slow auto-rotation)

```html
<div class="perspective-distant flex h-96 items-center justify-center">
  <div class="
    relative h-32 w-32 transform-3d
    rotate-x-12 rotate-y-45
    animate-spin [animation-duration:12s]
  ">
    <!-- Front -->
    <div class="absolute inset-0 flex items-center justify-center
                bg-red-500/80 text-white font-bold
                translate-z-16">F</div>

    <!-- Back -->
    <div class="absolute inset-0 flex items-center justify-center
                bg-blue-500/80 text-white font-bold
                -translate-z-16 rotate-y-180">B</div>

    <!-- Right -->
    <div class="absolute inset-0 flex items-center justify-center
                bg-green-500/80 text-white font-bold
                rotate-y-90 translate-z-16">R</div>

    <!-- Left -->
    <div class="absolute inset-0 flex items-center justify-center
                bg-yellow-500/80 text-white font-bold
                -rotate-y-90 translate-z-16">L</div>

    <!-- Top -->
    <div class="absolute inset-0 flex items-center justify-center
                bg-purple-500/80 text-white font-bold
                rotate-x-90 translate-z-16">T</div>

    <!-- Bottom -->
    <div class="absolute inset-0 flex items-center justify-center
                bg-pink-500/80 text-white font-bold
                -rotate-x-90 translate-z-16">Bot</div>
  </div>
</div>
```

`translate-z-16` = `calc(0.25rem * 16)` = 4rem = 64px. The cube is 8rem (128px) wide, so each face sits at half-width from centre.

## Example 3: Subtle hover lift

```html
<article class="perspective-midrange">
  <div class="
    rounded-xl bg-white p-6 shadow
    transition-transform duration-300
    hover:rotate-x-6 hover:translate-z-2 hover:shadow-2xl
  ">
    <h3 class="text-lg font-semibold">Hover me</h3>
    <p class="text-slate-600 mt-2">
      Tilts forward and lifts toward the viewer on hover. Subtle but
      noticeable depth effect.
    </p>
  </div>
</article>
```

`perspective-midrange` (800px) keeps the effect understated, suitable for a list of product cards.

## Example 4: 3D carousel with arbitrary rotations

```html
<div class="perspective-distant overflow-hidden h-96 flex items-center justify-center">
  <ul class="
    relative h-32 w-48 transform-3d
    animate-spin [animation-duration:20s]
  ">
    <li class="absolute inset-0 grid place-items-center
               rounded-lg bg-red-500/90 text-white
               rotate-y-[0deg] translate-z-[200px]">Slide 1</li>
    <li class="absolute inset-0 grid place-items-center
               rounded-lg bg-blue-500/90 text-white
               rotate-y-[60deg] translate-z-[200px]">Slide 2</li>
    <li class="absolute inset-0 grid place-items-center
               rounded-lg bg-green-500/90 text-white
               rotate-y-[120deg] translate-z-[200px]">Slide 3</li>
    <li class="absolute inset-0 grid place-items-center
               rounded-lg bg-yellow-500/90 text-white
               rotate-y-[180deg] translate-z-[200px]">Slide 4</li>
    <li class="absolute inset-0 grid place-items-center
               rounded-lg bg-purple-500/90 text-white
               rotate-y-[240deg] translate-z-[200px]">Slide 5</li>
    <li class="absolute inset-0 grid place-items-center
               rounded-lg bg-pink-500/90 text-white
               rotate-y-[300deg] translate-z-[200px]">Slide 6</li>
  </ul>
</div>
```

Six panels equally spaced (60deg apart) on a 200px-radius cylinder. Arbitrary `rotate-y-[60deg]` is required because 60 is not in the default scale.

## Example 5: Theme-driven perspective values

```css
/* src/app.css */
@import "tailwindcss";

@theme {
  --perspective-product:    1500px;
  --perspective-portrait:    600px;
  --perspective-cinema:      900px;
}
```

```html
<!-- Product page : long depth, near-flat -->
<section class="perspective-product">
  <article class="transform-3d transition-transform hover:rotate-y-6 hover:translate-z-1">
    ...
  </article>
</section>

<!-- Portrait gallery : short depth, more dramatic -->
<section class="perspective-portrait">
  <figure class="transform-3d hover:rotate-x-12 hover:translate-z-2">
    ...
  </figure>
</section>
```

## Example 6: Responsive 3D (disable on mobile)

```html
<div class="perspective-none md:perspective-near lg:perspective-midrange">
  <div class="md:transform-3d transition-transform duration-500 md:hover:rotate-y-180">
    <div class="md:backface-hidden">Front</div>
    <div class="md:backface-hidden md:rotate-y-180 absolute inset-0">Back</div>
  </div>
</div>
```

On phones the flip is effectively a 2D state change (no depth). On `md` and above the full 3D card-flip activates.

## Example 7: Perspective-origin for off-centre depth

```html
<div class="perspective-near perspective-origin-top-left h-64 w-64">
  <div class="transform-3d rotate-y-30 rotate-x-15 h-full w-full bg-blue-500 rounded-lg">
    Tilted as if the camera is positioned at the top-left of the parent.
  </div>
</div>
```

Default origin is `center` ; changing to `top-left` repositions the vanishing point.

## Example 8: Combining `backface-visible` with double-sided imagery

```html
<!-- A coin that shows DIFFERENT designs on each face -->
<div class="perspective-near group">
  <div class="
    relative h-40 w-40 transform-3d
    transition-transform duration-1000
    group-hover:rotate-y-180
  ">
    <img src="/heads.png" class="absolute inset-0 backface-hidden" />
    <img src="/tails.png" class="absolute inset-0 backface-hidden rotate-y-180" />
  </div>
</div>

<!-- A translucent panel that shows mirrored content on both sides : use backface-visible -->
<div class="perspective-near group">
  <div class="
    relative h-40 w-40 transform-3d
    transition-transform duration-1000
    group-hover:rotate-y-180
  ">
    <div class="absolute inset-0 bg-blue-500/50 backface-visible">
      Visible from both sides (mirrored)
    </div>
  </div>
</div>
```

`backface-hidden` for two distinct designs ; `backface-visible` (default) for shared content visible from both sides.

## Example 9: Combining 3D with hover-state interaction

```html
<article class="group perspective-near">
  <div class="
    relative h-48 w-32 transform-3d transition-transform duration-500
    group-hover:rotate-y-12
  ">
    <div class="absolute inset-0 rounded-lg bg-slate-100">
      <h3 class="p-4 text-slate-900">Card</h3>
    </div>
    <!-- Decorative shadow layer set BEHIND the card via -translate-z -->
    <div class="
      absolute inset-0 rounded-lg
      bg-slate-900/20 blur-md
      -translate-z-4
      group-hover:-translate-z-6 group-hover:blur-lg
      transition-transform
    "></div>
  </div>
</article>
```

The shadow layer sits behind the card (negative Z) and intensifies on hover, producing a parallax-like depth shift.

## Example 10: Sub-pixel arbitrary rotations

```html
<div class="perspective-near">
  <div class="transform-3d rotate-y-[17.5deg] rotate-x-[3.25deg]">
    Sub-degree precision for fine-tuning camera angles.
  </div>
</div>
```

Arbitrary values accept any valid CSS angle (`deg`, `rad`, `turn`). The default scale stops at integer degrees ; for non-integer angles use arbitrary syntax.
