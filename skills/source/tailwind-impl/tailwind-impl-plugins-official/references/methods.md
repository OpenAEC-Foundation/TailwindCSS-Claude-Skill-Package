# tailwind-impl-plugins-official : Methods Reference

Complete public API for each of the four official Tailwind plugins,
for both v3 (JS-config) and v4 (CSS-first).

## Plugin Install Matrix

| Plugin | npm package | v3 entry | v4 entry |
|--------|-------------|----------|----------|
| Typography | `@tailwindcss/typography` | `plugins: [require(...)]` | `@plugin "..."` |
| Forms | `@tailwindcss/forms` | `plugins: [require(...)]` | `@plugin "..."` |
| Container Queries | `@tailwindcss/container-queries` | `plugins: [require(...)]` | (built-in, do not install) |
| Aspect Ratio | `@tailwindcss/aspect-ratio` | `plugins: [require(...)]` + `corePlugins.aspectRatio: false` | (deprecated, do not install) |

Install commands :

```bash
npm install -D @tailwindcss/typography
npm install -D @tailwindcss/forms
npm install -D @tailwindcss/container-queries     # v3 only
npm install -D @tailwindcss/aspect-ratio          # v3 legacy only
```

## Section 1 : @tailwindcss/typography (prose)

### 1.1 Class anatomy

```
prose                       base typography
prose-{size}                size modifier (sm, base, lg, xl, 2xl)
prose-{color}               color theme (gray, slate, zinc, neutral, stone)
prose-invert                dark mode color inversion
prose-{element}:{utility}   element-level override
not-prose                   escape descendant from typography
max-w-none                  remove 65ch max-width
```

### 1.2 Size scale

| Class | Body font-size | Body line-height | Use case |
|-------|----------------|------------------|----------|
| `prose-sm` | 0.875rem (14px) | 1.7142857 | Sidebars |
| `prose-base` | 1rem (16px) | 1.75 | Default body |
| `prose-lg` | 1.125rem (18px) | 1.7777777 | Articles |
| `prose-xl` | 1.25rem (20px) | 1.8 | Marketing |
| `prose-2xl` | 1.5rem (24px) | 1.6666666 | Hero/lede |

### 1.3 Color themes

| Class | Headings | Body text | Links |
|-------|----------|-----------|-------|
| `prose-gray` (default) | gray-900 | gray-700 | gray-900 |
| `prose-slate` | slate-900 | slate-700 | slate-900 |
| `prose-zinc` | zinc-900 | zinc-700 | zinc-900 |
| `prose-neutral` | neutral-900 | neutral-700 | neutral-900 |
| `prose-stone` | stone-900 | stone-700 | stone-900 |
| `prose-invert` | inverted (for dark mode) | inverted | inverted |

### 1.4 Element-level modifier full list

Each accepts ANY utility class. Syntax `prose-{element}:{utility}` :

`prose-headings` (h1-h4, th), `prose-h1`, `prose-h2`, `prose-h3`,
`prose-h4`, `prose-p`, `prose-a`, `prose-strong`, `prose-em`,
`prose-s`, `prose-blockquote`, `prose-figure`, `prose-figcaption`,
`prose-img`, `prose-video`, `prose-kbd`, `prose-code`, `prose-pre`,
`prose-ol`, `prose-ul`, `prose-li`, `prose-table`, `prose-thead`,
`prose-tr`, `prose-th`, `prose-td`, `prose-hr`, `prose-lead`.

### 1.5 v3 customisation via theme.typography

```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      typography: (theme) => ({
        DEFAULT: {
          css: {
            "--tw-prose-body": theme("colors.zinc.700"),
            "--tw-prose-headings": theme("colors.zinc.900"),
            "--tw-prose-links": theme("colors.emerald.600"),
            "--tw-prose-bold": theme("colors.zinc.900"),
            "--tw-prose-code": theme("colors.rose.700"),
            "--tw-prose-quotes": theme("colors.zinc.900"),
            a: { textDecoration: "none", fontWeight: "600" },
            "h1, h2, h3, h4": { fontFamily: theme("fontFamily.display") },
          },
        },
        lg: { css: { "--tw-prose-body": theme("colors.zinc.800") } },
        invert: { css: { "--tw-prose-body": theme("colors.zinc.300") } },
      }),
    },
  },
  plugins: [require("@tailwindcss/typography")],
};
```

### 1.6 v4 customisation via CSS variables

```css
@import "tailwindcss";
@plugin "@tailwindcss/typography";

@theme {
  --color-prose-body: theme(--color-zinc-700);
  --color-prose-headings: theme(--color-zinc-900);
  --color-prose-links: theme(--color-emerald-600);
  --color-prose-bold: theme(--color-zinc-900);
  --color-prose-code: theme(--color-rose-700);
}
```

### 1.7 Prose CSS variables (full list)

`--tw-prose-body`, `--tw-prose-headings`, `--tw-prose-lead`,
`--tw-prose-links`, `--tw-prose-bold`, `--tw-prose-counters`,
`--tw-prose-bullets`, `--tw-prose-hr`, `--tw-prose-quotes`,
`--tw-prose-quote-borders`, `--tw-prose-captions`, `--tw-prose-code`,
`--tw-prose-pre-code`, `--tw-prose-pre-bg`, `--tw-prose-th-borders`,
`--tw-prose-td-borders`.

Each has an `invert` counterpart : `--tw-prose-invert-body`, etc.,
used when `prose-invert` is applied.

## Section 2 : @tailwindcss/forms

### 2.1 Strategy comparison

| Aspect | `base` | `class` |
|--------|--------|---------|
| Default | yes | no |
| Auto-styles every `<input>` | yes | no |
| Adds `form-*` utility classes | no | yes |
| Side effects on third-party CSS | possible | none |
| Best for | greenfield apps | mixed-framework apps |

### 2.2 v3 strategy config

```js
require("@tailwindcss/forms")({ strategy: "base" })   // default
require("@tailwindcss/forms")({ strategy: "class" })
```

### 2.3 v4 strategy config

```css
@plugin "@tailwindcss/forms";                          /* default base */

@plugin "@tailwindcss/forms" {
  strategy: "class";
}
```

### 2.4 Form classes (strategy: class)

| Class | Applies to |
|-------|-----------|
| `form-input` | `<input type="text" | email | password | url | number | search | tel | date | time | datetime-local | month | week | color>` |
| `form-textarea` | `<textarea>` |
| `form-select` | `<select>` |
| `form-multiselect` | `<select multiple>` |
| `form-checkbox` | `<input type="checkbox">` |
| `form-radio` | `<input type="radio">` |

### 2.5 What the reset normalises

- Removes browser-default appearance (`appearance: none`).
- Sets consistent padding, border, font-size across all input types.
- Adds custom checkmark / radio dot via SVG background.
- Adds focus ring (`outline` + `box-shadow`) using
  `theme.colors.blue.600` by default.
- Sets `cursor: pointer` on select/checkbox/radio.

### 2.6 Customising the focus color (v3)

```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        blue: { 600: "#0070f3" },   // replaces forms-default focus
      },
    },
  },
};
```

### 2.7 Disabling reset for specific inputs

Strategy `class` is the answer. Or scope the reset by selector :

```css
/* Manual override for one input type */
.no-form-reset[type="text"] {
  background-image: none;
  appearance: auto;
}
```

## Section 3 : @tailwindcss/container-queries (v3 only)

### 3.1 Anatomy

```
@container                  declare container
@container/{name}           named container
@{size}:{utility}           variant on default container
@{size}/{name}:{utility}    variant on named container
@[{custom-size}]:{utility}  arbitrary container size
```

### 3.2 Default size scale

| Variant | Min container width |
|---------|--------------------|
| `@xs` | 20rem (320px) |
| `@sm` | 24rem (384px) |
| `@md` | 28rem (448px) |
| `@lg` | 32rem (512px) |
| `@xl` | 36rem (576px) |
| `@2xl` | 42rem (672px) |
| `@3xl` | 48rem (768px) |
| `@4xl` | 56rem (896px) |
| `@5xl` | 64rem (1024px) |
| `@6xl` | 72rem (1152px) |
| `@7xl` | 80rem (1280px) |

### 3.3 Configuring custom container sizes (v3)

```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      containers: {
        "2xs": "16rem",
        "8xl": "90rem",
      },
    },
  },
  plugins: [require("@tailwindcss/container-queries")],
};
```

### 3.4 Native v4 equivalent

```css
@import "tailwindcss";

@theme {
  --container-2xs: 16rem;
  --container-8xl: 90rem;
}
```

Then `@2xs:hidden`, `@8xl:grid-cols-12` etc. work natively.

## Section 4 : @tailwindcss/aspect-ratio (v3 legacy only)

### 4.1 Classes

`aspect-w-1`, `aspect-w-2`, ..., `aspect-w-16` paired with
`aspect-h-1` to `aspect-h-16`. `aspect-none` removes the ratio.

```html
<div class="aspect-w-16 aspect-h-9">
  <iframe src="..."></iframe>
</div>
```

### 4.2 Disabling the core plugin (v3 mandatory)

If you use this plugin AND the native `aspect-*` utility, they
collide. Disable the core utility :

```js
// tailwind.config.js
module.exports = {
  corePlugins: { aspectRatio: false },
  plugins: [require("@tailwindcss/aspect-ratio")],
};
```

### 4.3 Native modern utilities (v3 default + v4)

```html
<iframe class="aspect-video w-full"></iframe>          <!-- 16:9 -->
<img    class="aspect-square w-32" />                  <!-- 1:1 -->
<div    class="aspect-[4/3] w-full"></div>             <!-- arbitrary -->
<div    class="aspect-[1.85/1] w-full"></div>          <!-- decimal allowed -->
```

ALWAYS prefer the native utility unless you support Safari < 14.1
without polyfill.

## Section 5 : Loading Order

When multiple plugins are loaded :

```js
// v3
plugins: [
  require("@tailwindcss/typography"),
  require("@tailwindcss/forms"),
  require("@tailwindcss/container-queries"),
]
```

```css
/* v4 */
@plugin "@tailwindcss/typography";
@plugin "@tailwindcss/forms";
```

Order does NOT affect correctness (no shared classes across these
plugins). Order DOES affect output ordering in the compiled CSS file
which is cosmetic only.

## Section 6 : Verifying a Plugin Loaded

Quick smoke test :

```bash
# After build, grep for plugin-emitted classes
grep -c "prose" dist/output.css        # should be > 0 if typography loaded
grep -c "form-input" dist/output.css    # should be > 0 if forms class strategy
grep -c "@container" dist/output.css    # should be > 0 if container-queries v3
```

If a class you use is missing from output, the plugin is either not
loaded OR the file containing the class is not in the content scan
scope (v3) / `@source` scope (v4). See `tailwind-errors-build`.
