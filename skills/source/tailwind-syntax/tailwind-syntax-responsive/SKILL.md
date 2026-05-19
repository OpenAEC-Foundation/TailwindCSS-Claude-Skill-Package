---
name: tailwind-syntax-responsive
description: >
  Use when building responsive layouts with Tailwind CSS, picking between
  viewport breakpoints (sm/md/lg/xl/2xl) and container queries (@sm, @md,
  named @container/name), targeting a breakpoint range with max-* variants,
  using arbitrary breakpoints (min-[400px], max-[800px]), customising the
  default breakpoint set, or migrating a v3 codebase that installed
  @tailwindcss/container-queries to v4 where container queries are built-in.
  Prevents the mobile-first mistake (sm:text-center hiding text on mobile),
  the v3-to-v4 plugin trap (forgetting to remove the container-queries
  plugin in v4 still works but emits warnings), and the named-container
  scoping mistake (@sm:flex bleeding across nested @container parents).
  Covers viewport breakpoints, max-* range targeting, arbitrary breakpoints,
  custom breakpoint registration (v3 theme.screens vs v4 @theme
  --breakpoint-*), container queries (built-in v4 vs plugin v3), named
  containers, the full @3xs through @7xl scale, and container-query length
  units (cqw, cqh, cqi, cqb).
  Keywords: tailwind responsive, mobile-first, sm md lg xl 2xl, max-md,
  max-lg, arbitrary breakpoint, min-[400px], max-[800px], custom breakpoint,
  theme.screens, --breakpoint-md, @breakpoint, container query, @container,
  @container/name, named container, @sm @md @lg @xl @2xl @3xs @7xl,
  @tailwindcss/container-queries, container queries plugin, plugin
  deprecated v4, container query units, cqw, cqh, cqi, cqb, responsive
  layout broken on mobile, why does sm: hide my text, how do I add a custom
  breakpoint, container query not working, scoped responsive, intrinsic
  responsive, layout breakpoint vs container breakpoint.
license: MIT
compatibility: "Designed for Claude Code. Requires Tailwind CSS v3.4 or v4.0+."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# Tailwind CSS Responsive Design

Two independent systems exist : viewport breakpoints (page-level) and
container queries (component-level). Use viewport breakpoints to change
layout based on the BROWSER size ; use container queries to change layout
based on a specific PARENT container's size.

Companion skills :

- `tailwind-syntax-utility-classes` : the utilities you wrap in responsive prefixes
- `tailwind-syntax-variants` : the full variant grammar (state, attribute, custom)
- `tailwind-core-v3-vs-v4` : the broader v3-to-v4 differences

## Quick Reference : Default Viewport Breakpoints

Shared between v3 and v4 :

| Prefix | Min width | CSS (v4) | CSS (v3) |
|--------|-----------|----------|----------|
| `sm` | 40rem / 640px | `@media (width >= 40rem)` | `@media (min-width: 640px)` |
| `md` | 48rem / 768px | `@media (width >= 48rem)` | `@media (min-width: 768px)` |
| `lg` | 64rem / 1024px | `@media (width >= 64rem)` | `@media (min-width: 1024px)` |
| `xl` | 80rem / 1280px | `@media (width >= 80rem)` | `@media (min-width: 1280px)` |
| `2xl` | 96rem / 1536px | `@media (width >= 96rem)` | `@media (min-width: 1536px)` |

v4 emits modern range syntax (`width >= 40rem`). v3 emits classic
`min-width`. Functionally identical at runtime.

## The Mobile-First Rule

Unprefixed utilities apply at ALL screen sizes. Prefixed utilities
(`sm:`, `md:`, `lg:`, etc.) apply at THAT breakpoint AND UP.

```html
<!-- ALWAYS : center on mobile, left-align from 640px up -->
<p class="text-center sm:text-left">Hello</p>

<!-- NEVER : intended "center on small only" but actually applies from 640px up -->
<p class="sm:text-center">Hello</p>
```

NEVER write `sm:text-center` thinking it means "small screens only". It
means "from sm breakpoint upward". To target small-only, use `max-sm:`.

## Range Targeting With max-* Variants

| Variant | CSS (v4) | Effective range |
|---------|----------|-----------------|
| `max-sm` | `@media (width < 40rem)` | below 640px |
| `max-md` | `@media (width < 48rem)` | below 768px |
| `max-lg` | `@media (width < 64rem)` | below 1024px |
| `max-xl` | `@media (width < 80rem)` | below 1280px |
| `max-2xl` | `@media (width < 96rem)` | below 1536px |

Stack with a min variant to target a SINGLE breakpoint range :

```html
<!-- Applies at md only (768px to 1023px) -->
<div class="md:max-lg:flex"></div>

<!-- Applies at md and lg (768px to 1279px) -->
<div class="md:max-xl:bg-red-500"></div>
```

`max-*` was introduced in Tailwind v3.4 and remains available in v4 with
identical syntax.

## Arbitrary Breakpoints

For one-off values not in the breakpoint scale, use bracket notation :

```html
<div class="min-[400px]:flex max-[600px]:hidden">
  Visible from 400px up, hidden below 600px (so visible 600px to infinity).
</div>

<div class="min-[1400px]:max-[1599px]:bg-yellow-200">
  Yellow background only in the 1400px to 1599px range.
</div>
```

Works identically in v3 and v4.

## Custom Breakpoints

### v3 : theme.screens in tailwind.config.js

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  theme: {
    screens: {
      'sm': '640px',
      'md': '768px',
      'lg': '1024px',
      'xl': '1280px',
      '2xl': '1536px',
      'tablet': '640px',
      'laptop': '1024px',
      'desktop': '1280px',
      'ultra': '1920px',
    },
  },
}
```

Defining `screens` REPLACES the defaults. To EXTEND :

```js
module.exports = {
  theme: {
    extend: {
      screens: {
        'ultra': '1920px',
        '3xl': '1920px',
      },
    },
  },
}
```

### v4 : @theme --breakpoint-* in CSS

```css
@import "tailwindcss";

@theme {
  --breakpoint-xs: 30rem;
  --breakpoint-3xl: 120rem;
  --breakpoint-ultra: 120rem;
}
```

To REMOVE a default :

```css
@theme {
  --breakpoint-2xl: initial;
}
```

To RESET ALL and define custom :

```css
@theme {
  --breakpoint-*: initial;
  --breakpoint-tablet: 40rem;
  --breakpoint-laptop: 64rem;
  --breakpoint-desktop: 80rem;
}
```

ALWAYS use rem (not px) in v4 to align with the rest of the v4 token
scale. NEVER mix rem and px in the same breakpoint set : it produces
inconsistent breakpoints when the user changes root font-size.

## Container Queries : v3 Plugin vs v4 Built-In

### v3 : install the plugin

```bash
npm install -D @tailwindcss/container-queries
```

```js
// tailwind.config.js
module.exports = {
  plugins: [require('@tailwindcss/container-queries')],
}
```

Without the plugin, `@container` and `@sm:flex` simply do not generate any
CSS in v3.

### v4 : built-in (NEVER install the plugin)

```css
/* src/app.css */
@import "tailwindcss";
/* That's it. Container queries available immediately. */
```

The v3 plugin README explicitly notes : "As of Tailwind CSS v4.0, container
queries are supported in the framework by default and this plugin is no
longer required." Leaving the plugin in a v4 project is harmless but
useless dead weight.

Source : https://github.com/tailwindlabs/tailwindcss-container-queries

## Container Queries : The API

Mark a parent as a container, then use `@sm`, `@md`, etc. on descendants
to react to the CONTAINER's size (not the viewport).

```html
<div class="@container">
  <div class="flex flex-col @md:flex-row">
    Stack vertically until the @container reaches the @md size, then row.
  </div>
</div>
```

Default container size scale (v4 built-in : 13 sizes) :

| Variant | Min width |
|---------|-----------|
| `@3xs` | 16rem / 256px |
| `@2xs` | 18rem / 288px |
| `@xs` | 20rem / 320px |
| `@sm` | 24rem / 384px |
| `@md` | 28rem / 448px |
| `@lg` | 32rem / 512px |
| `@xl` | 36rem / 576px |
| `@2xl` | 42rem / 672px |
| `@3xl` | 48rem / 768px |
| `@4xl` | 56rem / 896px |
| `@5xl` | 64rem / 1024px |
| `@6xl` | 72rem / 1152px |
| `@7xl` | 80rem / 1280px |

The v3 plugin shipped fewer sizes (`@xs` through `@7xl`, 12 sizes, no
`@3xs` or `@2xs`). NEVER assume v3 has the full v4 scale.

## Container Queries : Max and Range

```html
<div class="@container">
  <div class="flex flex-row @max-md:flex-col">
    Row by default, column when @container is BELOW @md.
  </div>
</div>
```

| Variant | CSS |
|---------|-----|
| `@max-sm` | `@container (width < 24rem)` |
| `@max-md` | `@container (width < 28rem)` |
| `@max-lg` | `@container (width < 32rem)` |
| `@max-xl` | `@container (width < 36rem)` |
| `@max-2xl` | `@container (width < 42rem)` |

Range stacking :

```html
<div class="@container">
  <!-- Applies only when @container is between @sm and @md -->
  <div class="@sm:@max-md:bg-yellow-100"></div>
</div>
```

## Container Queries : Named Containers

When nested `@container` parents exist, name them to target a specific one :

```html
<div class="@container/sidebar">
  <div class="@container/card">
    <!-- React to the sidebar container's size -->
    <p class="text-sm @md/sidebar:text-base">Title</p>
    <!-- React to the card container's size -->
    <p class="text-xs @lg/card:text-sm">Body</p>
  </div>
</div>
```

ALWAYS name containers when they nest. Unnamed `@md:` targets the NEAREST
ancestor `@container`. Add a name to make the intent explicit.

## Container Queries : Arbitrary Sizes

```html
<div class="@container">
  <div class="@min-[475px]:flex-row @max-[960px]:flex-col"></div>
</div>
```

Identical bracket-notation pattern to viewport arbitrary breakpoints.

## Container Queries : Custom Sizes

### v4

```css
@import "tailwindcss";

@theme {
  --container-8xl: 96rem;
  --container-card: 30rem;
}
```

Then `@8xl:flex-row` or `@card:p-4`.

### v3 (plugin)

```js
// tailwind.config.js
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

## Container Query Length Units (v4)

v4 supports `cqw`, `cqi`, `cqh`, `cqb` units inside a container :

```html
<div class="@container">
  <p class="w-[50cqw] text-[5cqi]">
    Width is 50% of container width ; font-size is 5% of container inline-size.
  </p>
</div>
```

| Unit | Meaning |
|------|---------|
| `cqw` | 1% of container query width |
| `cqh` | 1% of container query height |
| `cqi` | 1% of container query inline-size (logical width) |
| `cqb` | 1% of container query block-size (logical height) |

The `@container-size` variant on the parent enables block-size containers :

```html
<div class="@container-size">
  <div class="h-[50cqb]"></div>
</div>
```

NEVER use `cqb` or `cqh` without `@container-size` on a parent : block-size
containment is opt-in because the browser cost is higher than width-only.

## Decision Tree : Viewport vs Container Query

```
Does the layout react to PAGE size (browser window)?
├── YES → Use viewport breakpoints : sm:flex md:grid-cols-2
│         Independent of where the component sits in the tree.
│
└── NO  → Does the layout react to a SPECIFIC PARENT'S size?
    ├── YES → Use container queries : @container on parent, @md:flex-row on child.
    │         Component-intrinsic ; renders correctly in any layout slot.
    │
    └── BOTH → Combine : @container the component, use both viewport
               prefixes for page-level concerns AND @sm/@md for
               container-local concerns. They compose with `:` between.
```

ALWAYS prefer container queries for reusable components (cards, sidebars,
nav items). ALWAYS use viewport breakpoints for page-level layout (grid
templates, header height).

## v3 to v4 Migration : Container Queries

Two steps :

1. Remove `@tailwindcss/container-queries` from `package.json` devDependencies.
2. Remove `require('@tailwindcss/container-queries')` from the legacy
   `tailwind.config.js` plugins array (if you kept a v3 config via `@config`).

Existing `@container` and `@sm:` class usage works without changes : the
v4 built-in implementation is API-compatible with the v3 plugin (plus the
two extra `@3xs` and `@2xs` sizes).

NEVER keep the plugin installed and active in v4 : it tries to register
the same variants and produces silent ordering oddities.

## Reference Files

- `references/methods.md` : every variant, every size, full breakpoint and
  container query API
- `references/examples.md` : side-by-side v3 vs v4 setup, named containers,
  custom breakpoints, length units, real layouts
- `references/anti-patterns.md` : mobile-first reversal, max-* range
  confusion, named-container bleeding, plugin-left-in-v4, cqb without
  @container-size

## Verified Sources

- https://tailwindcss.com/docs/responsive-design (v4 viewport + container)
- https://github.com/tailwindlabs/tailwindcss-container-queries (v3 plugin + deprecation)
- https://v3.tailwindcss.com/docs/responsive-design (v3 viewport baseline)
