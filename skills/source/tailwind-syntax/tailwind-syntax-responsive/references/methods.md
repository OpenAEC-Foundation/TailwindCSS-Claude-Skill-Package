# Responsive Design : Complete API Reference

Source : https://tailwindcss.com/docs/responsive-design (v4) and
https://v3.tailwindcss.com/docs/responsive-design (v3), verified 2026-05-19.

## 1. Viewport Breakpoint Variants

### Min-width (default, mobile-first)

| Variant | Min width | v4 CSS | v3 CSS |
|---------|-----------|--------|--------|
| (none) | 0 | always | always |
| `sm:` | 40rem / 640px | `@media (width >= 40rem)` | `@media (min-width: 640px)` |
| `md:` | 48rem / 768px | `@media (width >= 48rem)` | `@media (min-width: 768px)` |
| `lg:` | 64rem / 1024px | `@media (width >= 64rem)` | `@media (min-width: 1024px)` |
| `xl:` | 80rem / 1280px | `@media (width >= 80rem)` | `@media (min-width: 1280px)` |
| `2xl:` | 96rem / 1536px | `@media (width >= 96rem)` | `@media (min-width: 1536px)` |

### Max-width (range targeting, since v3.4)

| Variant | Max width | v4 CSS |
|---------|-----------|--------|
| `max-sm:` | < 640px | `@media (width < 40rem)` |
| `max-md:` | < 768px | `@media (width < 48rem)` |
| `max-lg:` | < 1024px | `@media (width < 64rem)` |
| `max-xl:` | < 1280px | `@media (width < 80rem)` |
| `max-2xl:` | < 1536px | `@media (width < 96rem)` |

### Arbitrary

| Pattern | CSS |
|---------|-----|
| `min-[400px]:` | `@media (width >= 400px)` |
| `max-[600px]:` | `@media (width < 600px)` |
| `min-[35rem]:` | `@media (width >= 35rem)` |

### Stacking

```html
<!-- Single breakpoint range -->
<div class="md:max-lg:flex"></div>           <!-- md only (768-1023px) -->
<div class="md:max-xl:bg-red-500"></div>     <!-- md and lg (768-1279px) -->

<!-- Combine with other variants -->
<div class="hover:md:underline"></div>       <!-- hover on md and up -->
<div class="md:hover:underline"></div>       <!-- equivalent, order is intent only -->

<!-- Arbitrary range -->
<div class="min-[400px]:max-[799px]:hidden"></div>
```

## 2. Custom Viewport Breakpoints

### v3 : `theme.screens`

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  theme: {
    // REPLACE the defaults
    screens: {
      'tablet': '640px',
      'laptop': '1024px',
      'desktop': '1280px',
    },
    // OR : EXTEND without removing defaults
    extend: {
      screens: {
        'ultra': '1920px',
      },
    },
  },
}
```

Object form for custom media queries :

```js
screens: {
  'sm': '640px',
  'print': { raw: 'print' },
  'narrow': { max: '767px' },
  'wide': { min: '1280px', max: '1535px' },
}
```

### v4 : `@theme --breakpoint-*`

```css
@import "tailwindcss";

@theme {
  /* Add a new breakpoint */
  --breakpoint-xs: 30rem;
  --breakpoint-3xl: 120rem;

  /* Remove a default */
  --breakpoint-2xl: initial;

  /* Reset all defaults and define custom set */
  --breakpoint-*: initial;
  --breakpoint-tablet: 40rem;
  --breakpoint-laptop: 64rem;
}
```

v4 does NOT support `{ raw: '...' }` or `{ max, min }` objects ; the CSS
variable is a single length value. For non-length media queries (print,
hover), use `@custom-variant` instead :

```css
@custom-variant print (@media print);
```

## 3. Container Query Variants

### Min-width (default scale, v4 built-in)

| Variant | Min width |
|---------|-----------|
| `@3xs:` | 16rem / 256px |
| `@2xs:` | 18rem / 288px |
| `@xs:` | 20rem / 320px |
| `@sm:` | 24rem / 384px |
| `@md:` | 28rem / 448px |
| `@lg:` | 32rem / 512px |
| `@xl:` | 36rem / 576px |
| `@2xl:` | 42rem / 672px |
| `@3xl:` | 48rem / 768px |
| `@4xl:` | 56rem / 896px |
| `@5xl:` | 64rem / 1024px |
| `@6xl:` | 72rem / 1152px |
| `@7xl:` | 80rem / 1280px |

v3 plugin shipped `@xs` through `@7xl` only (12 sizes, no `@3xs` / `@2xs`).

### Max-width

| Variant | Max width |
|---------|-----------|
| `@max-sm:` | < 24rem |
| `@max-md:` | < 28rem |
| `@max-lg:` | < 32rem |
| `@max-xl:` | < 36rem |
| `@max-2xl:` | < 42rem |

### Arbitrary

| Pattern | CSS |
|---------|-----|
| `@min-[475px]:` | `@container (width >= 475px)` |
| `@max-[960px]:` | `@container (width < 960px)` |
| `@[17.5rem]:` | (v3 plugin syntax, equivalent to `@min-[17.5rem]:`) |

### Container marker

| Marker | Effect |
|--------|--------|
| `@container` | Mark as containment context (default : width only) |
| `@container/name` | Named container ; lets descendants target by name |
| `@container-size` | Enable block-size containment too (allows cqb / cqh units) |
| `@container-normal` | Reset to no containment (v3 plugin) |

### Named container targeting

```html
<div class="@container/sidebar">
  <div class="@md/sidebar:flex-row">
    Reacts to the parent named "sidebar", not the nearest unnamed @container.
  </div>
</div>
```

Combine with max/range :

```html
<div class="@container/card">
  <div class="@sm/card:@max-lg/card:bg-yellow-100"></div>
</div>
```

## 4. Custom Container Sizes

### v4

```css
@import "tailwindcss";

@theme {
  --container-8xl: 96rem;
  --container-card: 30rem;
  --container-*: initial;          /* reset all */
}
```

### v3 (plugin)

```js
module.exports = {
  theme: {
    extend: {
      containers: {
        '8xl': '96rem',
        'card': '30rem',
      },
    },
  },
  plugins: [require('@tailwindcss/container-queries')],
}
```

## 5. Container Query Length Units (v4 only)

| Unit | Meaning | Requires |
|------|---------|----------|
| `cqw` | 1% of container query width | `@container` ancestor |
| `cqi` | 1% of container query inline-size | `@container` ancestor |
| `cqh` | 1% of container query height | `@container-size` ancestor |
| `cqb` | 1% of container query block-size | `@container-size` ancestor |
| `cqmin` | smaller of cqi/cqb | depends on which axis is contained |
| `cqmax` | larger of cqi/cqb | depends on which axis is contained |

Used via arbitrary values :

```html
<div class="@container">
  <p class="w-[50cqw] text-[5cqi] p-[2cqw]">Scales with container.</p>
</div>
```

```html
<div class="@container-size">
  <div class="h-[60cqb]">Half the container height.</div>
</div>
```

## 6. v3 Plugin API Surface

Source : https://github.com/tailwindlabs/tailwindcss-container-queries.

```bash
npm install -D @tailwindcss/container-queries
```

```js
// tailwind.config.js
module.exports = {
  plugins: [require('@tailwindcss/container-queries')],
}
```

Provides : `@container`, `@xs:` through `@7xl:`, named-container syntax,
arbitrary `@[17.5rem]:`, `@container-normal` reset. Does NOT provide
`@max-*` (added later via core merge), `@container-size`, or the cq* units
(those are CSS-native and require the browser to support them ; the plugin
does not pre-process them).

## 7. v4 Removal of the Plugin

`@tailwindcss/container-queries` README, deprecation section : "As of
Tailwind CSS v4.0, container queries are supported in the framework by
default and this plugin is no longer required."

In a v4 project :

1. Remove from `package.json` devDependencies.
2. Remove from the v3 `tailwind.config.js` plugins array if you still
   load it via `@config "./tailwind.config.js";`.

The v4 built-in implementation is API-compatible with the plugin plus the
extra `@3xs` / `@2xs` sizes and the cq* length units.

## 8. Variant Order Rules

When stacking responsive + container + state variants :

| Order | Example | Effect |
|-------|---------|--------|
| viewport + container | `md:@lg:flex` | viewport md AND container @lg both true |
| state + viewport | `hover:md:underline` | hover, scoped to md and up |
| container + state | `@md:hover:underline` | hover, scoped to container @md and up |
| max + min | `md:max-lg:flex` | md only (single-breakpoint range) |
| arbitrary + named | `@min-[475px]/card:flex` | named-container arbitrary breakpoint |

v3 reads chains right-to-left ; v4 reads chains left-to-right. The chain
SEMANTICS are the same once the order is correct. See `tailwind-syntax-variants`
for the full stacking-order rules.

## 9. The `not-prefix` (intentionally separate)

`not-*:` variants (NOT containing) are a separate v4 grammar topic ; they
do not combine with breakpoint variants in any special way. See
`tailwind-syntax-variants`.

## 10. Browser Support

| Feature | Min browser version |
|---------|---------------------|
| Viewport breakpoints (min-width media) | all modern browsers |
| Container queries `@container` | Safari 16.4+, Chrome 105+, Firefox 110+ |
| cq* units | Same as container queries |
| `@container-size` (size containment) | Safari 16.4+, Chrome 105+, Firefox 110+ |

v4 viewport breakpoints use modern range syntax which still requires
Safari 16.4+ ; the v4 baseline. For older browser support, stay on v3.4.
