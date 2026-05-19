# Gradients : Side-by-Side Examples

Verified against https://tailwindcss.com/docs/background-image and
https://v3.tailwindcss.com/docs/background-image. v3 and v4 columns
shown together where the API diverges.

## 1. Horizontal Linear Gradient

### v3
```html
<div class="h-32 bg-gradient-to-r from-cyan-500 to-blue-500"></div>
```

### v4
```html
<div class="h-32 bg-linear-to-r from-cyan-500 to-blue-500"></div>
```

Only the prefix changed (`bg-gradient` -> `bg-linear`). Codemod handles it.

## 2. Three-Stop Gradient With via

### v3
```html
<div class="h-32 bg-gradient-to-r from-indigo-500 via-purple-500 to-pink-500"></div>
```

### v4
```html
<div class="h-32 bg-linear-to-r from-indigo-500 via-purple-500 to-pink-500"></div>
```

## 3. Diagonal Gradient

### v3 + v4
```html
<div class="h-32 bg-gradient-to-tr from-emerald-400 to-cyan-500"></div>
<div class="h-32 bg-linear-to-tr  from-emerald-400 to-cyan-500"></div>
```

## 4. Angle Gradient (45 degrees)

### v3 : arbitrary required
```html
<div class="h-32 bg-[linear-gradient(45deg,theme(colors.blue.500),theme(colors.purple.500))]"></div>
```

### v4 : utility
```html
<div class="h-32 bg-linear-45 from-blue-500 to-purple-500"></div>
```

## 5. Negative Angle

### v4
```html
<div class="h-32 -bg-linear-45 from-blue-500 to-purple-500"></div>
```

### v3
```html
<div class="h-32 bg-[linear-gradient(-45deg,theme(colors.blue.500),theme(colors.purple.500))]"></div>
```

## 6. Radial Gradient

### v3 : arbitrary required
```html
<div class="size-32 bg-[radial-gradient(circle,theme(colors.white),theme(colors.zinc.900))]"></div>
```

### v4
```html
<div class="size-32 bg-radial from-white to-zinc-900"></div>
```

## 7. Radial With Custom Position

### v3
```html
<div class="size-32 bg-[radial-gradient(at_25%_25%,theme(colors.white),theme(colors.zinc.900))]"></div>
```

### v4
```html
<div class="size-32 bg-radial-[at_25%_25%] from-white to-zinc-900"></div>
```

## 8. Conic Gradient

### v3
```html
<div class="size-32 bg-[conic-gradient(theme(colors.red.500),theme(colors.yellow.500),theme(colors.green.500),theme(colors.red.500))]"></div>
```

### v4
```html
<div class="size-32 bg-conic from-red-500 via-yellow-500 via-66% to-green-500"></div>
```

## 9. Conic At Specific Angle

### v4
```html
<div class="size-24 rounded-full bg-conic-180 from-red-500 to-blue-500"></div>
```

### v3
```html
<div class="size-24 rounded-full bg-[conic-gradient(from_180deg,theme(colors.red.500),theme(colors.blue.500))]"></div>
```

## 10. Rainbow Conic (full hue wheel)

### v4
```html
<div class="size-24 rounded-full
  bg-conic/[in_hsl_longer_hue]
  from-red-600 to-red-600"></div>
```

Uses the `longer-hue` interpolation arc so the gradient walks every hue
between two identical endpoints. v3 has no equivalent ; would require
hand-listing stops.

## 11. Stop Positions

### v4
```html
<div class="h-32 bg-linear-to-r
  from-blue-500 from-10%
  via-purple-500 via-30%
  to-pink-500 to-90%"></div>
```

### v3
```html
<div class="h-32 bg-[linear-gradient(to_right,theme(colors.blue.500)_10%,theme(colors.purple.500)_30%,theme(colors.pink.500)_90%)]"></div>
```

## 12. Negative Start Position (band of solid colour at edge)

### v4
```html
<div class="h-32 bg-linear-to-r
  -from-10% from-blue-500
  to-pink-500 to-100%"></div>
```

The `-from-10%` lets the gradient begin OUTSIDE the box on the left, so
the visible left edge is already partially mixed.

## 13. Interpolation : sRGB vs OKLCh

### v4 : compare two boxes
```html
<div class="grid grid-cols-2 gap-2">
  <div class="h-32 bg-linear-to-r/srgb  from-red-500 to-green-500"></div>
  <div class="h-32 bg-linear-to-r/oklch from-red-500 to-green-500"></div>
</div>
```

The `/srgb` version has a gray midpoint (muddy). The `/oklch` version
keeps the midpoint vivid.

### v3 : no native equivalent
v3 cannot select interpolation space via utilities ; must hand-write
CSS via `@layer utilities` or rely on browser default (which is sRGB).

## 14. Hero Section Background

### v4
```html
<section class="
  min-h-screen
  bg-linear-to-br/oklch
  from-indigo-700 via-purple-700 to-pink-700
  text-white
">
  <div class="container mx-auto py-32 px-6">
    <h1 class="text-5xl font-bold">Hello world</h1>
  </div>
</section>
```

### v3
```html
<section class="
  min-h-screen
  bg-gradient-to-br
  from-indigo-700 via-purple-700 to-pink-700
  text-white
">
  ...
</section>
```

## 15. Glow Button (radial)

### v4
```html
<button class="
  relative px-6 py-3 rounded font-medium text-white
  bg-radial-[at_50%_0%]
  from-blue-400 via-blue-600 to-blue-900
">
  Click me
</button>
```

### v3
```html
<button class="
  relative px-6 py-3 rounded font-medium text-white
  bg-[radial-gradient(at_50%_0%,theme(colors.blue.400),theme(colors.blue.600),theme(colors.blue.900))]
">
  Click me
</button>
```

## 16. Animated Spinner Ring (conic)

### v4
```html
<div class="
  size-16 rounded-full
  bg-conic from-blue-500 to-transparent
  motion-safe:animate-spin
  motion-reduce:opacity-50
"></div>
```

### v3
```html
<div class="
  size-16 rounded-full
  bg-[conic-gradient(theme(colors.blue.500),transparent)]
  motion-safe:animate-spin
  motion-reduce:opacity-50
"></div>
```

## 17. Status Badge (linear, slight angle)

### v4
```html
<span class="
  inline-block px-3 py-1 rounded-full text-xs font-medium text-white
  bg-linear-45 from-emerald-500 to-emerald-700
">
  Active
</span>
```

### v3
```html
<span class="
  inline-block px-3 py-1 rounded-full text-xs font-medium text-white
  bg-[linear-gradient(45deg,theme(colors.emerald.500),theme(colors.emerald.700))]
">
  Active
</span>
```

## 18. Dark-Mode-Aware Gradient

### v4
```html
<section class="
  bg-linear-to-r from-blue-100 to-purple-100
  dark:bg-linear-to-r dark:from-blue-900 dark:to-purple-900
">
  ...
</section>
```

### v3
```html
<section class="
  bg-gradient-to-r from-blue-100 to-purple-100
  dark:bg-gradient-to-r dark:from-blue-900 dark:to-purple-900
">
  ...
</section>
```

## 19. Hover-Reactive Gradient

### v4
```html
<button class="
  px-4 py-2 rounded text-white
  bg-linear-to-r from-blue-500 to-purple-500
  hover:from-blue-600 hover:to-purple-600
  active:from-blue-700 active:to-purple-700
">
  Submit
</button>
```

Same syntax in v3 with `bg-gradient-to-r`.

## 20. Reading Gradient From CSS Variable

### v4
```css
@theme {
  --gradient-brand: linear-gradient(135deg, #1da1f2, #6e36c5);
}
```

```html
<div class="h-32 bg-(--gradient-brand)"></div>
```

### v3 : arbitrary value referencing CSS variable
```css
:root {
  --gradient-brand: linear-gradient(135deg, #1da1f2, #6e36c5);
}
```

```html
<div class="h-32 bg-[var(--gradient-brand)]"></div>
```

## 21. Removing Gradient On Hover

### v4
```html
<div class="
  h-32 bg-linear-to-r from-blue-500 to-purple-500
  hover:bg-none hover:bg-white
"></div>
```

`bg-none` clears `background-image`. Add a solid `bg-{color}` after if you
want a visible background.

## 22. Full Migration Walk-Through

Starting v3 markup :
```html
<section class="bg-gradient-to-br from-blue-500 via-purple-500 to-pink-500"></section>
<aside class="bg-[radial-gradient(circle_at_top,theme(colors.white),theme(colors.zinc.100))]"></aside>
<header class="bg-[conic-gradient(from_45deg,theme(colors.red.500),theme(colors.blue.500))]"></header>
```

After `npx @tailwindcss/upgrade` (v4) :
```html
<section class="bg-linear-to-br from-blue-500 via-purple-500 to-pink-500"></section>
<aside class="bg-radial-[at_top] from-white to-zinc-100"></aside>
<header class="bg-conic-45 from-red-500 to-blue-500"></header>
```

The codemod handles the linear rename automatically. Radial and conic
arbitrary gradients are rewritten to the new utilities where it can
identify the pattern.
