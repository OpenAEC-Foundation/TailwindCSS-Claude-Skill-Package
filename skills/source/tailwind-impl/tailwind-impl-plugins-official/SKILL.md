---
name: tailwind-impl-plugins-official
description: >
  Use when installing, configuring, or customising any of the four
  official Tailwind plugins: @tailwindcss/typography (prose classes for
  rendered Markdown), @tailwindcss/forms (form-input/select/checkbox
  base reset), @tailwindcss/container-queries (@container variants),
  @tailwindcss/aspect-ratio (legacy aspect-w-*/aspect-h-* syntax).
  Prevents the v3-syntax-in-v4 trap (require() arrays do not work in
  v4, you need @plugin), the missing-plugin trap (forgetting to add
  container-queries to v3 silently emits zero @-variant classes), the
  unneeded-plugin-in-v4 trap (installing container-queries or
  aspect-ratio in v4 wastes bundle size and ships duplicated utilities),
  the forms-strategy mismatch (strategy base auto-styles every form
  element globally, strategy class only adds form-* classes), the
  not-prose escape miss (Markdown rendered inside a button or card
  inherits prose typography unintentionally), and the typography
  theme-customisation confusion (v3 uses theme.typography in JS, v4
  uses @plugin block syntax).
  Covers each plugin's complete public surface: prose size modifiers
  (prose-sm to prose-2xl), color themes (prose-gray, prose-slate,
  prose-zinc, prose-neutral, prose-stone, prose-invert), element-level
  modifiers (prose-headings, prose-a, prose-img, prose-code), forms
  strategies (base vs class), form-input/textarea/select/multiselect/
  checkbox/radio classes, container-queries variants (@xs to @7xl,
  arbitrary @[400px], named containers @container/sidebar),
  aspect-ratio legacy syntax (aspect-w-16 aspect-h-9), v4 native
  replacements (aspect-video, aspect-square, aspect-[16/9]), and
  v3-to-v4 install translation for every plugin.
  Keywords: tailwindcss typography, tailwindcss forms, tailwindcss
  container queries, tailwindcss aspect ratio, prose, prose-sm,
  prose-lg, prose-xl, prose-2xl, prose-invert, prose-gray, prose-slate,
  prose-zinc, prose-neutral, prose-stone, prose-headings, prose-a,
  prose-img, prose-code, not-prose, max-w-none, form-input, form-select,
  form-textarea, form-checkbox, form-radio, form-multiselect, strategy
  base, strategy class, @container, @sm variant, @md variant, @lg
  variant, @[400px], named container, container/sidebar, aspect-w,
  aspect-h, aspect-video, aspect-square, aspect-ratio plugin, @plugin
  directive, @plugin typography, @plugin forms, require typography,
  require forms, my prose styles not showing, form input not styled,
  container query not working, aspect ratio iframe responsive, how do
  I add markdown styles tailwind, how to style form inputs tailwind,
  responsive video embed tailwind, install tailwind typography plugin
  v4, install tailwind forms plugin v4
license: MIT
compatibility: "Designed for Claude Code. Requires Tailwind CSS v3.4 or v4.0+."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# Tailwind Official Plugins

Four plugins ship from `@tailwindlabs` :

| Plugin | Purpose | v3 status | v4 status |
|--------|---------|-----------|-----------|
| `@tailwindcss/typography` | `prose` class for rendered Markdown | Plugin | Plugin |
| `@tailwindcss/forms` | Form-element base reset | Plugin | Plugin |
| `@tailwindcss/container-queries` | `@container` + `@md:` variants | Plugin | BUILT-IN (do not install) |
| `@tailwindcss/aspect-ratio` | `aspect-w-*` / `aspect-h-*` legacy syntax | Plugin | DEPRECATED (use native `aspect-*`) |

ALWAYS check the version row before installing. Two of the four are
the wrong choice for new v4 code.

Companion skills :

- `tailwind-impl-config-v3` : where to register plugins in JS config
- `tailwind-impl-config-v4` : `@plugin` directive in CSS-first config
- `tailwind-impl-build-postcss-cli` : framework-agnostic install paths
- `tailwind-syntax-modern-utilities` : native aspect-ratio + container-query utilities in v4

## Quick Reference : Install Translation

### v3 (JS config)

```js
// tailwind.config.js
module.exports = {
  plugins: [
    require("@tailwindcss/typography"),
    require("@tailwindcss/forms"),
    require("@tailwindcss/container-queries"),
    require("@tailwindcss/aspect-ratio"),   // legacy use only
  ],
};
```

### v4 (CSS-first)

```css
/* app.css */
@import "tailwindcss";
@plugin "@tailwindcss/typography";
@plugin "@tailwindcss/forms";
/* NO container-queries (built-in)   */
/* NO aspect-ratio (use native aspect-*) */
```

### Install all common plugins

```bash
npm install -D @tailwindcss/typography @tailwindcss/forms
```

For v3 also :

```bash
npm install -D @tailwindcss/container-queries
# only install aspect-ratio if you need legacy syntax for Safari < 14.1
```

## Decision Tree : Which Plugin Do I Need

```
Rendering Markdown / CMS content with default-styled HTML ?
  -> @tailwindcss/typography (prose)

Styling form inputs, selects, checkboxes ?
  -> @tailwindcss/forms

Need element-size-relative responsive styling ?
  ├── v3 -> @tailwindcss/container-queries
  └── v4 -> Built-in, NO plugin needed

Need aspect-ratio constraint on a container with child ?
  ├── v3 modern -> Native aspect-* utilities (no plugin)
  ├── v3 legacy + Safari < 14.1 fallback -> @tailwindcss/aspect-ratio
  └── v4 -> Native aspect-* utilities ONLY
```

## Plugin 1 : @tailwindcss/typography (prose)

### Purpose

Adds the `prose` class. Apply to a wrapper element and every nested
HTML element (`h1`, `p`, `ul`, `code`, `pre`, `blockquote`, `img`, etc.)
gets sensible default typography. Designed for rendered Markdown or
CMS-output where you cannot put utility classes on individual elements.

### Minimum usage

```html
<article class="prose">
  <h1>Hello</h1>
  <p>Markdown-rendered paragraph.</p>
</article>
```

### Size modifiers

| Class | Body size | Use for |
|-------|-----------|---------|
| `prose-sm` | 14px | Sidebars, dense reading |
| `prose-base` | 16px | Default (omit modifier) |
| `prose-lg` | 18px | Long-form articles |
| `prose-xl` | 20px | Marketing pages |
| `prose-2xl` | 24px | Hero blocks |

### Color theme modifiers

`prose-gray` (default), `prose-slate`, `prose-zinc`, `prose-neutral`,
`prose-stone`. Switch per page or per section :

```html
<article class="prose prose-slate">
```

### Dark mode

```html
<article class="prose dark:prose-invert">
```

### Element-level modifiers

Override one element's style without leaving the prose system :

```html
<article class="prose prose-headings:text-blue-900 prose-a:text-emerald-600 prose-img:rounded-xl prose-code:bg-zinc-100">
```

Supported : `prose-headings`, `prose-h1`, `prose-h2`, `prose-h3`,
`prose-h4`, `prose-p`, `prose-a`, `prose-strong`, `prose-em`,
`prose-ul`, `prose-ol`, `prose-li`, `prose-code`, `prose-pre`,
`prose-blockquote`, `prose-img`, `prose-figure`, `prose-figcaption`,
`prose-hr`, `prose-table`, `prose-th`, `prose-td`, `prose-kbd`,
`prose-lead`.

### Escape via `not-prose`

A descendant that should NOT inherit typography :

```html
<article class="prose">
  <p>Markdown text</p>
  <div class="not-prose">
    <!-- Manually-styled UI block, prose styles do not apply here -->
    <button class="px-3 py-1 bg-blue-500 text-white">Action</button>
  </div>
</article>
```

### Width control

Prose ships a 65-character `max-width`. Override with `max-w-none` :

```html
<article class="prose max-w-none">
```

### Customising typography (v3)

```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      typography: {
        DEFAULT: {
          css: {
            color: "#333",
            a: { color: "#3182ce", "&:hover": { color: "#2c5aa0" } },
            h1: { fontWeight: "800" },
          },
        },
      },
    },
  },
  plugins: [require("@tailwindcss/typography")],
};
```

### Customising typography (v4)

Customisation is done via CSS variables in `@theme` :

```css
@import "tailwindcss";
@plugin "@tailwindcss/typography";

@theme {
  --color-prose-body: #333;
  --color-prose-headings: #111;
  --color-prose-links: #3182ce;
}
```

Full variable list in `references/methods.md`.

## Plugin 2 : @tailwindcss/forms

### Purpose

Normalises form-element styling across browsers so Tailwind utilities
can style them without each consumer needing `appearance: none` and
extensive resets.

### Two strategies

```
strategy: "base"  -> Resets EVERY form element globally.
                     No class required.
                     Adds: type=text, textarea, select, checkbox, radio.

strategy: "class" -> Only adds form-* classes.
                     Must apply form-input, form-select, etc. explicitly.
```

Default in v3 + v4 : `base`.

### v3 strategy config

```js
// tailwind.config.js
module.exports = {
  plugins: [
    require("@tailwindcss/forms")({ strategy: "class" }),
  ],
};
```

### v4 strategy config

```css
@plugin "@tailwindcss/forms" {
  strategy: "class";
}
```

### Form classes (when strategy is "class")

```html
<input type="text" class="form-input">
<input type="email" class="form-input">
<input type="password" class="form-input">
<textarea class="form-textarea"></textarea>
<select class="form-select"><option>One</option></select>
<select multiple class="form-multiselect"></select>
<input type="checkbox" class="form-checkbox">
<input type="radio" class="form-radio">
```

### Strategy decision

| Project type | Strategy |
|--------------|----------|
| New project, all forms in your codebase | `base` (less typing) |
| Mixing Tailwind with a CSS framework that styles inputs (Bootstrap, Bulma, MUI) | `class` (scoped opt-in) |
| Email templates that render in many clients | `class` (explicit only) |
| Embedded widget that ships into third-party sites | `class` (zero side effects) |

## Plugin 3 : @tailwindcss/container-queries (v3 ONLY)

### Purpose

Adds the `@container` class to mark an element as a container, plus
variants `@sm:`, `@md:`, `@lg:`, etc. to style based on the container's
size instead of the viewport.

### CRITICAL : v4 does NOT need this plugin

In v4 container queries are part of the framework. Installing this
plugin in a v4 project DOES NOTHING USEFUL and adds bundle weight.

```css
/* v4 entry CSS */
@import "tailwindcss";
/* DO NOT add @plugin "@tailwindcss/container-queries"; */
```

### v3 install

```js
// tailwind.config.js
module.exports = {
  plugins: [require("@tailwindcss/container-queries")],
};
```

### Basic usage

```html
<div class="@container">
  <div class="@lg:underline">Underlined when container >= 32rem</div>
</div>
```

### Variant scale

`@xs` 20rem, `@sm` 24rem, `@md` 28rem, `@lg` 32rem, `@xl` 36rem,
`@2xl` 42rem, `@3xl` 48rem, `@4xl` 56rem, `@5xl` 64rem, `@6xl` 72rem,
`@7xl` 80rem.

### Arbitrary container sizes

```html
<div class="@container">
  <div class="@[400px]:bg-blue-500">Blue when container >= 400px</div>
</div>
```

### Named containers

```html
<div class="@container/sidebar">
  <main class="@container/main">
    <div class="@lg/sidebar:hidden @md/main:flex">
      <!-- Hidden when 'sidebar' container >= 32rem ;
           Flex when 'main' container >= 28rem -->
    </div>
  </main>
</div>
```

## Plugin 4 : @tailwindcss/aspect-ratio (DEPRECATED in v4)

### Purpose (legacy)

Pre-v3.0, the only way to set an aspect ratio on a child element
(typically an iframe or video embed) was the padding-bottom hack. This
plugin provided `aspect-w-*` and `aspect-h-*` to wrap that hack.

### v3 status

Tailwind v3.0 added native `aspect-ratio` utilities (`aspect-video`,
`aspect-square`, `aspect-[16/9]`). The plugin is INSTALLED ONLY when
you need Safari < 14.1 fallback support, which uses the
padding-bottom technique.

### v4 status

DEPRECATED. Native CSS `aspect-ratio` is universally supported
(Safari 15+, Chrome 88+, Firefox 89+) and v4 targets only modern
browsers. NEVER install this plugin in a v4 project.

### v3 install (legacy syntax only)

```js
// tailwind.config.js
module.exports = {
  corePlugins: { aspectRatio: false },   // disable native to avoid conflict
  plugins: [require("@tailwindcss/aspect-ratio")],
};
```

### v3 legacy usage

```html
<div class="aspect-w-16 aspect-h-9">
  <iframe src="https://..."></iframe>
</div>
```

### Modern replacement (any version)

```html
<iframe class="aspect-video w-full" src="https://..."></iframe>
<img class="aspect-square w-32" src="..." />
<div class="aspect-[16/9] w-full bg-zinc-100"></div>
```

Single class. No wrapper required. No corePlugins toggle.

## Quick Reference : Migration Table v3 -> v4

| v3 line | v4 line | Note |
|---------|---------|------|
| `require("@tailwindcss/typography")` | `@plugin "@tailwindcss/typography";` | Stays a plugin |
| `require("@tailwindcss/forms")` | `@plugin "@tailwindcss/forms";` | Stays a plugin |
| `require("@tailwindcss/forms")({ strategy: "class" })` | `@plugin "@tailwindcss/forms" { strategy: "class"; }` | Block syntax |
| `require("@tailwindcss/container-queries")` | (remove entirely) | Built-in |
| `require("@tailwindcss/aspect-ratio")` | (remove entirely, rewrite usages) | Deprecated |
| `corePlugins: { aspectRatio: false }` | (remove entirely) | No longer relevant |

## What This Skill Does NOT Cover

- Third-party community plugins (e.g. `tailwindcss-animate`,
  `@kobalte/tailwindcss`) : see `tailwind-impl-plugins-community`.
- Writing your own custom plugin (`addUtilities`, `matchUtilities`,
  `theme()` callback) : see `tailwind-impl-plugins-custom`.
- Native v4 container-query or aspect-ratio utility behaviour :
  see `tailwind-syntax-modern-utilities`.
- Forms-styling without the plugin (manual `appearance-none` resets) :
  see `tailwind-impl-forms-styling`.

## References

- `references/methods.md` : Every public API for each plugin (full
  signatures, theme variables, config options).
- `references/examples.md` : Realistic patterns : Markdown blog,
  contact form, dashboard with container queries, responsive iframe.
- `references/anti-patterns.md` : v3-syntax-in-v4 traps, forgotten
  not-prose escapes, container-queries in v4, strategy mismatches.

## Official Source Links

- Typography : https://github.com/tailwindlabs/tailwindcss-typography
- Forms : https://github.com/tailwindlabs/tailwindcss-forms
- Container Queries : https://github.com/tailwindlabs/tailwindcss-container-queries
- Aspect Ratio : https://github.com/tailwindlabs/tailwindcss-aspect-ratio
- v4 Typography docs : https://tailwindcss.com/docs/typography-plugin
