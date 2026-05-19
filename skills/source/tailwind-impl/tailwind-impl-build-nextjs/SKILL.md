---
name: tailwind-impl-build-nextjs
description: >
  Use when installing or wiring Tailwind CSS into a Next.js project, for
  either the App Router or the Pages Router, on Tailwind v3 or v4, and when
  hooking next/font/google families into the Tailwind theme so utilities
  like font-sans resolve to the loaded family. Prevents the four most
  common Next-js install traps: importing globals.css in the wrong file
  for the active router, forgetting the @tailwindcss/postcss plugin under
  v4, breaking the v3 content glob on catch-all route folders named
  [...slug], and pasting font CSS variables into next/font/local without
  declaring them inside @theme inline so they never become utilities.
  Covers v4 App Router with @import tailwindcss, v4 Pages Router, v3 App
  Router with @tailwind directives, v3 Pages Router, next/font wiring via
  className on html plus @theme inline { --font-sans: var(--font-inter) },
  why React Server Components have zero runtime cost from Tailwind, and
  the Turbopack arbitrary-value miss workaround for Next 16 plus v4 from
  tailwindlabs/tailwindcss issue 19825.
  Keywords: tailwind nextjs, tailwind next-js, install tailwind next,
  next app router tailwind, next pages router tailwind, app/globals-css,
  pages/_app-tsx, @tailwindcss/postcss, postcss-config-mjs, tailwind v4
  next, tailwind v3 next, @import tailwindcss next, @tailwind directives
  next, tailwind init -p next, next/font google tailwind, next/font local
  tailwind, --font-inter @theme inline, font-sans next/font, RSC
  tailwind, server components tailwind, no runtime cost styling,
  turbopack arbitrary value miss, aspect-[12/5] missing turbopack,
  z-[100] missing, next 16 tailwind v4 issue 19825, catch-all route
  [...slug] glob, @source './[[]**[]]', how do I add tailwind to next,
  my tailwind classes are not applying in nextjs, blank styles next,
  why doesn't dark mode work next, getting started tailwind next.
license: MIT
compatibility: "Designed for Claude Code. Requires Tailwind CSS v3.4 or v4.0+."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# Tailwind CSS in Next.js : Install and Wire-up

Covers every supported combination of Tailwind version (v3.4 vs v4.0+) and
Next.js router (App Router vs Pages Router), plus next/font integration
and the documented Turbopack quirk.

Companion skills :

- `tailwind-impl-config-v4` : v4 CSS-first config surface (@theme, @source)
- `tailwind-impl-config-v3` : v3 JS config surface (tailwind.config.js)
- `tailwind-impl-build-vite` : same install topic for the Vite plugin
- `tailwind-core-v3-vs-v4` : version-wide API differences

## Decision : Which Combination Do You Have

| Tailwind version | Router | Entry CSS lives in | Imported from |
|------------------|--------|--------------------|---------------|
| v4 | App Router | `app/globals.css` | `app/layout.tsx` |
| v4 | Pages Router | `styles/globals.css` | `pages/_app.tsx` |
| v3 | App Router | `app/globals.css` | `app/layout.tsx` |
| v3 | Pages Router | `styles/globals.css` | `pages/_app.tsx` |

The CSS file location is identical across versions. Only the file
contents differ : `@import "tailwindcss";` for v4, three `@tailwind`
directives for v3.

## Minimum Setup : Tailwind v4 + Next.js App Router

```bash
npm install tailwindcss @tailwindcss/postcss postcss
```

`postcss.config.mjs` (project root) :

```js
const config = {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
export default config;
```

`app/globals.css` :

```css
@import "tailwindcss";
```

`app/layout.tsx` :

```tsx
import "./globals.css";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

That is the complete v4 + App Router install. No `tailwind.config.js`, no
`@tailwind` directives, no `autoprefixer`, no `postcss-import`. The
`@tailwindcss/postcss` plugin runs the Oxide engine which bundles all
three.

## Minimum Setup : Tailwind v4 + Next.js Pages Router

Install and PostCSS config are identical to the App Router setup above.
Only the import location moves :

`styles/globals.css` :

```css
@import "tailwindcss";
```

`pages/_app.tsx` :

```tsx
import "@/styles/globals.css";
import type { AppProps } from "next/app";

export default function App({ Component, pageProps }: AppProps) {
  return <Component {...pageProps} />;
}
```

## Minimum Setup : Tailwind v3 + Next.js App Router

```bash
npm install -D tailwindcss@3 postcss autoprefixer
npx tailwindcss init -p
```

`tailwindcss init -p` generates two files :

`tailwind.config.js` :

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./app/**/*.{js,ts,jsx,tsx,mdx}",
    "./pages/**/*.{js,ts,jsx,tsx,mdx}",
    "./components/**/*.{js,ts,jsx,tsx,mdx}",
    "./src/**/*.{js,ts,jsx,tsx,mdx}",
  ],
  theme: { extend: {} },
  plugins: [],
};
```

`postcss.config.js` :

```js
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

`app/globals.css` :

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

`app/layout.tsx` : identical to the v4 case, just `import "./globals.css";`.

## Minimum Setup : Tailwind v3 + Next.js Pages Router

Same install, same `tailwind.config.js`, same `postcss.config.js`. The CSS
file lives at `styles/globals.css` (same three `@tailwind` directives),
imported once in `pages/_app.tsx` :

```tsx
import "@/styles/globals.css";
import type { AppProps } from "next/app";

export default function App({ Component, pageProps }: AppProps) {
  return <Component {...pageProps} />;
}
```

## Wire next/font/google Into the Tailwind Theme

next/font emits a CSS variable that you opt into by applying a className.
Tailwind reads that variable as a token via `@theme` (v4) or
`fontFamily.extend` (v3).

### v4 pattern

`app/layout.tsx` :

```tsx
import { Inter } from "next/font/google";
import "./globals.css";

const inter = Inter({
  subsets: ["latin"],
  variable: "--font-inter",
});

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={inter.variable}>
      <body className="font-sans">{children}</body>
    </html>
  );
}
```

`app/globals.css` :

```css
@import "tailwindcss";

@theme inline {
  --font-sans: var(--font-inter), ui-sans-serif, system-ui, sans-serif;
}
```

ALWAYS use `@theme inline { ... }` (not bare `@theme { ... }`) when the
token value contains a `var(...)` reference to another CSS variable that
exists at runtime. Bare `@theme` resolves the variable at build time and
emits the resolved literal, which is empty before next/font runs and
strips the family.

### v3 pattern

`pages/_app.tsx` (or `app/layout.tsx`) wires the className the same way.
The config file declares the family :

```js
// tailwind.config.js
module.exports = {
  content: [/* unchanged */],
  theme: {
    extend: {
      fontFamily: {
        sans: ["var(--font-inter)", "ui-sans-serif", "system-ui", "sans-serif"],
      },
    },
  },
};
```

## React Server Components : Zero Runtime Cost

Tailwind is a build-time compiler. It scans source files for class
tokens, generates a single static CSS bundle, and injects that bundle via
the same `<link rel="stylesheet">` Next.js uses for every other CSS
import. There is no client runtime, no Server Component restriction, no
`"use client"` boundary needed for styling. RSC, Client Components, and
Server Actions all consume the same CSS.

Consequences :

- NEVER move a Server Component to Client just to use Tailwind classes.
- ALWAYS apply Tailwind classes directly on RSC output ; no wrapper.
- Bundle size scales with the number of utilities used, not with the
  number of components. Adding a hundred RSC components that share the
  same utilities adds zero CSS bytes.

## Turbopack Arbitrary-Value Miss (Issue 19825)

Affects Next.js 16.1.6 + Tailwind v4.2.1 + `next dev --turbo` (and
`next dev` once Turbopack becomes the default). Classes with arbitrary
square-bracket values render in the DOM but the corresponding CSS rule
is missing from the injected stylesheet.

Reproduced tokens : `aspect-[12/5]`, `z-[100]`, `h-[80vh]`. The same
classes compile correctly under webpack (`next dev` without `--turbo`).

### Workarounds

1. **Inline style for layout-critical values** :
   ```tsx
   <div style={{ aspectRatio: "12/5" }} className="w-full bg-zinc-900" />
   ```
2. **Safelist the exact token via `@source inline`** so the scanner is
   forced to emit the rule regardless of Turbopack's incremental scan :
   ```css
   @source inline("aspect-[12/5] z-[100] h-[80vh]");
   ```
3. **Disable Turbopack for the affected build** : remove `--turbo` from
   `next dev` until the upstream fix lands.

NEVER ship `aspect-[12/5]` style arbitrary values to a Turbopack
production build without first verifying the rule appears in
`.next/static/css`. The DOM showing the class is not proof the CSS
exists.

## Catch-All Route Glob Trap (Tailwind v3)

The default v3 content glob `./app/**/*.{js,ts,jsx,tsx,mdx}` treats
square brackets as a character class. Files inside a folder named
`[...slug]` or `[id]` are silently excluded from the scan. Symptom :
classes on dynamic-route pages produce no CSS.

Fix (v3) : add a second content entry that escapes the brackets, or
list the catch-all folder explicitly :

```js
content: [
  "./app/**/*.{js,ts,jsx,tsx,mdx}",
  "./app/[[]**[]]/*.{js,ts,jsx,tsx,mdx}",
],
```

Fix (v4) : add an explicit `@source` :

```css
@import "tailwindcss";
@source "./app/[[]**[]]/**/*.{js,ts,jsx,tsx,mdx}";
```

## Verification Checklist

1. CSS file imports exactly once (root layout for App Router, `_app.tsx`
   for Pages Router). Importing twice doubles the bundle.
2. `npm run dev` shows utilities applying within one second on hot
   reload.
3. Dynamic-route pages (any folder with `[id]`, `[...slug]`,
   `[[...slug]]`) render utilities correctly. If not, the bracket-glob
   trap is active.
4. If next/font is wired : a token like `text-3xl` renders with the
   loaded family, not the browser fallback. Inspect computed
   `font-family` in DevTools.
5. Production : `npm run build` finishes without `PostCSS plugin
   tailwindcss requires PostCSS 8` (v3 only ; install `postcss@8`).
6. Turbopack : if `next dev --turbo`, confirm arbitrary-value classes
   actually emit CSS, not just DOM attributes.

## References

- `references/methods.md` : full per-router install walkthroughs with
  every file and exact contents
- `references/examples.md` : working `app/layout.tsx`, `pages/_app.tsx`,
  fonts wiring, mixed-mode setups
- `references/anti-patterns.md` : the eight Next.js-specific traps with
  symptoms, root cause, fix, and verification

## Sources

- https://tailwindcss.com/docs/installation/framework-guides/nextjs
- https://v3.tailwindcss.com/docs/guides/nextjs
- https://github.com/tailwindlabs/tailwindcss/issues/19825
- https://github.com/tailwindlabs/tailwindcss/issues/16287
- https://nextjs.org/docs/app/api-reference/components/font
