# tailwind-impl-plugins-official : Examples

End-to-end realistic patterns for each official plugin.

## Example 1 : Markdown Blog Post (Typography, v4)

### Setup

```bash
npm install -D @tailwindcss/typography
```

```css
/* app.css */
@import "tailwindcss";
@plugin "@tailwindcss/typography";
```

### Layout

```tsx
// blog/[slug]/page.tsx
export default function BlogPost({ html }: { html: string }) {
  return (
    <article className="prose prose-lg prose-slate dark:prose-invert mx-auto px-4">
      <div dangerouslySetInnerHTML={{ __html: html }} />
    </article>
  );
}
```

### Result

Rendered Markdown gets : 18px body, slate color palette, inverted in
dark mode, automatic styling for headings, links, blockquotes, code
blocks, lists, images, tables. No utility classes needed on individual
elements.

## Example 2 : Markdown Blog Post (Typography, v3 with customisation)

### Setup

```bash
npm install -D @tailwindcss/typography
```

### Custom theme

```js
// tailwind.config.js
module.exports = {
  content: ["./src/**/*.{html,js,jsx,ts,tsx}"],
  theme: {
    extend: {
      typography: (theme) => ({
        DEFAULT: {
          css: {
            "--tw-prose-body": theme("colors.zinc.700"),
            "--tw-prose-headings": theme("colors.zinc.900"),
            "--tw-prose-links": theme("colors.emerald.600"),
            a: { textDecoration: "none", fontWeight: "600" },
            "code::before": { content: '""' },
            "code::after": { content: '""' },
            "h1, h2": { fontFamily: theme("fontFamily.serif") },
          },
        },
        invert: {
          css: {
            "--tw-prose-body": theme("colors.zinc.300"),
            "--tw-prose-headings": theme("colors.white"),
          },
        },
      }),
    },
  },
  plugins: [require("@tailwindcss/typography")],
};
```

### Layout

```jsx
<article className="prose prose-lg dark:prose-invert max-w-none">
  {markdownHTML}
</article>
```

## Example 3 : Mixing Prose with UI Components (`not-prose`)

```tsx
<article className="prose">
  <h1>Documentation</h1>
  <p>Click the button below to copy.</p>

  <div className="not-prose">
    <button className="rounded-md bg-zinc-900 px-4 py-2 text-white">
      Copy snippet
    </button>
    <pre className="rounded bg-zinc-100 p-3 font-mono text-sm">npm install foo</pre>
  </div>

  <p>This text is back inside prose.</p>
</article>
```

The button and pre block keep their Tailwind utility styling untouched
by prose. The paragraph that follows resumes prose typography.

## Example 4 : Contact Form (Forms, strategy: base)

### Setup

```bash
npm install -D @tailwindcss/forms
```

```css
/* v4 */
@import "tailwindcss";
@plugin "@tailwindcss/forms";
```

### Form (no form-* classes needed)

```html
<form class="space-y-4">
  <label class="block">
    <span class="text-sm font-medium">Name</span>
    <input type="text" name="name"
           class="mt-1 block w-full rounded border-zinc-300 shadow-sm
                  focus:border-blue-500 focus:ring-blue-500" />
  </label>

  <label class="block">
    <span class="text-sm font-medium">Email</span>
    <input type="email" name="email"
           class="mt-1 block w-full rounded border-zinc-300 shadow-sm
                  focus:border-blue-500 focus:ring-blue-500" />
  </label>

  <label class="block">
    <span class="text-sm font-medium">Message</span>
    <textarea name="message" rows="4"
              class="mt-1 block w-full rounded border-zinc-300 shadow-sm"></textarea>
  </label>

  <label class="inline-flex items-center">
    <input type="checkbox" class="rounded border-zinc-300 text-blue-600" />
    <span class="ml-2 text-sm">Subscribe to updates</span>
  </label>

  <button type="submit" class="rounded-md bg-blue-600 px-4 py-2 text-white">
    Send
  </button>
</form>
```

Note : no `form-input`, `form-textarea`, etc. The base strategy resets
ALL form elements globally so utilities `rounded`, `border-zinc-300`,
`focus:ring-blue-500` work directly.

## Example 5 : Embedded Widget (Forms, strategy: class)

For a widget that ships into third-party sites where global resets
would conflict with the host page styling.

### v4 config

```css
@import "tailwindcss";
@plugin "@tailwindcss/forms" {
  strategy: "class";
}
```

### v3 config

```js
module.exports = {
  plugins: [
    require("@tailwindcss/forms")({ strategy: "class" }),
  ],
};
```

### Markup (explicit form-* classes)

```html
<div id="my-widget">
  <input type="text" class="form-input rounded border-zinc-300" />
  <select class="form-select rounded border-zinc-300">
    <option>One</option>
  </select>
  <input type="checkbox" class="form-checkbox rounded text-blue-600" />
</div>
```

Only inputs with `form-*` get the reset. The rest of the host page is
untouched.

## Example 6 : Dashboard Card with Container Queries (v3)

### Setup (v3)

```bash
npm install -D @tailwindcss/container-queries
```

```js
// tailwind.config.js
module.exports = {
  plugins: [require("@tailwindcss/container-queries")],
};
```

### Card component

```html
<div class="@container rounded-lg border border-zinc-200 p-4">
  <!-- Header reflows based on CARD width, not viewport width -->
  <div class="flex flex-col gap-2 @md:flex-row @md:items-center @md:justify-between">
    <h2 class="text-lg font-semibold">Sales</h2>
    <span class="text-xs text-zinc-500 @md:text-sm">Last 30 days</span>
  </div>

  <!-- Grid layout : 1 col tiny, 2 col medium, 4 col wide -->
  <div class="mt-4 grid grid-cols-1 gap-3 @sm:grid-cols-2 @lg:grid-cols-4">
    <div class="rounded bg-zinc-50 p-2">$12,400</div>
    <div class="rounded bg-zinc-50 p-2">128 orders</div>
    <div class="rounded bg-zinc-50 p-2">34 customers</div>
    <div class="rounded bg-zinc-50 p-2">2.4% conv.</div>
  </div>
</div>
```

The same card adapts whether it sits in a narrow sidebar (1 column)
or a wide main panel (4 columns) without media queries.

## Example 7 : Same Card in v4 (no plugin)

### Setup

```css
@import "tailwindcss";
/* NO @plugin "@tailwindcss/container-queries"; */
```

### Markup (identical to v3 example)

```html
<div class="@container rounded-lg border border-zinc-200 p-4">
  <div class="flex flex-col gap-2 @md:flex-row @md:items-center @md:justify-between">
    ...
  </div>
</div>
```

Variants work the same because v4 ships container queries natively.
Skipping the plugin install saves one npm dependency and one line
in CSS.

## Example 8 : Named Containers for Layout Composition (v3)

```html
<div class="@container/sidebar flex-shrink-0 border-r">
  <main class="@container/main flex-1">
    <header class="@lg/main:flex @lg/main:justify-between">
      <!-- Layout responds to MAIN container width -->
    </header>

    <aside class="@md/sidebar:block hidden">
      <!-- Visibility responds to SIDEBAR container width -->
    </aside>
  </main>
</div>
```

Each container tracks its own size independently. A sidebar at 28rem
can show its nested aside while the main panel still has its own
breakpoints below 32rem.

## Example 9 : Responsive Video Embed (modern, both v3 and v4)

NO plugin needed. Native utility is the modern path.

```html
<div class="mx-auto max-w-3xl">
  <iframe class="aspect-video w-full rounded-lg"
          src="https://www.youtube.com/embed/..."
          frameborder="0" allowfullscreen></iframe>
</div>
```

Or with an arbitrary ratio :

```html
<iframe class="aspect-[21/9] w-full" src="..."></iframe>
<img    class="aspect-[4/3] w-full object-cover" src="..." />
<div    class="aspect-square w-32 bg-zinc-100"></div>
```

## Example 10 : Legacy Aspect Ratio Plugin (v3, Safari < 14.1 support)

Use ONLY if you need Safari 14.0 or older to honour aspect ratio.

### Setup

```bash
npm install -D @tailwindcss/aspect-ratio
```

```js
// tailwind.config.js
module.exports = {
  corePlugins: { aspectRatio: false },   // disable native conflict
  plugins: [require("@tailwindcss/aspect-ratio")],
};
```

### Markup

```html
<div class="aspect-w-16 aspect-h-9">
  <iframe src="..."></iframe>
</div>
```

### Migration to modern

```html
<!-- Before: legacy plugin -->
<div class="aspect-w-16 aspect-h-9"><iframe src="..."></iframe></div>

<!-- After: native v4 / v3.x -->
<iframe class="aspect-video w-full" src="..."></iframe>
```

## Example 11 : Stacking All Plugins (v4)

```css
/* app.css */
@import "tailwindcss";
@plugin "@tailwindcss/typography";
@plugin "@tailwindcss/forms" {
  strategy: "class";
}
/* container-queries built-in, no @plugin needed */
/* aspect-ratio deprecated, no @plugin allowed */

@theme {
  --color-prose-links: #0070f3;
}
```

```jsx
// Marketing page mixing prose article + signup form + container card
<main className="mx-auto max-w-6xl p-8">
  <article className="prose prose-lg dark:prose-invert">
    <h1>Product launch</h1>
    <p>Read about our latest release...</p>
  </article>

  <section className="not-prose mt-12">
    <form>
      <input type="email" className="form-input rounded border-zinc-300" />
      <button className="rounded bg-blue-600 px-4 py-2 text-white">
        Subscribe
      </button>
    </form>
  </section>

  <section className="@container mt-12 rounded-lg border p-4">
    <div className="grid grid-cols-1 @md:grid-cols-2 @lg:grid-cols-3 gap-4">
      <div>Feature one</div>
      <div>Feature two</div>
      <div>Feature three</div>
    </div>
  </section>
</main>
```

Three plugins (well, two plugins + built-in container queries) working
together cleanly.

## Example 12 : Stacking All Plugins (v3)

```js
// tailwind.config.js
module.exports = {
  content: ["./src/**/*.{html,js,jsx,ts,tsx}"],
  theme: {
    extend: {
      typography: (theme) => ({
        DEFAULT: { css: { "--tw-prose-links": "#0070f3" } },
      }),
    },
  },
  plugins: [
    require("@tailwindcss/typography"),
    require("@tailwindcss/forms")({ strategy: "class" }),
    require("@tailwindcss/container-queries"),
  ],
};
```

Same markup as example 11 works identically.
