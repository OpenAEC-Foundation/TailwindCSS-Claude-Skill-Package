# Responsive Design : Side-by-Side Examples

All examples verified against official docs. v3 and v4 versions provided
side-by-side for every pattern.

## 1. Setup : Container Queries Available

### v3 : install + register plugin

```bash
npm install -D @tailwindcss/container-queries
```

```js
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ['./src/**/*.{html,js,ts,jsx,tsx}'],
  plugins: [require('@tailwindcss/container-queries')],
}
```

```css
/* src/app.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### v4 : built-in

```bash
# Nothing extra to install.
```

```css
/* src/app.css */
@import "tailwindcss";
```

## 2. Mobile-First Layout

Same syntax in v3 and v4.

```html
<!-- text-center until 640px wide, then text-left -->
<p class="text-center sm:text-left">Hello</p>

<!-- one column until md, two columns up to lg, three columns from lg up -->
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  <div>A</div><div>B</div><div>C</div>
</div>
```

## 3. Single-Breakpoint Range

```html
<!-- bg-red only at md (768px to 1023px) -->
<div class="md:max-lg:bg-red-500">md only</div>

<!-- bg-yellow at md and lg (768px to 1279px) -->
<div class="md:max-xl:bg-yellow-300">md and lg</div>

<!-- arbitrary range : 1024px to 1399px -->
<div class="min-[1024px]:max-[1399px]:bg-blue-200"></div>
```

## 4. Custom Viewport Breakpoint

### v3

```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      screens: {
        '3xl': '120rem',
        'ultra': '120rem',
      },
    },
  },
}
```

```html
<div class="grid grid-cols-1 md:grid-cols-3 3xl:grid-cols-6">...</div>
```

### v4

```css
@import "tailwindcss";

@theme {
  --breakpoint-3xl: 120rem;
  --breakpoint-ultra: 120rem;
}
```

```html
<div class="grid grid-cols-1 md:grid-cols-3 3xl:grid-cols-6">...</div>
```

## 5. Reset All Default Breakpoints

### v3

```js
module.exports = {
  theme: {
    // REPLACE (no extend) drops every default
    screens: {
      'tablet': '40rem',
      'laptop': '64rem',
      'desktop': '80rem',
    },
  },
}
```

### v4

```css
@theme {
  --breakpoint-*: initial;
  --breakpoint-tablet: 40rem;
  --breakpoint-laptop: 64rem;
  --breakpoint-desktop: 80rem;
}
```

## 6. Container Queries : Basic

```html
<div class="@container">
  <div class="flex flex-col @md:flex-row gap-4">
    <img src="hero.png" class="w-full @md:w-1/2" />
    <p class="text-sm @md:text-base">
      Stacks below the @md breakpoint, side-by-side above.
    </p>
  </div>
</div>
```

Identical in v3 (with plugin installed) and v4 (built-in).

## 7. Named Containers

```html
<aside class="@container/sidebar">
  <section class="@container/card">
    <header class="@max-sm/sidebar:hidden">
      Header hidden when sidebar is small.
    </header>
    <p class="text-base @lg/card:text-lg">
      Body font scales by CARD size, not sidebar size.
    </p>
  </section>
</aside>
```

The `/sidebar` and `/card` suffixes scope each variant to its named
container. Without names, both variants would race for the nearest
ancestor.

## 8. Max-Container Variants

```html
<div class="@container">
  <div class="flex flex-row @max-md:flex-col">
    Row by default ; column when @container is BELOW the @md size.
  </div>
</div>
```

## 9. Range Inside a Container

```html
<div class="@container">
  <!-- Yellow only when container size is between @sm and @md -->
  <div class="@sm:@max-md:bg-yellow-100"></div>
</div>
```

## 10. Custom Container Size

### v4

```css
@import "tailwindcss";

@theme {
  --container-card: 30rem;
  --container-wide: 60rem;
}
```

```html
<div class="@container">
  <div class="@card:flex @wide:flex-row">...</div>
</div>
```

### v3 (plugin)

```js
module.exports = {
  theme: {
    extend: {
      containers: {
        card: '30rem',
        wide: '60rem',
      },
    },
  },
  plugins: [require('@tailwindcss/container-queries')],
}
```

```html
<div class="@container">
  <div class="@card:flex @wide:flex-row">...</div>
</div>
```

## 11. Container Query Units (v4 only)

```html
<div class="@container">
  <!-- 50% of container width -->
  <p class="w-[50cqw]"></p>

  <!-- font-size scales smoothly with container width -->
  <h1 class="text-[clamp(1rem,5cqw,3rem)]"></h1>
</div>
```

```html
<!-- Block-size containment required for cqb / cqh -->
<div class="@container-size">
  <div class="h-[60cqb] bg-blue-200"></div>
</div>
```

## 12. Arbitrary Viewport and Container Mixed

```html
<div class="@container">
  <div class="
    min-[1024px]:grid-cols-3
    @min-[475px]:gap-4
    @max-[960px]:p-2
  ">
    Page-level grid columns + container-local gap and padding.
  </div>
</div>
```

## 13. Real Layout : Responsive Card

### Viewport-only version

```html
<article class="rounded-md p-4 shadow-sm flex flex-col gap-3 md:flex-row md:gap-6">
  <img src="cover.jpg" class="w-full md:w-1/3 rounded" />
  <div>
    <h2 class="text-lg md:text-xl">Title</h2>
    <p class="text-sm md:text-base text-gray-600">Body</p>
  </div>
</article>
```

Always lays out the same regardless of where the card sits (sidebar,
main, modal). The breakpoint is the BROWSER size.

### Container-query version

```html
<article class="@container rounded-md p-4 shadow-sm">
  <div class="flex flex-col gap-3 @md:flex-row @md:gap-6">
    <img src="cover.jpg" class="w-full @md:w-1/3 rounded" />
    <div>
      <h2 class="text-lg @md:text-xl">Title</h2>
      <p class="text-sm @md:text-base text-gray-600">Body</p>
    </div>
  </div>
</article>
```

Now the card lays out based on its OWN width. Drop it into a narrow
sidebar : it stacks. Drop it into a wide main area : it goes side-by-side.
Independent of the browser size.

## 14. Migration : v3 + Plugin to v4

Before (v3) :

```bash
# package.json
npm install -D @tailwindcss/container-queries
```

```js
// tailwind.config.js
module.exports = {
  plugins: [require('@tailwindcss/container-queries')],
}
```

```html
<div class="@container">
  <div class="@md:flex">...</div>
</div>
```

After (v4) :

```bash
# Remove the plugin
npm uninstall @tailwindcss/container-queries
```

```css
/* No tailwind.config.js ; everything in CSS */
@import "tailwindcss";
```

```html
<!-- Markup unchanged -->
<div class="@container">
  <div class="@md:flex">...</div>
</div>
```

## 15. Custom Media Query (v3 raw vs v4 @custom-variant)

### v3 raw form

```js
module.exports = {
  theme: {
    screens: {
      'print': { raw: 'print' },
      'narrow-hover': { raw: '(any-hover: hover) and (max-width: 767px)' },
    },
  },
}
```

```html
<div class="print:hidden narrow-hover:bg-red-500"></div>
```

### v4 equivalent (via @custom-variant)

```css
@import "tailwindcss";

@custom-variant print (@media print);
@custom-variant narrow-hover (@media (any-hover: hover) and (max-width: 767px));
```

```html
<div class="print:hidden narrow-hover:bg-red-500"></div>
```

## 16. Removing a Default Breakpoint

### v3 : redefine `screens` without it

```js
module.exports = {
  theme: {
    screens: {
      sm: '640px',
      md: '768px',
      lg: '1024px',
      xl: '1280px',
      // 2xl intentionally omitted
    },
  },
}
```

### v4 : set to initial

```css
@theme {
  --breakpoint-2xl: initial;
}
```
