# Anti-Patterns : Tailwind Core Architecture

Architectural anti-patterns mined from official docs and the Tailwind GitHub issue tracker. Each entry follows : symptom, root cause, fix.

## 1. Dynamic class names via template literals

### Symptom
A component reads a prop like `color="red"` and assembles its class string :

```tsx
<div class={`bg-${color}-500 text-white`}>...</div>
```

In dev (HMR re-scans), the class might appear. In production, the element has the class attribute but no CSS exists. The DOM is correctly tagged; the stylesheet is empty for that token.

### Root cause
Tailwind treats source files as **plain text** and looks for literal complete tokens. The string `bg-${color}-500` is never `bg-red-500` in the file; it is the literal text `bg-${color}-500`. The scanner cannot resolve the runtime value and generates nothing.

### Fix (ranked)
1. **Static map** (preferred) :
   ```tsx
   const bg = { red: 'bg-red-500', blue: 'bg-blue-500', green: 'bg-green-500' }[color]
   <div class={`${bg} text-white`}>...</div>
   ```
2. **v4 `@source inline()` with brace expansion** (when the value space is truly runtime) :
   ```css
   @source inline("{hover:,}bg-{red,blue,green}-{50,{100..900..100},950}");
   ```
3. **v3 `safelist`** :
   ```js
   safelist: [
     { pattern: /bg-(red|blue|green)-(50|100|500|900)/, variants: ['hover'] }
   ]
   ```
4. **Inline style** (only when the value is genuinely outside the design system) :
   ```tsx
   <div style={{ backgroundColor: color }}>...</div>
   ```

See [tailwind-errors-dynamic-classes] for the full four-tier treatment and `tailwindlabs/tailwindcss` issue 18136.

## 2. Premature `@apply` extraction

### Symptom
A developer sees `class="px-4 py-2 bg-blue-600 text-white rounded-md text-sm font-medium hover:bg-blue-700"` repeated twice and reaches for :

```css
@layer components {
  .btn-primary {
    @apply px-4 py-2 bg-blue-600 text-white rounded-md text-sm font-medium;
  }
  .btn-primary:hover {
    @apply bg-blue-700;
  }
}
```

Later they need a one-off larger variant and write `class="btn-primary text-base"`. The override fails or partially fails depending on layer order.

### Root cause
- `@apply` inlines utility declarations into the `components` layer.
- Utilities applied later on the same element live in the `utilities` layer which has higher cascade priority, but **only for properties not already set with equal specificity**.
- Once `.btn-primary` sets `text-sm`, `text-base` placed on the same element loses to it in v3 because the utility-source order made `text-base` emit earlier in the layer.
- Premature extraction freezes a name (`btn-primary`) before its boundary is real.

### Fix (ranked)
1. **Extract a component** in the framework :
   ```tsx
   export function Button({ size = 'sm', children }) {
     const sizeClass = size === 'lg' ? 'px-6 py-3 text-base' : 'px-4 py-2 text-sm'
     return (
       <button className={`${sizeClass} bg-blue-600 text-white rounded-md font-medium hover:bg-blue-700`}>
         {children}
       </button>
     )
   }
   ```
2. **Wait** until the duplication crosses 3 files and a real product name emerges.
3. **v4 `@utility`** (acceptable last resort) instead of `@apply` in `@layer components` : utilities applied on the element properly override component properties without `!`.

NEVER `@apply` first. ALWAYS extract a component first.

## 3. Mixing utility-first with runtime CSS-in-JS

### Symptom
A project starts with styled-components, then adds Tailwind for new features. A button is styled with both :

```tsx
const Wrapper = styled.div`
  padding: 16px;
  background: white;
`

<Wrapper className="rounded-lg shadow-md hover:shadow-lg">...</Wrapper>
```

Cascade ordering becomes unpredictable across server/client and across initial render vs hydration. Bundle ships both engines.

### Root cause
- Runtime CSS-in-JS injects styles at component render time; the order of injection depends on render order.
- Tailwind emits CSS at build time in fixed layer order.
- The two compete for the same properties; specificity-equal rules resolve by source order, which is not stable.

### Fix
ALWAYS choose one styling engine per component tree. NEVER bridge them per element. If a migration is in flight, migrate one route or feature at a time and keep the boundaries clean.

## 4. Storing design decisions as scattered arbitrary values

### Symptom
A codebase uses `bg-[#1da1f2]` in 14 components for the brand colour.

### Root cause
Arbitrary values are an **escape hatch** for one-offs. Using them system-wide :
- Defeats the design-system enforcement that motivates Tailwind in the first place.
- Makes brand-colour changes a 14-file refactor.
- Breaks tooling (eslint-plugin-tailwindcss, prettier-plugin-tailwindcss) that assumes tokens.

### Fix
ALWAYS add a token to the theme :

v4 :
```css
@theme {
  --color-brand: #1da1f2;
}
```

v3 :
```js
theme: { extend: { colors: { brand: '#1da1f2' } } }
```

Then refactor : `bg-brand`, `text-brand`, `border-brand`. One token change updates all 14 places.

## 5. Writing custom CSS outside the layer system

### Symptom
A developer adds custom CSS at the top of `app.css` outside any `@layer` block :

```css
@import "tailwindcss";

.hero-banner {
  font-size: 3rem;
  background: red;
}
```

A utility like `text-xl` applied on the same element fails to override `.hero-banner { font-size: 3rem }`.

### Root cause
CSS outside any `@layer` is **unlayered**. The CSS Cascade Layers spec gives unlayered CSS higher priority than layered CSS regardless of source order, so unlayered rules win against Tailwind utilities.

### Fix
ALWAYS place custom CSS inside the correct layer :

```css
@layer components {
  .hero-banner {
    font-size: 3rem;
    background: red;
  }
}
```

Or, in v4, prefer `@utility` for single-purpose custom utilities so they participate in the variant pipeline.

## 6. Using `sm:` to mean "small screens only"

### Symptom
A developer writes `class="sm:text-center text-left"` expecting the text to centre on mobile only.

### Root cause
Tailwind breakpoints are **mobile-first**. `sm:` means "at the `sm` breakpoint and **above**", which is roughly tablet and up. There is no concept of "small screens only" in this prefix.

### Fix
ALWAYS read variants as "and up". For desktop-first overrides, use `max-` variants :
- `class="text-center sm:text-left"` : centred up to `sm` breakpoint, left from `sm` and up.
- `class="text-left max-sm:text-center"` : reads as "left always, except centred below `sm`".

See [tailwind-syntax-responsive].

## 7. Catch-all glob exclusion in v3 / v4 content config

### Symptom
A Next.js project uses App Router with `app/blog/[...slug]/page.tsx`. Classes inside this file are never emitted.

### Root cause
Glob libraries treat `[...]` as a character class. The bracketed directory segment is silently excluded.

### Fix
Add an explicit `@source` (v4) or `content` entry (v3) with escaped brackets :

v4 :
```css
@source './[[]**[]]/**/*.{js,ts,jsx,tsx,mdx}';
```

v3 :
```js
content: [
  './app/**/*.{js,ts,jsx,tsx,mdx}',
  './[[]**[]]/**/*.{js,ts,jsx,tsx,mdx}',
]
```

See `tailwindlabs/tailwindcss` issue 16287.

## 8. `corePlugins: { float: false }` in a v4 project

### Symptom
A v4 project loads a v3 config via `@config "./tailwind.config.js"` and expects `corePlugins.float: false` to disable `float-*` utilities. The utilities still appear in the bundle.

### Root cause
v4 **removed** `corePlugins`, `safelist`, and `separator` config options. They are silently ignored when present in a loaded JS config.

### Fix
- ALWAYS accept that specific core utilities cannot be disabled in v4.
- Use `@source not` to scope content scanning if the goal is bundle size.
- Use a linter (`eslint-plugin-tailwindcss`) to prevent specific utilities at code-review time.

See LESSONS.md L-003.

## 9. Shipping Tailwind utilities to email HTML

### Symptom
A transactional-email template uses `class="px-4 py-6 bg-white rounded-lg shadow-md"`. The email arrives unstyled in Outlook, partially styled in Gmail.

### Root cause
- Most email clients strip `<style>` tags or refuse to load external stylesheets.
- `@layer` is rejected by Outlook's Word-based renderer.
- The "responsive design" promise of Tailwind relies on `@media` queries which many clients ignore.

### Fix
ALWAYS use a tool like MJML or Foundation for Emails to author email HTML. ALWAYS inline final styles with a CSS-inliner like `juice` or `inline-css`. If Tailwind utilities are the source format, run them through an inliner before sending.

## 10. Mixing v3 and v4 syntax in one project

### Symptom
A project uses `@import "tailwindcss"` (v4) but also has `@tailwind base; @tailwind components; @tailwind utilities;` left over. The build emits duplicate base styles.

### Root cause
The `@tailwind` directives are **removed** in v4. They are not erroneous but are no-ops; the `@import` already loads the same content. Some bundlers warn, others silently merge.

### Fix
ALWAYS remove the three `@tailwind` directives when migrating to v4. Run `npx @tailwindcss/upgrade` which handles this and a long list of other migrations. See [tailwind-impl-migration-v3-v4] and LESSONS.md L-001.

## 11. Expecting `@apply` to work inside Vue / Svelte / CSS-modules `<style>` blocks in v4

### Symptom
In a Vue SFC :

```vue
<style scoped>
h1 {
  @apply text-2xl font-bold;
}
</style>
```

Build fails with "Cannot apply unknown utility class".

### Root cause
v4 compiles scoped stylesheets in isolation. They have no implicit access to the theme context. The utility classes are not visible to `@apply`.

### Fix
Prepend `@reference` :

```vue
<style scoped>
@reference "../app.css";
h1 {
  @apply text-2xl font-bold;
}
</style>
```

`@reference` imports theme variables WITHOUT duplicating CSS in output. See [tailwind-impl-apply-directive] and LESSONS.md L-002.

## 12. Treating Tailwind as a runtime library

### Symptom
A developer expects Tailwind to support theme switching by mutating a JS object at runtime ("I'll change the theme from a dropdown").

### Root cause
Tailwind compiles CSS at build time. The theme is fixed by the time the CSS is in the bundle. Runtime theming is achieved by toggling CSS variables, not by re-running the build.

### Fix
ALWAYS implement runtime theming via the existing CSS-variable layer :

```css
@theme {
  --color-bg: white;
  --color-fg: black;
}

[data-theme="dark"] {
  --color-bg: #0f172a;
  --color-fg: #f1f5f9;
}
```

```tsx
<body data-theme={isDark ? 'dark' : 'light'}>
```

Markup keeps `bg-bg`, `text-fg`; the variables change at runtime. NEVER expect Tailwind to recompile in the browser.

## References

- https://tailwindcss.com/docs/detecting-classes-in-source-files (plain-text scan, source variants)
- https://tailwindcss.com/docs/upgrade-guide (removed options, breaking changes)
- https://tailwindcss.com/docs/functions-and-directives (`@apply`, `@reference`)
- https://tailwindcss.com/docs/styling-with-utility-classes (extraction strategies)
- https://github.com/tailwindlabs/tailwindcss/issues/18136 (dynamic class anti-pattern canonical)
- https://github.com/tailwindlabs/tailwindcss/issues/16287 (catch-all glob escape)
- https://github.com/tailwindlabs/tailwindcss/issues/16346 (@apply in scoped styles, v4)
- LESSONS.md L-001 to L-006

Verified 2026-05-19.
