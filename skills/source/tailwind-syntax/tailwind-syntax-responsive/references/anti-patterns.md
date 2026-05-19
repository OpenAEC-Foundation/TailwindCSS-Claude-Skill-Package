# Responsive Design : Anti-Patterns

Each entry maps a real failure to its root cause and the verified fix.
Sources : https://tailwindcss.com/docs/responsive-design,
https://v3.tailwindcss.com/docs/responsive-design,
https://github.com/tailwindlabs/tailwindcss-container-queries.

## AP-1 : `sm:` Used to Mean "Small Screens Only"

### Symptom
A class like `sm:text-center` is used hoping to center text on mobile
phones. Result : on a 360px-wide phone, the text is NOT centered ; it only
centers from 640px upward.

### Root cause
Tailwind is mobile-first. Prefixed utilities apply at the breakpoint AND
ABOVE, not "below" or "at" only. `sm:` means "from 640px up".

### NEVER
```html
<p class="sm:text-center">Hello</p>
```

### ALWAYS use unprefixed for mobile, prefix for larger
```html
<!-- center on mobile, left-align from 640px up -->
<p class="text-center sm:text-left">Hello</p>

<!-- if you really want center only on small screens : use max-sm -->
<p class="max-sm:text-center sm:text-left">Hello</p>
```

## AP-2 : Forgetting `@tailwindcss/container-queries` In v3

### Symptom
v3 project uses `<div class="@container">...<p class="@md:text-lg">` but
nothing happens. CSS output contains no `@container` rules.

### Root cause
In v3, container queries are a SEPARATE plugin. Without
`@tailwindcss/container-queries` installed AND registered in
`tailwind.config.js`, the `@container` and `@sm:` variants generate no CSS.

### ALWAYS in v3
```bash
npm install -D @tailwindcss/container-queries
```

```js
// tailwind.config.js
module.exports = {
  plugins: [require('@tailwindcss/container-queries')],
}
```

### NEVER assume `@container` is built into v3 ; it is only built into v4.

## AP-3 : Keeping the Container-Queries Plugin Active In v4

### Symptom
v4 project still has `@tailwindcss/container-queries` in package.json AND
in a legacy `tailwind.config.js` loaded via `@config "./..."`. Container
query CSS is generated TWICE, and variant ordering is unpredictable.

### Root cause
v4 ships container queries in core. The plugin tries to register the same
variants, leading to duplicate definitions.

### NEVER
```bash
# v4 project
npm install -D @tailwindcss/container-queries     # delete this
```

```js
// legacy tailwind.config.js still loaded in v4
module.exports = {
  plugins: [require('@tailwindcss/container-queries')],     // delete this
}
```

### ALWAYS in v4 : uninstall the plugin AND remove from legacy config

```bash
npm uninstall @tailwindcss/container-queries
```

The v3 plugin README states explicitly : "As of Tailwind CSS v4.0,
container queries are supported in the framework by default and this
plugin is no longer required."

## AP-4 : Named Container Targeting Without a Name

### Symptom
Nested `@container` parents both define container contexts. A child uses
`@md:flex` and reacts to the WRONG container (often the innermost ancestor).

### Root cause
Unnamed `@md:` always resolves to the NEAREST ancestor `@container`. When
that's not the parent you meant, layout breaks unpredictably.

### NEVER nest unnamed containers
```html
<aside class="@container">
  <section class="@container">
    <p class="@md:text-lg">Which container am I reacting to?</p>
  </section>
</aside>
```

### ALWAYS name containers when they nest
```html
<aside class="@container/sidebar">
  <section class="@container/card">
    <p class="@md/sidebar:text-lg">Reacts to the sidebar size.</p>
    <p class="@md/card:text-base">Reacts to the card size.</p>
  </section>
</aside>
```

## AP-5 : Using `cqb` or `cqh` Without `@container-size`

### Symptom
`<div class="h-[50cqb]">` renders with height 0 or with unpredictable
fallback heights. Browser inspector shows `cqb` resolving to nothing.

### Root cause
`cqb` and `cqh` units require block-size containment. The default
`@container` only enables INLINE (width) containment for performance.

### NEVER
```html
<div class="@container">
  <div class="h-[50cqb]">Height collapses to 0</div>
</div>
```

### ALWAYS opt into size containment
```html
<div class="@container-size">
  <div class="h-[50cqb]">Half the container height.</div>
</div>
```

Size containment is more expensive than width-only ; opt into it only
where actually needed.

## AP-6 : Trying `max-[400px]:` Without the Right Syntax

### Symptom
`<div class="max-[400px]:flex">` produces a class that does not match any
generated CSS rule. Layout does not change at 400px.

### Root cause
The arbitrary syntax for max-width is `max-[400px]:`, which IS correct.
The most common confusion is using `max-[400px]:` thinking it caps at 400px,
but `max-` means "ABOVE this is false", i.e. "applies BELOW 400px".

### ALWAYS read the semantics out loud
```html
<!-- "from 400px and up" -->
<div class="min-[400px]:flex"></div>

<!-- "below 400px" -->
<div class="max-[400px]:hidden"></div>

<!-- "between 400px and 800px" -->
<div class="min-[400px]:max-[800px]:bg-yellow-200"></div>
```

## AP-7 : Treating Container Queries As Viewport Replacement

### Symptom
A page-level layout uses container queries everywhere. As soon as a parent
DOES NOT have `@container`, nothing reacts to the viewport.

### Root cause
`@sm:`, `@md:` etc require an ancestor with `@container`. They DO NOT
react to the browser size. Container queries are component-intrinsic, not
page-level.

### NEVER use container queries for top-level page layout
```html
<!-- Wrong : no @container ancestor, so @md never matches -->
<body>
  <header class="@md:flex">...</header>
</body>
```

### ALWAYS use viewport breakpoints for page-level
```html
<body>
  <header class="md:flex">...</header>
</body>
```

Use `@container` + `@md:` for COMPONENT-LEVEL layout that should adapt to
the slot the component sits in.

## AP-8 : Mixing rem and px in Custom Breakpoints

### Symptom
Setting `--breakpoint-sm: 640px` and `--breakpoint-md: 48rem` produces
inconsistent layout behaviour when the user changes their root font-size
in browser preferences.

### Root cause
rem scales with user font-size preference. px does not. A mixed scale has
breakpoints that move relative to each other depending on user settings.

### NEVER mix units
```css
@theme {
  --breakpoint-sm: 640px;        /* px : fixed */
  --breakpoint-md: 48rem;        /* rem : scales */
}
```

### ALWAYS pick ONE unit per scale (v4 defaults are all rem)
```css
@theme {
  --breakpoint-sm: 40rem;
  --breakpoint-md: 48rem;
  --breakpoint-lg: 64rem;
}
```

## AP-9 : Calling `@container/name` Without Slash-Suffix Selector

### Symptom
```html
<div class="@container/sidebar">
  <p class="@md:text-lg">Doesn't react to sidebar.</p>
</div>
```
The `@md:` without a slash-suffix targets ANY nearest `@container`,
including unnamed ones. Inside a single named container with no nesting,
the result is usually correct by accident, but it breaks as soon as
another container is introduced.

### NEVER target a named container with a bare variant
```html
<div class="@container/sidebar">
  <p class="@md:text-lg">Implicit ; brittle.</p>
</div>
```

### ALWAYS use the slash-suffix when the container is named
```html
<div class="@container/sidebar">
  <p class="@md/sidebar:text-lg">Explicit ; safe under refactoring.</p>
</div>
```

## AP-10 : Custom Breakpoint With `extend.screens` Replacing Defaults in v3

### Symptom
v3 `tailwind.config.js` defines `theme.screens = { ultra: '120rem' }`.
After build, `md:flex` produces no CSS. The whole default breakpoint set
disappeared.

### Root cause
In v3, defining `theme.screens` REPLACES the defaults. To EXTEND, use
`theme.extend.screens`.

### NEVER (v3)
```js
module.exports = {
  theme: {
    screens: { ultra: '120rem' },     // wipes out sm/md/lg/xl/2xl
  },
}
```

### ALWAYS (v3)
```js
module.exports = {
  theme: {
    extend: {
      screens: { ultra: '120rem' },   // ADDS to defaults
    },
  },
}
```

In v4 the same semantic distinction exists via `--breakpoint-*: initial;`
to clear defaults.

## AP-11 : Stacking `@container` and Page-Level Variant in the Wrong Place

### Symptom
```html
<div class="md:@container">
  <p class="@sm:flex-row">Variant resolves at the wrong layer.</p>
</div>
```
`md:@container` means "apply @container only when viewport is md+". Below
md, `@sm:flex-row` has no ancestor container and silently does nothing.

### NEVER conditionally mark a container with viewport variants
```html
<div class="md:@container">...</div>      <!-- container only above md -->
```

### ALWAYS mark `@container` unconditionally
```html
<div class="@container">
  <div class="@sm:flex-row">Always reacts to container size.</div>
</div>
```

If you genuinely need viewport-gated container behaviour, restructure the
DOM so the container marker is always present and switch behaviour with
viewport variants on the children.

## AP-12 : Forgetting to Reset The `2xl` Default in v4

### Symptom
v4 project targets desktop layouts. The `2xl` breakpoint (96rem / 1536px)
sits awkwardly between the project's chosen `xl` (90rem) and `3xl` (120rem)
custom breakpoints, producing weird transitions.

### Root cause
v4 keeps all defaults unless explicitly removed. Adding `--breakpoint-3xl`
does not remove `--breakpoint-2xl`.

### NEVER leave unwanted defaults active
```css
@theme {
  --breakpoint-3xl: 120rem;     /* added */
  /* 2xl still active at 96rem */
}
```

### ALWAYS clear what you don't want
```css
@theme {
  --breakpoint-2xl: initial;
  --breakpoint-3xl: 120rem;
}
```

Or reset all and define from scratch :
```css
@theme {
  --breakpoint-*: initial;
  --breakpoint-tablet: 40rem;
  --breakpoint-laptop: 64rem;
  --breakpoint-desktop: 80rem;
}
```

## AP-13 : Trying to Use `{ raw: 'print' }` in v4

### Symptom
v4 project copies a v3 pattern with `screens: { print: { raw: 'print' } }`
into `@theme` as `--breakpoint-print: print;`. Build fails ; `print` is
not a length.

### Root cause
v4 breakpoint variables are CSS lengths. Non-length media queries (print,
hover, prefers-reduced-motion) must be registered as custom variants.

### NEVER
```css
@theme {
  --breakpoint-print: print;     /* invalid : not a length */
}
```

### ALWAYS use `@custom-variant` for non-length media queries
```css
@import "tailwindcss";

@custom-variant print (@media print);
@custom-variant reduced-motion (@media (prefers-reduced-motion: reduce));
@custom-variant any-hover (@media (any-hover: hover));
```

Then use them like `<div class="print:hidden reduced-motion:transition-none">`.

## AP-14 : Container Query In an Unsupported Browser

### Symptom
Layout renders correctly in modern Chrome but breaks in older Safari (15)
or older Firefox. `@container` has no effect.

### Root cause
Container queries require Safari 16.4+, Chrome 105+, Firefox 110+. v4
already enforces Safari 16.4+ so this is mainly a v3 concern (where the
plugin generated `@container` rules but the browser ignored them).

### ALWAYS check the browser baseline before adopting container queries.
If the project targets browsers older than the container-query baseline,
fall back to viewport breakpoints.

## AP-15 : Using `@xs` Expecting v3 Plugin Has It (It Doesn't Have @3xs)

### Symptom
v3 project upgrades to v4 codebase patterns. New components use
`@3xs:flex-col`. In a still-v3 codebase, this produces no CSS.

### Root cause
The v3 plugin shipped 12 sizes (`@xs` through `@7xl`). v4 added 2 more
(`@3xs` at 16rem and `@2xs` at 18rem) for a total of 13.

### NEVER assume the v3 scale matches the v4 scale at the small end.

### ALWAYS in v3 : use `@xs` (20rem / 320px) as the smallest variant, or
define a custom one via `theme.extend.containers`.

```js
// v3 with plugin
module.exports = {
  theme: {
    extend: {
      containers: {
        '3xs': '16rem',
        '2xs': '18rem',
      },
    },
  },
  plugins: [require('@tailwindcss/container-queries')],
}
```
