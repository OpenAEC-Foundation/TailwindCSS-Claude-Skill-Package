# tailwind-impl-plugins-official : Anti-Patterns

The traps that break official-plugin installs.

## Anti-Pattern 1 : v3 require() syntax in a v4 project

### Symptom

```
warn: tailwind.config.js found but ignored (v4 uses CSS-first config)
```

OR : the project has `tailwind.config.js` listing plugins but `prose`
classes never appear in the compiled CSS.

### Wrong (v4 project)

```js
// tailwind.config.js
module.exports = {
  plugins: [require("@tailwindcss/typography")],
};
```

### Right (v4 project)

```css
/* app.css */
@import "tailwindcss";
@plugin "@tailwindcss/typography";
```

### Root cause

v4 reads NO `tailwind.config.js` by default. Plugins MUST be declared
in the entry CSS via `@plugin`. The `plugins:` array in JS config is
silently ignored unless `@config "./tailwind.config.js"` is also added
to the entry CSS.

## Anti-Pattern 2 : v4 @plugin syntax in a v3 project

### Symptom

Build fails or `@plugin` shows up as a raw CSS rule the browser cannot
parse :

```
Error: Unknown at-rule @plugin
```

### Wrong (v3 project)

```css
@plugin "@tailwindcss/typography";
```

### Right (v3 project)

```js
// tailwind.config.js
module.exports = {
  plugins: [require("@tailwindcss/typography")],
};
```

### Root cause

`@plugin` is a v4-only CSS directive. v3 has no handler for it ; PostCSS
either leaves it (browser rejects) or strips it (no plugin loads).

## Anti-Pattern 3 : Installing container-queries plugin in v4

### Symptom

`npm install @tailwindcss/container-queries` succeeds. Dependency
appears in `package.json`. Adding `@plugin "@tailwindcss/container-queries";`
emits warnings or duplicate `@container` utilities.

### Wrong (v4)

```css
@import "tailwindcss";
@plugin "@tailwindcss/container-queries";   /* unnecessary in v4 */
```

### Right (v4)

```css
@import "tailwindcss";
/* container queries are built-in, do nothing */
```

### Root cause

Tailwind v4 ships container queries as core utilities. The plugin is
v3-specific. Installing it in v4 adds bundle weight, duplicates
container-query utility classes in CSS, and creates a redundant
dependency that will diverge from the v4 implementation over time.

## Anti-Pattern 4 : Installing aspect-ratio plugin in v4

### Symptom

`aspect-w-16 aspect-h-9` does not work. Aspect utilities behave
inconsistently. Or you accidentally enabled the deprecation warning :

```
warn: aspect-w-* and aspect-h-* are deprecated, use aspect-video, aspect-square, aspect-[16/9]
```

### Wrong (v4)

```html
<div class="aspect-w-16 aspect-h-9">
  <iframe src="..."></iframe>
</div>
```

### Right (v4)

```html
<iframe class="aspect-video w-full" src="..."></iframe>
```

### Root cause

v4 dropped support for the padding-bottom hack and the
`aspect-w-*`/`aspect-h-*` plugin. The native `aspect-ratio` CSS
property is universally available in v4-supported browsers. ALWAYS use
`aspect-video`, `aspect-square`, or `aspect-[ratio]` natively.

## Anti-Pattern 5 : Wrong forms strategy for the project

### Symptom (strategy: base in a multi-framework page)

The host page's existing buttons and inputs get unexpectedly restyled.
Bootstrap or MUI components lose their visual identity.

### Symptom (strategy: class without form-* classes)

```html
<input type="text" class="rounded border-zinc-300">
```

The input renders with browser-default appearance (no rounded border,
ugly checkbox, native styling shows through). Tailwind utilities don't
take effect on form elements until you add `form-input`.

### Fix : pick the strategy that matches the context

```
Greenfield app, all forms in your codebase    -> base
Widget embedded in third-party page           -> class
Email template                                -> class
Mixing with Bootstrap, MUI, Bulma             -> class
Static landing page, one form                 -> base (less typing)
```

### Root cause

Strategy `base` applies a GLOBAL reset (`input { ... }`) that affects
EVERY form element on the page, including ones styled by other CSS.
Strategy `class` applies styles only to elements with `form-*` classes,
so unstyled inputs render with browser defaults.

## Anti-Pattern 6 : Forgetting `corePlugins.aspectRatio: false` (v3 aspect plugin)

### Symptom

Both `aspect-video` AND `aspect-w-16 aspect-h-9` emit CSS. Output CSS
is bloated. Two classes target the same element with conflicting
strategies.

### Wrong (v3 with legacy plugin)

```js
module.exports = {
  plugins: [require("@tailwindcss/aspect-ratio")],
  // corePlugins.aspectRatio not disabled
};
```

### Right (v3 with legacy plugin)

```js
module.exports = {
  corePlugins: { aspectRatio: false },
  plugins: [require("@tailwindcss/aspect-ratio")],
};
```

### Root cause

v3 ships native `aspect-*` utilities by default. Installing the legacy
plugin AND keeping the native utility means both emit CSS, so the
output contains duplicate aspect-ratio implementations. Disable the
core utility OR (preferred) drop the plugin entirely and use the
native syntax.

## Anti-Pattern 7 : Markdown content rendering inside a button

### Symptom

Buttons or cards inside a `prose` wrapper inherit prose typography :
auto-link colors, bold paragraph leading, italic blockquote treatment.
Result : ugly UI components inside otherwise-styled articles.

### Wrong

```html
<article class="prose">
  <p>Read more about our pricing.</p>
  <button class="rounded bg-blue-600 px-4 py-2 text-white">
    Pricing details
  </button>   <!-- prose styles bleed onto this button -->
</article>
```

### Right

```html
<article class="prose">
  <p>Read more about our pricing.</p>
  <div class="not-prose">
    <button class="rounded bg-blue-600 px-4 py-2 text-white">
      Pricing details
    </button>
  </div>
</article>
```

### Root cause

`prose` styles cascade through every descendant. Wrap any UI block
that should NOT inherit prose styling with `not-prose`. Common cases :
buttons, cards, code playgrounds, callouts, anything with its own
design system.

## Anti-Pattern 8 : Customising typography with wrong selector specificity

### Symptom

```js
typography: {
  DEFAULT: {
    css: {
      ".prose a": { color: "red" },   // WRONG : selector inside CSS object
    },
  },
}
```

Custom color does not apply. Default link color persists.

### Right

```js
typography: {
  DEFAULT: {
    css: {
      a: { color: "red" },            // bare element selector
    },
  },
}
```

### Root cause

The typography plugin's `css` object uses bare element selectors
(`a`, `h1`, `code`). It auto-prefixes the selectors with `.prose` and
nests them correctly. Manually adding `.prose a` produces a double-
class selector `.prose .prose a` which never matches.

## Anti-Pattern 9 : Container query without `@container` parent

### Symptom

```html
<div class="@lg:underline">
  This never underlines because there is no container declared.
</div>
```

### Right

```html
<div class="@container">
  <div class="@lg:underline">Underlines when container >= 32rem.</div>
</div>
```

### Root cause

`@{size}:` variants are MEANINGLESS without an ancestor marked with
`@container`. The variant matches "the nearest ancestor that is a
container query container". With no such ancestor, the variant never
activates. ALWAYS mark the container EXPLICITLY.

## Anti-Pattern 10 : Mismatched named containers

### Symptom

```html
<div class="@container/main">
  <div class="@lg/sidebar:hidden">
    <!-- This never hides because there is no @container/sidebar ancestor -->
  </div>
</div>
```

### Right

```html
<div class="@container/sidebar">
  <div class="@lg/sidebar:hidden">
    <!-- Now matches : 'sidebar' container exists -->
  </div>
</div>
```

### Root cause

Named variant `@lg/sidebar:` matches an ancestor with EXACTLY
`@container/sidebar`. Typos or missing names mean no match. Keep
named containers consistent and grep your codebase for usage before
renaming.

## Anti-Pattern 11 : Customising prose CSS variables outside @theme (v4)

### Symptom

```css
@import "tailwindcss";
@plugin "@tailwindcss/typography";

:root {
  --tw-prose-body: #333;     /* WRONG : assignment outside @theme */
}
```

Custom variable does not affect prose output. Default colors persist.

### Right

```css
@import "tailwindcss";
@plugin "@tailwindcss/typography";

@theme {
  --color-prose-body: #333;
}
```

### Root cause

In v4 the typography plugin reads its colors from the v4 theme system,
which is the `@theme` directive. Assigning `--tw-prose-body` on
`:root` does not feed into the build-time generation of prose
selectors. ALWAYS customise via `@theme`.

## Anti-Pattern 12 : Forgetting the `-D` (devDependency) flag

### Symptom

Plugin appears in `dependencies` instead of `devDependencies` in
`package.json`. Production deploy bundles the plugin into the runtime,
inflating image size.

### Wrong

```bash
npm install @tailwindcss/typography
```

### Right

```bash
npm install -D @tailwindcss/typography
```

### Root cause

Tailwind plugins run at BUILD TIME. They generate CSS during
compilation and play NO role at runtime. Always install with `-D` so
the production bundle excludes them.

## Anti-Pattern 13 : Loading the same plugin twice

### Symptom

Compiled CSS has duplicate `prose` selectors. Bundle size doubles.
Sometimes selectors conflict in unexpected ways.

### Wrong (v4)

```css
@plugin "@tailwindcss/typography";
@plugin "@tailwindcss/typography";   /* duplicate */
```

### Wrong (v3)

```js
plugins: [
  require("@tailwindcss/typography"),
  require("@tailwindcss/typography"),   /* duplicate */
]
```

### Root cause

The plugin loader does not deduplicate. Loading the same plugin twice
emits twice the CSS. Audit the plugin list before committing.
