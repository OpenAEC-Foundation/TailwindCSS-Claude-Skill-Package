# Vooronderzoek : Tailwind CSS

> Phase 2 deep research output for the OpenAEC Tailwind CSS Claude Skill Package.
> Verification date: 2026-05-19. All claims traceable to WebFetched official sources listed in section 18.

---

## 1. Status : Phase 2 Complete

| Item | Value |
|------|-------|
| Methodology phase | 2 of 7 (Deep Research) |
| Scope | Tailwind CSS v3.4.x (JIT, JS-config) AND v4.0.x / v4.3.x (Oxide, CSS-first) |
| Sources WebFetched | 17 (see section 18) |
| New sub-topics discovered | 9 (see section 17) |
| Anti-patterns mined from GitHub issues + docs | 12 (see section 16) |
| Lessons recorded in LESSONS.md | 6 |

### Verification Table

| Topic | v3 source | v4 source | Verified |
|-------|-----------|-----------|----------|
| Architecture and philosophy | v3.tailwindcss.com/docs | tailwindcss.com/docs/utility-first, /styling-with-utility-classes | 2026-05-19 |
| Engine internals | tailwindcss.com/blog/just-in-time | tailwindcss.com/blog/tailwindcss-v4 | 2026-05-19 |
| Configuration | v3.tailwindcss.com/docs/configuration | tailwindcss.com/docs/theme, /functions-and-directives | 2026-05-19 |
| Variants | v3.tailwindcss.com/docs/hover-focus-and-other-states | tailwindcss.com/docs/responsive-design, /dark-mode | 2026-05-19 |
| Container queries | github.com/tailwindlabs/tailwindcss-container-queries | tailwindcss.com/docs/responsive-design | 2026-05-19 |
| Dark mode | v3.tailwindcss.com/docs/dark-mode | tailwindcss.com/docs/dark-mode | 2026-05-19 |
| Plugin API | v3.tailwindcss.com/docs/plugins | tailwindcss.com/docs/functions-and-directives | 2026-05-19 |
| Color system | (legacy) | tailwindcss.com/docs/colors | 2026-05-19 |
| Spacing system | (legacy) | tailwindcss.com/docs/padding | 2026-05-19 |
| Source detection | (content array) | tailwindcss.com/docs/detecting-classes-in-source-files | 2026-05-19 |
| Migration | n/a | tailwindcss.com/docs/upgrade-guide | 2026-05-19 |
| Vite install | (PostCSS only) | tailwindcss.com/docs/installation/using-vite | 2026-05-19 |
| Next.js install | (PostCSS only) | tailwindcss.com/docs/installation/framework-guides/nextjs | 2026-05-19 |
| Typography plugin | github.com/tailwindlabs/tailwindcss-typography | same (v4 via @plugin) | 2026-05-19 |
| Forms plugin | github.com/tailwindlabs/tailwindcss-forms | same (v4 via @plugin) | 2026-05-19 |
| tailwind-merge | github.com/dcastil/tailwind-merge | same | 2026-05-19 |
| Anti-patterns | github.com/tailwindlabs/tailwindcss/issues | same | 2026-05-19 |

---

## 2. Architecture and Design Philosophy

Tailwind CSS is a **utility-first** CSS framework. Instead of writing custom rules in stylesheets, developers compose pre-defined single-purpose classes (`mx-auto`, `flex`, `text-xl`, `bg-sky-500`) directly in markup. The framework is *constraint-based*: every utility maps to a design token (spacing, color, type-scale, breakpoint) defined in the theme, so the markup expresses *intent within a system* rather than ad-hoc CSS.

The official rationale published at [tailwindcss.com/docs/utility-first](https://tailwindcss.com/docs/utility-first) lists four primary benefits:

1. **Productivity** : no class-naming, no file-switching between HTML and CSS, no specificity wars.
2. **Maintainability** : changes to one element cannot leak to others because classes are local.
3. **Scalability** : CSS payload grows logarithmically (not linearly) because utilities are reused; with v4 content-scanning, only utilities actually used appear in the bundle.
4. **Safety** : adding/removing a utility affects only the element where it lives.

Tailwind explicitly differentiates itself from inline `style=""` because utilities support **states** (`hover:`, `focus:`, `disabled:`), **media queries** (`sm:`, `md:`, `dark:`), **container queries** (`@sm:`, `@md:`), and **theme tokens** that inline styles cannot express.

The "utility-soup" critique (long class lists) is answered with three official patterns : (a) **loops** in the templating layer, (b) **components** in the framework layer, (c) **`@layer components` + `@apply`** in CSS. The docs warn that premature component extraction is the bigger mistake : start with utilities, extract only when the same combination appears 3+ times.

### Design tokens as the source of truth
Both v3 and v4 enforce that **all design decisions live in the theme**, not in markup. v3 expressed the theme as a JavaScript object (`tailwind.config.js`) ; v4 expresses it as CSS custom properties inside `@theme { ... }`. The token namespaces are identical in concept (colors, fonts, spacing, breakpoints, radii, shadows) ; only the syntax differs.

---

## 3. Engine Model : JIT (v3) vs Oxide (v4)

### v3 : Just-in-Time (JIT) compiler

Released as the default in Tailwind v3.0, the JIT compiler scans content files at build time and **generates CSS on demand** for the exact utility tokens found. Documented behavior at [tailwindcss.com/blog/just-in-time-the-next-generation-of-tailwind-css](https://tailwindcss.com/blog/just-in-time-the-next-generation-of-tailwind-css) :

- Initial compile ~800ms (vs 3 to 8s with the pre-JIT CLI, 30 to 45s with webpack).
- Incremental rebuilds as fast as **3ms**.
- All variants enabled by default (no `variants: { ... }` config required).
- **Arbitrary values** via square-bracket syntax : `bg-[#1da1f2]`, `w-[calc(100%-2rem)]`, `top-[-113px]`.
- Variant stacking : `sm:hover:active:disabled:opacity-75`.
- Production CSS identical to dev CSS (no separate purge step).

### v4 : Oxide engine (Rust)

Released January 22, 2025. The engine is a ground-up rewrite (parts in Rust) optimised for speed, smaller installs, and modern-CSS native features. Performance numbers from the official [v4 launch post](https://tailwindcss.com/blog/tailwindcss-v4) :

| Build type | v3.4 | v4.0 | Speedup |
|---|---|---|---|
| Full build | 378ms | 100ms | **3.78x** |
| Incremental with CSS changes | 44ms | 5ms | **8.8x** |
| Incremental without CSS changes | 35ms | 192us | **182x** |

Key engine-level changes:

- Uses native CSS **cascade layers** (`@layer base/components/utilities`) rather than synthetic ordering.
- Registers custom properties with `@property` for proper interpolation and animation of utilities (e.g. gradients animate smoothly).
- Uses `color-mix()` to compute opacity-modified colors against CSS variables.
- Uses logical properties (`margin-inline`, `padding-block`) for RTL support without extra config.
- Built-in `@import` bundling (no `postcss-import` required).
- Built-in vendor prefixing (no `autoprefixer` required).

### Browser baseline (v4 only)
Per the [upgrade guide](https://tailwindcss.com/docs/upgrade-guide), v4 requires **Safari 16.4+, Chrome 111+, Firefox 128+** because it depends on `@property` and `color-mix()`. Projects targeting older browsers must remain on v3.4.

---

## 4. Configuration : v3 JS-config vs v4 CSS-first

### v3 : `tailwind.config.js`

```js
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ['./src/**/*.{html,js,ts,jsx,tsx}'],
  darkMode: 'class',                 // 'media' | 'class' | 'selector' | array
  theme: {
    extend: {
      colors: { brand: '#1da1f2' },
      spacing: { '128': '32rem' },
    },
  },
  plugins: [
    require('@tailwindcss/typography'),
    require('@tailwindcss/forms'),
  ],
  prefix: 'tw-',
  important: true,
  separator: ':',
  safelist: ['bg-red-500', { pattern: /bg-(red|green|blue)-(100|500|900)/ }],
  blocklist: ['container'],
  corePlugins: { float: false },
  presets: [require('@acmecorp/tailwind-base')],
  future: { hoverOnlyWhenSupported: true },
}
```

All keys are documented at [v3.tailwindcss.com/docs/configuration](https://v3.tailwindcss.com/docs/configuration).

### v4 : CSS-first `@theme`

The JS config file is **no longer auto-detected**. v4 expresses the theme as CSS variables inside `@theme { ... }`. The complete file replacing the above v3 config looks like :

```css
@import "tailwindcss";

@theme {
  --color-brand: #1da1f2;
  --spacing-128: 32rem;
}

@plugin "@tailwindcss/typography";
@plugin "@tailwindcss/forms";

@source "../node_modules/@my-org/ui-lib";
@source not "../src/legacy";

@custom-variant dark (&:where(.dark, .dark *));
```

Per the v4 upgrade guide, the following v3 options are **removed** in v4 :

- `corePlugins`
- `safelist` (replaced by `@source inline(...)`)
- `separator`
- `resolveConfig` JS export (use `getComputedStyle()` against the CSS variables)

Legacy JS config can be opt-in loaded for back-compat via `@config "./tailwind.config.js";`, but the new APIs (`@utility`, `@variant`, `@custom-variant`, `--value()`, `--modifier()`) live only in CSS.

### Prefix syntax change (breaking)
v3 prefix : `tw-flex tw-bg-red-500 hover:tw-bg-red-600`.
v4 prefix : `tw:flex tw:bg-red-500 tw:hover:bg-red-600` (prefix is treated as a variant at the front).

---

## 5. API Surface

### 5.1 Directives

| Directive | v3 | v4 | Behavior |
|-----------|----|----|----------|
| `@tailwind base/components/utilities` | required (3 directives) | removed (use `@import "tailwindcss";`) | Inject Tailwind layers |
| `@import "tailwindcss";` | n/a | required | Single-line replacement for the 3 `@tailwind` directives. Also inlines `postcss-import`-style imports. |
| `@apply` | yes | yes (with caveats) | Inline utility classes into custom CSS. In v4, `<style>` blocks of Vue/Svelte/CSS-modules need `@reference "../app.css";` first because they have no implicit theme context. |
| `@layer base/components/utilities` | yes | yes | Cascade-layer wrapper for custom CSS. v4 uses native CSS `@layer`. |
| `@config "./tailwind.config.js";` | n/a | yes (compat) | Load a v3-style JS config file in a v4 project. |
| `@theme { --token: value; }` | n/a | yes | Define design tokens as CSS variables that automatically generate utilities. Supports `@theme inline { ... }` and `@theme static { ... }`. |
| `@source "path";` | n/a | yes | Explicitly include source files for class scanning. Variants : `@source "path"`, `@source not "path"`, `@source inline("classnames")`. |
| `@source inline("...")` | n/a | yes | Safelist classes. Supports brace expansion : `inline("{hover:,focus:,}bg-red-{50,{100..900..100},950}")`. |
| `@utility name { ... }` | (use `@layer utilities` + plain class) | yes | First-class custom utility. Functional form : `@utility tab-* { tab-size: --value(--tab-size-*); }`. Components made with `@utility` are properly overridable by utility classes. |
| `@variant name { ... }` | n/a | yes | Apply a Tailwind variant inside custom CSS : `@variant dark { background: black; }`. |
| `@custom-variant name (selector)` | n/a | yes | Define a new variant : `@custom-variant theme-midnight (&:where([data-theme="midnight"] *));`. |
| `@plugin "name";` | n/a (use `plugins: []` in JS config) | yes | Load a JS-based plugin from CSS : `@plugin "@tailwindcss/typography";`. |
| `@reference "path";` | n/a | yes | Import theme variables and custom utilities into a scoped stylesheet (Vue/Svelte `<style>`, CSS modules) **without duplicating CSS** in output. |

### 5.2 Plugin API

The JS plugin API (v3 docs at [v3.tailwindcss.com/docs/plugins](https://v3.tailwindcss.com/docs/plugins)) is **preserved in v4** so existing plugins keep working. The `plugin()` function provides a destructured helper object :

```js
const plugin = require('tailwindcss/plugin')

module.exports = plugin(function({
  addUtilities, addComponents, addBase, addVariant,
  matchUtilities, matchComponents, matchVariant,
  theme, config, corePlugins, e,
}) {
  addUtilities({ '.content-auto': { contentVisibility: 'auto' } })

  matchUtilities(
    { tab: (v) => ({ tabSize: v }) },
    { values: theme('tabSize') }
  )

  addVariant('hocus', ['&:hover', '&:focus'])
  matchVariant('nth', (v) => `&:nth-child(${v})`, { values: { 1: '1' } })
})
```

| Helper | Purpose | Respects `prefix` | Respects `important` |
|--------|---------|-------------------|----------------------|
| `addUtilities` | Static utility classes | yes | yes |
| `addComponents` | Component-layer classes | yes | no (manual) |
| `addBase` | Base resets, `@font-face` | no | no |
| `addVariant(name, selector)` | Static custom variant | n/a | n/a |
| `matchUtilities(map, { values, type, supportsNegativeValues })` | Functional utilities with arbitrary-value support | yes | yes |
| `matchComponents` | Functional component classes | yes | no |
| `matchVariant(name, fn, { values, sort })` | Parameterized variants (`aria-*`, `data-*`, `nth-*`) | n/a | n/a |
| `theme('path.to.key')` | Read theme value | n/a | n/a |
| `config('key')` | Read full config value | n/a | n/a |
| `corePlugins('name')` | Test whether a core plugin is enabled | n/a | n/a |
| `e(str)` | Escape a string for safe use as a CSS class | n/a | n/a |

Plugins can also be authored with `plugin.withOptions(fn, configFn)` to accept user options, and can ship default theme values via a second argument.

In v4, the *native* equivalent of `matchUtilities` is `@utility name-* { property: --value(--namespace-*); }` plus `--modifier()` for the `/` modifier. See the [functions-and-directives docs](https://tailwindcss.com/docs/functions-and-directives) for `--value()`, `--alpha()`, and `--spacing()`.

### 5.3 Theme tokens (v4 namespaces)

| Namespace | Generates utilities | Default examples |
|-----------|---------------------|------------------|
| `--color-*` | `bg-*`, `text-*`, `border-*`, `ring-*`, `fill-*`, `stroke-*`, `accent-*`, `caret-*`, `decoration-*`, `outline-*`, `shadow-*` | `--color-red-500: oklch(0.637 0.237 25.331)` |
| `--font-*` | `font-sans`, `font-serif`, `font-mono`, etc. | `--font-sans: ui-sans-serif, system-ui, sans-serif, ...` |
| `--text-*` | `text-xs`, `text-base`, `text-2xl` | `--text-base: 1rem` |
| `--font-weight-*` | `font-thin` to `font-black` | `--font-weight-bold: 700` |
| `--tracking-*` | `tracking-tight`, `tracking-wide` | `--tracking-tight: -0.025em` |
| `--leading-*` | `leading-tight`, `leading-loose` | `--leading-tight: 1.25` |
| `--breakpoint-*` | `sm:`, `md:`, `lg:`, `xl:`, `2xl:` responsive variants | `--breakpoint-sm: 40rem` |
| `--container-*` | `@sm:`, `@md:` container-query variants + `max-w-md` etc. | `--container-md: 28rem` |
| `--spacing` | All `p-*`, `m-*`, `w-*`, `h-*`, `gap-*` (single base unit, scaled by integer) | `--spacing: 0.25rem` |
| `--radius-*` | `rounded-xs`, `rounded-lg`, etc. | `--radius-lg: 0.5rem` |
| `--shadow-*` | `shadow-xs`, `shadow-md`, `shadow-xl` | |
| `--inset-shadow-*` | `inset-shadow-xs` (new in v4) | |
| `--drop-shadow-*` | `drop-shadow-xs`, `drop-shadow-md` | |
| `--blur-*` | `blur-xs`, `blur-md`, `blur-xl` | |
| `--perspective-*` | `perspective-near`, `perspective-distant` (v4 only) | `--perspective-distant: 1200px` |
| `--aspect-*` | `aspect-video`, `aspect-square` | |
| `--ease-*` | `ease-in`, `ease-out`, `ease-in-out` | `--ease-out: cubic-bezier(0,0,0.2,1)` |
| `--animate-*` | `animate-spin`, `animate-bounce` | Plus colocated `@keyframes` inside `@theme` |

To **disable** a namespace : `--color-*: initial;`. To **reset everything** : `--*: initial;`. To **reference another CSS variable** dynamically : wrap in `@theme inline { ... }`.

---

## 6. Variant System (complete listing)

Documented at [v3.tailwindcss.com/docs/hover-focus-and-other-states](https://v3.tailwindcss.com/docs/hover-focus-and-other-states) for v3 and at [tailwindcss.com/docs/responsive-design](https://tailwindcss.com/docs/responsive-design) and `/dark-mode` for v4.

### Pseudo-class variants (both v3 and v4)
`hover`, `focus`, `focus-within`, `focus-visible`, `active`, `visited`, `target`, `first`, `last`, `only`, `odd`, `even`, `first-of-type`, `last-of-type`, `only-of-type`, `empty`, `disabled`, `enabled`, `checked`, `indeterminate`, `default`, `required`, `valid`, `invalid`, `in-range`, `out-of-range`, `placeholder-shown`, `autofill`, `read-only`, `open`.

### Pseudo-element variants
`before`, `after`, `placeholder`, `file`, `marker`, `selection`, `first-line`, `first-letter`, `backdrop`. v4 adds `details-content` and `popover-open` recognition with `open`.

### Media-query variants
Breakpoints : `sm`, `md`, `lg`, `xl`, `2xl`, plus the `max-*` counterparts (`max-sm`, `max-md`, ...). Arbitrary breakpoints : `min-[400px]:flex`, `max-[600px]:hidden`.

Preferences : `dark`, `light`, `motion-safe`, `motion-reduce`, `contrast-more`, `contrast-less`, `forced-colors`, `portrait`, `landscape`, `print`, `screen`.

Direction : `ltr`, `rtl`.

### Feature-query variants
`supports-[display:grid]`, `supports-[backdrop-filter]`. v4 adds `not-supports-*`.

### Attribute variants
`aria-checked`, `aria-disabled`, `aria-expanded`, `aria-hidden`, `aria-pressed`, `aria-readonly`, `aria-required`, `aria-selected`, plus arbitrary `aria-[...]`.
`data-active`, `data-state-open`, plus arbitrary `data-[...]`.

### Combinator variants
`group`, `group-hover`, `group-focus`, `group-active`, `group-aria-*`, `group-has-*`, `group-data-*`, plus named groups `group/sidebar` then `group-hover/sidebar`.
`peer`, `peer-hover`, `peer-focus`, `peer-checked`, `peer-has-*`, plus named peers.
`*` (direct-children variant).

### v4-only additions
| Variant | Purpose | Example |
|---------|---------|---------|
| `not-*` | Negate any variant | `not-hover:opacity-75`, `not-supports-hanging-punctuation:px-4` |
| `in-*` | Group-like behaviour without needing a `group` class on the ancestor | `in-[role=tabpanel]:hidden` |
| `inert` | Style elements with the `inert` attribute | `inert:opacity-50` |
| `nth-*` parameterised | `nth-[3n+1]:bg-blue-500` | |
| `starting:*` | Hooks into `@starting-style` for CSS-only enter animations | `starting:open:opacity-0` |
| `details-content` | Style `<details>` body separately from `<summary>` | |

### Arbitrary variant syntax (both versions)
`[&:nth-child(3)]:underline`, `[&_p]:text-red-500`, `[@media(width>800px)]:flex`. Stacks like any other variant.

### Stacking-order change in v4 (breaking)
v3 applied stacked variants right-to-left ; v4 applies **left-to-right** to match CSS reading order. The migration cited in the upgrade guide : `first:*:pt-0` becomes `*:first:pt-0`.

---

## 7. Responsive and Container Queries

### Breakpoints (mobile-first, both versions)

| Prefix | min-width | CSS |
|--------|-----------|-----|
| `sm` | 40rem (640px) | `@media (width >= 40rem)` |
| `md` | 48rem (768px) | `@media (width >= 48rem)` |
| `lg` | 64rem (1024px) | `@media (width >= 64rem)` |
| `xl` | 80rem (1280px) | `@media (width >= 80rem)` |
| `2xl` | 96rem (1536px) | `@media (width >= 96rem)` |

The cardinal rule : **unprefixed utilities apply to all screens ; prefixed utilities apply at that breakpoint and up**. `sm:text-center` does NOT mean "small screens only".

### Max-* and arbitrary breakpoints
`max-md:flex`, `min-[400px]:flex`, `max-[600px]:hidden`, range : `md:max-lg:bg-red-500`.

### Container queries

| | v3 | v4 |
|---|----|----|
| Availability | Via `@tailwindcss/container-queries` plugin | **Built-in** (plugin no longer needed) |
| Mark container | `class="@container"` | `class="@container"` |
| Variants | `@xs` (20rem) to `@7xl` (80rem) | Same scale ; uses `--container-*` namespace |
| Named containers | `@container/main` then `@sm/main:...` | Same |
| Max-* | `@max-md:flex-col` | `@max-md:flex-col` |
| Arbitrary sizes | `@[17.5rem]:underline` | `@min-[475px]:flex-row`, `@max-[960px]:hidden` |
| `cqw`/`cqh`/`cqb` units | n/a | Arbitrary-value : `w-[50cqw]` |

---

## 8. Dark Mode Strategies

### v3.4 (selector-based, since v3.4.1)
| Strategy | Config | Selector |
|----------|--------|----------|
| Media query (default) | `darkMode: 'media'` (or omit) | `@media (prefers-color-scheme: dark)` |
| Class | `darkMode: 'class'` | `.dark` on `<html>` |
| Selector (v3.4.1+) | `darkMode: 'selector'` | `.dark` ancestor (replaces `class`) |
| Selector with custom attribute | `darkMode: ['selector', '[data-theme="dark"]']` | `[data-theme="dark"]` |
| Fully custom | `darkMode: ['variant', '&:not(.light *)']` | Any selector |

### v4 (`@custom-variant`-based)
v4 has **no `darkMode` config**. Default is still `prefers-color-scheme`. To opt into class or data toggling, override the `dark` variant in CSS :

```css
@import "tailwindcss";
@custom-variant dark (&:where(.dark, .dark *));
```

Or for data-attribute :
```css
@custom-variant dark (&:where([data-theme=dark], [data-theme=dark] *));
```

Toggle via JS, respecting OS preference :

```js
document.documentElement.classList.toggle(
  "dark",
  localStorage.theme === "dark" ||
    (!("theme" in localStorage) &&
      window.matchMedia("(prefers-color-scheme: dark)").matches)
);
```

---

## 9. Build Integration Patterns (per tool)

### Vite (recommended for v4)
```bash
npm install tailwindcss @tailwindcss/vite
```
```ts
// vite.config.ts
import { defineConfig } from 'vite'
import tailwindcss from '@tailwindcss/vite'
export default defineConfig({ plugins: [tailwindcss()] })
```
```css
/* src/style.css */
@import "tailwindcss";
```

### PostCSS (Next.js, Astro, generic)
```bash
npm install tailwindcss @tailwindcss/postcss postcss
```
```js
// postcss.config.mjs
export default { plugins: { "@tailwindcss/postcss": {} } }
```
No `postcss-import` or `autoprefixer` needed in v4 (both built into the Oxide engine).

### Standalone CLI
```bash
npx @tailwindcss/cli -i input.css -o output.css --watch
```
v3 used `npx tailwindcss` ; v4 uses `npx @tailwindcss/cli`.

### Next.js (App Router)
Use `@tailwindcss/postcss` + `postcss.config.mjs` + `@import "tailwindcss"` in `app/globals.css`. Import once in the root layout. Compatible with both Server Components and Client Components since CSS is statically generated.

### Astro
v3 used the official `@astrojs/tailwind` integration. v4 recommends the Vite plugin directly because Astro uses Vite under the hood. **Caveat** : per [issue 16733](https://github.com/tailwindlabs/tailwindcss/issues/16733) some Astro+v4 combinations have regressed on specific patch versions ; pin a known-good version per release.

### Other frameworks
- **Remix** : PostCSS plugin in `postcss.config.cjs`.
- **Nuxt** : `@nuxtjs/tailwindcss` for v3 ; for v4 use the Vite plugin directly.
- **SvelteKit** : Vite plugin (v4) or PostCSS (v3). Scoped `<style>` blocks need `@reference "../app.css";` before `@apply`.

---

## 10. @apply and @layer Mechanics

`@apply` inlines the exact CSS that a utility class would have produced. It is intended for :
- Overriding third-party library styles that you cannot rewrite.
- Building component classes that bundle many utilities (`.btn`, `.card`).

### v3 behavior
Available globally inside any CSS file processed by Tailwind. Order matters : utilities applied later override earlier ones following normal cascade rules.

### v4 behavior (BREAKING for scoped styles)
`@apply` still exists, but **a scoped stylesheet that is processed independently from `app.css` (Vue `<style>`, Svelte `<style>`, CSS-modules, MDX-CSS) has no implicit access to the theme**. Without context, `@apply text-2xl` fails with "Cannot apply unknown utility class".

The fix is the new `@reference` directive ([docs](https://tailwindcss.com/docs/functions-and-directives)) :

```vue
<style>
  @reference "../../app.css"; /* or "tailwindcss" for defaults only */
  h1 { @apply text-2xl font-bold text-red-500; }
</style>
```

`@reference` imports theme variables **without** duplicating CSS in the output bundle.

### `@layer` ordering (both versions)
Tailwind uses three named layers : `base` (resets, typography defaults), `components` (multi-class component classes), `utilities` (single-purpose classes). v4 uses native CSS `@layer` so cascade ordering is honoured by the browser. Custom CSS outside `@layer` becomes "unlayered" and wins over layered CSS regardless of source order, which is occasionally desirable for true overrides.

### `@utility` vs `@layer utilities` (v4)
`@utility` is **preferred** in v4 because :
1. Components defined as `@utility` are properly overridable by user-applied utility classes (the v3 problem where `card rounded-none` failed to override component-level `rounded-lg`).
2. `@utility` supports functional syntax via `--value()` and `--modifier()`.
3. `@utility` participates in the variant pipeline (responsive, hover, dark, etc.) automatically.

---

## 11. Plugin Authorship Patterns

### Official plugins
| Plugin | Purpose | v3 usage | v4 usage |
|--------|---------|----------|----------|
| `@tailwindcss/typography` | `prose` class for rendered Markdown | `plugins: [require('@tailwindcss/typography')]` | `@plugin "@tailwindcss/typography";` |
| `@tailwindcss/forms` | Form-element reset | Same pattern | Same |
| `@tailwindcss/container-queries` | `@container`/`@md:` variants | Plugin required | **Removed** : built into core v4 |
| `@tailwindcss/aspect-ratio` | `aspect-w-*`/`aspect-h-*` legacy syntax | Plugin required | **Deprecated** : use `aspect-video`, `aspect-square`, `aspect-[16/9]` natively |

### Typography plugin modifiers
- Sizes : `prose-sm` (14px), `prose-base` (default, 16px), `prose-lg`, `prose-xl`, `prose-2xl`.
- Color themes : `prose-gray`, `prose-slate`, `prose-zinc`, `prose-neutral`, `prose-stone`. Dark mode : `prose-invert`.
- Element-level overrides : `prose-headings:`, `prose-a:`, `prose-img:`, `prose-code:`, etc.
- `not-prose` to escape typography styling on subtrees.
- `max-w-none` to override the default 65-character `max-width`.

### Forms plugin
Two strategies : `base` (global element styles only, no classes), `class` (only `form-input`/`form-textarea`/`form-select`/`form-multiselect`/`form-checkbox`/`form-radio` classes ; no global styling).

### Custom plugin skeleton (v3 + v4-compatible)
```js
const plugin = require('tailwindcss/plugin')
module.exports = plugin(
  function({ addUtilities, matchUtilities, theme }) {
    addUtilities({ '.scrollbar-hidden': { '&::-webkit-scrollbar': { display: 'none' } } })
    matchUtilities(
      { 'text-shadow': (value) => ({ textShadow: value }) },
      { values: theme('textShadow') }
    )
  },
  { theme: { textShadow: { sm: '0 1px 2px rgba(0,0,0,.1)' } } }
)
```

### Native v4 equivalent (CSS-first)
```css
@utility scrollbar-hidden {
  &::-webkit-scrollbar { display: none; }
}
@utility text-shadow-* {
  text-shadow: --value(--text-shadow-*, [length]);
}
@theme {
  --text-shadow-sm: 0 1px 2px rgba(0,0,0,.1);
}
```

---

## 12. Migration v3 to v4 (breaking changes)

Run the automated tool first : `npx @tailwindcss/upgrade` (requires Node.js 20+). Always run on a clean branch and review changes. The tool handles dependency updates, JS-config to CSS migration, deprecated-utility renames, and most template rewrites.

### Breaking changes table (curated from the [upgrade guide](https://tailwindcss.com/docs/upgrade-guide))

| Category | v3 | v4 | Notes |
|----------|----|----|-------|
| Package | `tailwindcss` | Add `@tailwindcss/postcss` or `@tailwindcss/vite` | Core still `tailwindcss` for theme/types |
| PostCSS deps | `postcss-import`, `autoprefixer` | Removed | Built into Oxide |
| Import | `@tailwind base; @tailwind components; @tailwind utilities;` | `@import "tailwindcss";` | |
| Config file | `tailwind.config.js` auto-loaded | Not auto-loaded ; use `@config "./..."` for legacy | |
| Opacity utilities | `bg-opacity-50` | `bg-red-500/50` | Removed `bg-opacity-*`, `text-opacity-*`, `border-opacity-*`, `divide-opacity-*`, `ring-opacity-*`, `placeholder-opacity-*` |
| Flexbox | `flex-shrink-*`, `flex-grow-*` | `shrink-*`, `grow-*` | |
| Text overflow | `overflow-ellipsis` | `text-ellipsis` | |
| `bg-gradient-*` | `bg-gradient-to-r` | `bg-linear-to-r` | Renamed for consistency with conic/radial |
| Shadow scale | `shadow-sm` (small), `shadow` (default), `shadow-md`, `shadow-lg`, `shadow-xl` | `shadow-xs`, `shadow-sm`, `shadow-md`, `shadow-lg`, `shadow-xl` | All sizes shifted by one |
| Blur scale | `blur-sm`, `blur` | `blur-xs`, `blur-sm` | Same shift |
| Radius scale | `rounded-sm`, `rounded` | `rounded-xs`, `rounded-sm` | Same shift |
| Drop-shadow | `drop-shadow-sm`, `drop-shadow` | `drop-shadow-xs`, `drop-shadow-sm` | Same shift |
| Backdrop blur | `backdrop-blur-sm`, `backdrop-blur` | `backdrop-blur-xs`, `backdrop-blur-sm` | Same shift |
| Outline | `outline outline-2` | `outline-2` (width 1px default) | `outline-none` becomes `outline-hidden` |
| Ring | `ring` = 3px, default colour blue-500 | `ring` = 1px, default colour `currentColor` | To preserve v3 : `--default-ring-width: 3px; --default-ring-color: var(--color-blue-500);` |
| Border colour | Default `gray-200` | Default `currentColor` | Manual restore via `@layer base` |
| `space-y-*` selector | `> :not([hidden]) ~ :not([hidden])` | `> :not(:last-child)` margin-bottom | Performance fix ; recommend `flex flex-col gap-*` instead |
| `divide-*` selector | Same complex selector | `> :not(:last-child)` border-bottom | Same fix |
| Important modifier | `!flex` | `flex!` | Position moved to end |
| CSS-var arbitrary | `bg-[--brand]` | `bg-(--brand)` | Parens, not brackets |
| Arbitrary commas | `grid-cols-[max-content,auto]` | `grid-cols-[max-content_auto]` | Underscores for spaces |
| Variant stacking | Right-to-left | Left-to-right | `first:*:pt-0` becomes `*:first:pt-0` |
| Transform reset | `transform-none` | `scale-none`, `rotate-none` etc. | Individual properties |
| Hover variant | Always applies | `@media (hover: hover)` gated | Override with `@custom-variant hover (&:hover);` |
| Preflight placeholder | `gray-400` | `currentColor` at 50% | Restore via `@layer base` |
| Preflight buttons | `cursor: pointer` | `cursor: default` | Restore via `@layer base` |
| `theme()` function | `theme(colors.red.500)` | Use `var(--color-red-500)` (or `theme(--color-red-500)`) | |
| Dark mode config | `darkMode: 'class'` | `@custom-variant dark (&:where(.dark, .dark *));` | |
| `corePlugins` | Disable utilities | Removed | No alternative ; use `@source not "..."` selectively |
| `safelist` | Array/regex in config | `@source inline("classnames")` | |
| `resolveConfig` JS | `import resolveConfig from 'tailwindcss/resolveConfig'` | Removed | Use `getComputedStyle(document.documentElement).getPropertyValue('--color-red-500')` |
| Sass/Less/Stylus | Worked in dev | **Not supported** | Tailwind is the preprocessor |
| Prefix syntax | `tw-flex` | `tw:flex` | Variant-style at front |

---

## 13. JIT and Arbitrary Values

Arbitrary values let any utility accept a one-off value via square-bracket syntax (or parentheses for CSS variables in v4) :

```html
<!-- Arbitrary colours, sizes, calc expressions -->
<div class="bg-[#1da1f2] w-[calc(100%-2rem)] top-[-113px]"></div>
<!-- Type hints for ambiguous values -->
<div class="bg-[length:200px_100px] bg-[url('/img/hero.png')]"></div>
<!-- Arbitrary properties (set any CSS property) -->
<div class="[mask-type:luminance]"></div>
<!-- CSS variables (v4 syntax) -->
<div class="bg-(--brand-color) text-(--brand-fg)"></div>
<!-- Arbitrary variants -->
<ul class="[&_p]:mt-4 [&>[data-active]+span]:text-blue-600"></ul>
```

### Type hints when needed
When a value is ambiguous (e.g. `bg-[200px_100px]` could be `background-position` or `background-size`), prefix with a CSS data-type : `bg-[length:200px_100px]`, `text-[color:var(--brand)]`.

### v4 dynamic utilities without arbitrary values
v4 generates spacing utilities for **any integer** automatically. `mt-17`, `w-29`, `gap-43` all work without arbitrary syntax because they desugar to `calc(var(--spacing) * N)`. Grid columns also accept any integer : `grid-cols-15`.

---

## 14. tailwind-merge Interop

`tailwind-merge` ([github.com/dcastil/tailwind-merge](https://github.com/dcastil/tailwind-merge)) solves the **utility-conflict problem** : when component composition produces `"px-2 py-1 bg-red"` + `"p-3 bg-[#B91C1C]"`, the result should be `"hover:bg-dark-red p-3 bg-[#B91C1C]"` (later wins on overlapping properties).

### Why it matters
Tailwind generates utilities in a fixed source order, so the **last class in the HTML attribute does NOT always win**. Two competing utilities both targeting `padding` resolve by CSS source order, not attribute order. `tailwind-merge` re-orders the string so that, after deduplication, only the intended utility survives.

### API
```ts
import { twMerge } from 'tailwind-merge'
twMerge('px-2 py-1 bg-red hover:bg-dark-red', 'p-3 bg-[#B91C1C]')
// => 'hover:bg-dark-red p-3 bg-[#B91C1C]'
```

### Ecosystem pairing
The shadcn/ui canonical pattern (`cn` helper) combines `clsx` (conditional class assembly) with `twMerge` (deduplication) :

```ts
import { clsx, type ClassValue } from 'clsx'
import { twMerge } from 'tailwind-merge'
export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

`cva` (class-variance-authority) goes one step further with typed variant maps for component APIs.

### Version compatibility
- `tailwind-merge` v2.x for Tailwind v3.x.
- `tailwind-merge` v3.x for Tailwind v4.x (v4.0 through v4.3 supported as of May 2026).

### Customisation
`extendTailwindMerge({ extend: { classGroups: { 'shadow': ['my-custom-shadow'] } } })` to register custom utilities. `createTailwindMerge(getDefaultConfig)` for a fresh instance with full custom config.

### Performance
`twMerge` is O(n) in the number of classes. The library caches normalised group lookups per call site, so reuse in render-heavy components is cheap.

---

## 15. Production Build Concerns

Both v3 and v4 strip unused utilities through content scanning. The scanning rules are :

- v3 uses the `content: [...]` glob in `tailwind.config.js`. A missing or wrong path means utilities never appear in the production bundle.
- v4 auto-detects everything except `.gitignore`'d paths, `node_modules`, binary files, and CSS files. To include `node_modules` packages (component libraries), use `@source "../node_modules/@my-org/ui-lib";`.
- Both versions treat source files as **plain text tokens**. A class must appear as a *literal complete token* in some scanned file to be generated. String concatenation, template literals, runtime composition all break detection.
- v4 introduces `@source inline("class-list")` for explicit safelisting and supports brace expansion for ranges (`bg-red-{50,{100..900..100},950}`).
- v4 introduces `@source not "path"` for explicit exclusion.

Common failure mode (decimal classes, per [issue 9401](https://github.com/tailwindlabs/tailwindcss/issues/9401)) : in v2 some bundlers treated `py-2.5` as `py-2` `.5` two tokens, breaking detection. This is fixed in v3 JIT and v4, but third-party scanners (Turbopack, certain PostCSS chains) sometimes regress.

---

## 16. Anti-Patterns (mined from official docs + GitHub issues)

| # | Anti-pattern | Why it fails | Fix | Source |
|---|--------------|--------------|-----|--------|
| 1 | Dynamic class string concatenation : `` `bg-${color}-500` `` | Tailwind scans source files as **plain text tokens**. `bg-${color}-500` is never a literal token, so the class is never generated. | Map props to complete static class strings : `const C = { red: 'bg-red-500', blue: 'bg-blue-500' }[color]`. | [/docs/detecting-classes](https://tailwindcss.com/docs/detecting-classes-in-source-files), [issue 18136](https://github.com/tailwindlabs/tailwindcss/issues/18136) |
| 2 | Server-injected class strings from user data not present in any source file | Same root cause : not statically detectable. The DOM has the class, but no CSS rule exists. | Use `@source inline("classnames")` to safelist, OR pre-compute server-side with the same Tailwind build, OR use inline `style=""` for fully dynamic values. | [issue 18136](https://github.com/tailwindlabs/tailwindcss/issues/18136) |
| 3 | Putting `@apply text-2xl` in a Vue `<style>` block without `@reference` in v4 | Scoped stylesheets are compiled in isolation and have no theme context. Error : "Cannot apply unknown utility class". | Prepend `@reference "../app.css";` (or `@reference "tailwindcss";`) inside the `<style>` block. | [/docs/functions-and-directives](https://tailwindcss.com/docs/functions-and-directives), [issue 16346](https://github.com/tailwindlabs/tailwindcss/issues/16346) |
| 4 | Glob `./**/*.{js,ts,jsx,tsx,mdx}` failing to match Next.js catch-all routes `[...slug]` | Glob engine interprets `[...]` as character-class. Files in bracketed directories are silently excluded. | Add explicit secondary source : `@source './[[]**[]]/*.{js,ts,jsx,tsx,mdx}';`. | [issue 16287](https://github.com/tailwindlabs/tailwindcss/issues/16287) |
| 5 | Using `sm:` to mean "small screens only" | `sm:` means "at sm breakpoint and above" (mobile-first). It does NOT cap the upper bound. | Default style is for mobile, override at `sm:` and above. Or use `max-sm:` to target below sm. | [/docs/responsive-design](https://tailwindcss.com/docs/responsive-design) |
| 6 | Composing utilities via string-template + expecting later wins : `<div class={`p-2 ${override}`} />` where `override="p-4"` | Wins resolved by CSS source order, not HTML attribute order. If `p-2` was emitted later it overrides `p-4`. | Use `twMerge('p-2 p-4')` to deduplicate deterministically, returning `'p-4'`. | [github.com/dcastil/tailwind-merge](https://github.com/dcastil/tailwind-merge) |
| 7 | Configuring `prefix: 'tw-'` in v4 and writing `@apply tw-bg-slate-100` | v4 prefix is a *variant prefix* (`tw:bg-slate-100`), not a class prefix. The old form throws "Cannot apply unknown utility class : tw-bg-slate-100". | Migrate to v4 prefix syntax (`tw:`) and update all `@apply` directives. | [issue 16346](https://github.com/tailwindlabs/tailwindcss/issues/16346) |
| 8 | Relying on `corePlugins: { float: false }` in v4 | `corePlugins` is **removed** in v4. The build will not error, but the config is silently ignored. | Use `@source not "..."` to scope content scanning, accept the utility exists, or write a lint rule. | [/docs/upgrade-guide](https://tailwindcss.com/docs/upgrade-guide) |
| 9 | Using `theme(colors.red.500)` in v4 CSS | `theme(...)` with dot-notation is deprecated. v4 expects CSS-variable paths : `theme(--color-red-500)` or just `var(--color-red-500)`. | Migrate dot-notation calls to CSS variables. | [/docs/upgrade-guide](https://tailwindcss.com/docs/upgrade-guide) |
| 10 | Using `space-y-4` with elements that have margin overrides | v3 selector `> :not([hidden]) ~ :not([hidden])` is fragile. v4 selector changed (using `last-child`), which silently changes layout for some patterns. | Prefer `flex flex-col gap-4` (or `grid gap-4`). `space-*` and `divide-*` exist for legacy reasons. | [/docs/upgrade-guide sections 7 and 8](https://tailwindcss.com/docs/upgrade-guide) |
| 11 | Stacking v3-style variants in v4 : `first:*:pt-0` | v4 changed stacking to left-to-right. The selector now compiles to a different element. | Reverse the order : `*:first:pt-0`. The upgrade tool handles common cases. | [/docs/upgrade-guide section 14](https://tailwindcss.com/docs/upgrade-guide) |
| 12 | Expecting `transition-colors` not to animate `outline-color` | v4 added `outline-color` to the default colour-transition set. Hover styles that set `outline-2` now animate the outline colour from `currentColor` to the implicit transparent. | Set `outline-cyan-500 transition hover:outline-2` (declare the colour before the hover) to get a stable transition. | [/docs/upgrade-guide section 18](https://tailwindcss.com/docs/upgrade-guide) |

Bonus runtime issue (Turbopack-specific, [issue 19825](https://github.com/tailwindlabs/tailwindcss/issues/19825)) : `aspect-[12/5]`, `z-[100]`, `h-[80vh]` arbitrary values can be missed by Next.js 16.1 Turbopack incremental scanner. Workaround : use inline `style` for layout-critical arbitrary values until the upstream fix lands.

---

## 17. Newly Discovered Sub-Topics (not in raw masterplan)

Items uncovered by Phase 2 research that warrant their own skill or expansion of an existing topic :

1. **`@reference` directive for scoped stylesheets** (Vue/Svelte/CSS-modules in v4). Was not mentioned in raw masterplan. Belongs in `impl-apply-directive` AND `errors-v4-migration`.
2. **`--value()`, `--modifier()`, `--alpha()`, `--spacing()` CSS functions for v4 functional utilities**. Belongs in `impl-config-v4` or a new `syntax-functional-utilities` skill.
3. **`@source inline()` safelisting with brace expansion** (`@source inline("{hover:,}bg-red-{50,{100..900..100},950}")`). Belongs in `errors-purge-issues`.
4. **3D transform utilities** (`rotate-x-*`, `rotate-y-*`, `rotate-z-*`, `translate-z-*`, `perspective-*`, `transform-3d`) are entirely new in v4 and merit a dedicated `syntax-3d-transforms` skill.
5. **Expanded gradient APIs** (`bg-linear-45`, `bg-linear-to-r/srgb`, `bg-linear-to-r/oklch`, `bg-conic-*`, `bg-radial-*`, interpolation modifiers). New in v4 ; consider `syntax-gradients` skill.
6. **`@starting-style` + `starting:` variant + `transition-discrete`** for CSS-only enter/exit animations on popovers/dialogs. New in v4 ; consider `syntax-starting-style` skill.
7. **`inset-shadow-*` and `inset-ring-*`** (layered inset shadows, up to 4 layers per element). v4-only ; covered briefly in `core-design-system`.
8. **`field-sizing-content`** for auto-resizing textareas, **`color-scheme`** utilities, **`font-stretch`** for variable fonts. Worth a short v4-only `syntax-modern-utilities` reference.
9. **`@theme inline` vs `@theme static`** modifiers control how variable references are emitted. Belongs in `impl-config-v4`.

---

## 18. SOURCES Update (verification log)

All URLs in the table below were WebFetched on 2026-05-19 and used to populate this document. SOURCES.md is updated in tandem.

| URL | Type | Status |
|-----|------|--------|
| https://tailwindcss.com/blog/tailwindcss-v4 | Official blog (v4 launch) | Verified 2026-05-19 |
| https://tailwindcss.com/docs/installation/using-vite | Official docs (Vite install v4) | Verified 2026-05-19 |
| https://tailwindcss.com/docs/upgrade-guide | Official migration guide | Verified 2026-05-19 |
| https://tailwindcss.com/docs/functions-and-directives | Official docs (v4 directives) | Verified 2026-05-19 |
| https://tailwindcss.com/docs/theme | Official docs (v4 theme namespaces) | Verified 2026-05-19 |
| https://tailwindcss.com/docs/adding-custom-styles | Official docs (@utility, @layer) | Verified 2026-05-19 |
| https://tailwindcss.com/docs/dark-mode | Official docs (v4 dark mode) | Verified 2026-05-19 |
| https://tailwindcss.com/docs/responsive-design | Official docs (v4 responsive + container queries) | Verified 2026-05-19 |
| https://tailwindcss.com/docs/colors | Official docs (v4 colour system) | Verified 2026-05-19 |
| https://tailwindcss.com/docs/detecting-classes-in-source-files | Official docs (source detection) | Verified 2026-05-19 |
| https://tailwindcss.com/docs/padding | Official docs (v4 spacing) | Verified 2026-05-19 |
| https://tailwindcss.com/docs/utility-first | Official docs (philosophy) | Verified 2026-05-19 |
| https://tailwindcss.com/docs/styling-with-utility-classes | Official docs (advanced syntax + anti-patterns) | Verified 2026-05-19 |
| https://tailwindcss.com/docs/installation/framework-guides/nextjs | Official docs (Next.js install) | Verified 2026-05-19 |
| https://tailwindcss.com/blog/just-in-time-the-next-generation-of-tailwind-css | Official blog (JIT origin) | Verified 2026-05-19 |
| https://v3.tailwindcss.com/docs/installation | v3 docs (install) | Verified 2026-05-19 |
| https://v3.tailwindcss.com/docs/configuration | v3 docs (config API) | Verified 2026-05-19 |
| https://v3.tailwindcss.com/docs/hover-focus-and-other-states | v3 docs (variants) | Verified 2026-05-19 |
| https://v3.tailwindcss.com/docs/plugins | v3 docs (plugin API) | Verified 2026-05-19 |
| https://v3.tailwindcss.com/docs/dark-mode | v3 docs (dark mode) | Verified 2026-05-19 |
| https://github.com/tailwindlabs/tailwindcss-typography | Plugin repo | Verified 2026-05-19 |
| https://github.com/tailwindlabs/tailwindcss-forms | Plugin repo | Verified 2026-05-19 |
| https://github.com/tailwindlabs/tailwindcss-container-queries | Plugin repo (v3-only) | Verified 2026-05-19 |
| https://github.com/dcastil/tailwind-merge | Companion library | Verified 2026-05-19 |
| https://github.com/tailwindlabs/tailwindcss/issues/18136 | Issue tracker (dynamic class anti-pattern) | Verified 2026-05-19 |
| https://github.com/tailwindlabs/tailwindcss/issues/16346 | Issue tracker (@apply breaks in v4) | Verified 2026-05-19 |
| https://github.com/tailwindlabs/tailwindcss/issues/16287 | Issue tracker (catch-all glob escape) | Verified 2026-05-19 |
| https://github.com/tailwindlabs/tailwindcss/issues/16733 | Issue tracker (v4.0.8 Astro break) | Verified 2026-05-19 |
| https://github.com/tailwindlabs/tailwindcss/issues/19825 | Issue tracker (Turbopack arbitrary value miss) | Verified 2026-05-19 |
| https://github.com/tailwindlabs/tailwindcss/issues/9401 | Issue tracker (decimal class purge) | Verified 2026-05-19 |
