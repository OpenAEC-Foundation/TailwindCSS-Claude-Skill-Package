# Anti-Patterns : Tailwind in Next.js

Each trap : symptom, root cause, fix, verification step.

## Trap 1 : Importing globals.css Twice

### Symptom
CSS bundle weight roughly doubles. Specificity wars where the same
selector wins inconsistently. Hot reload becomes sluggish.

### Root cause
Importing `app/globals.css` in both `app/layout.tsx` AND in a sub-layout
(`app/(marketing)/layout.tsx`), or importing it in both
`pages/_app.tsx` AND in a page-level component. Next.js does NOT
deduplicate top-level CSS imports across multiple entry files when one
of them is a nested layout.

### Fix
Import the global stylesheet exactly once :

- App Router : only in `app/layout.tsx`. NEVER in nested layouts.
- Pages Router : only in `pages/_app.tsx`. NEVER in individual pages.

### Verification
`grep -rn "globals.css" app pages` should return exactly one line.

## Trap 2 : Forgetting @tailwindcss/postcss Under v4

### Symptom
`npm run dev` starts, but no classes apply. Console shows no error.
DevTools shows the class on the element but no rule in any stylesheet.

### Root cause
v4 split the PostCSS integration out of the main `tailwindcss` package.
Installing only `tailwindcss` (without `@tailwindcss/postcss`) and
keeping a v3-style `postcss.config.js` :

```js
module.exports = { plugins: { tailwindcss: {}, autoprefixer: {} } };
```

leaves PostCSS with no plugin that understands `@import "tailwindcss"`.
The directive passes through unchanged.

### Fix
Install the v4 PostCSS plugin :

```bash
npm install @tailwindcss/postcss
```

Rewrite the config :

```js
// postcss.config.mjs
const config = {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
export default config;
```

Delete `autoprefixer` from the plugins list ; v4 includes it.

### Verification
After `npm run dev`, view-source on any page. The injected `<link>` to
`/_next/static/css/*.css` should serve a file containing actual utility
rules, not just `@import "tailwindcss";` text.

## Trap 3 : v3 Content Glob Misses Catch-All Routes

### Symptom
Static pages render Tailwind utilities correctly, but dynamic routes
(any folder named `[id]`, `[...slug]`, `[[...slug]]`) show unstyled
elements. Same className, same component, different result depending on
which page renders it.

### Root cause
The default content glob `./app/**/*.{js,ts,jsx,tsx,mdx}` treats `[id]`
as a character-class pattern. Folders whose names start with `[` are
silently excluded. Files inside are never scanned, so their tokens
never become CSS rules.

Referenced by Tailwind issue 16287.

### Fix
v3 : add an escaped-bracket pattern to the content array :

```js
content: [
  "./app/**/*.{js,ts,jsx,tsx,mdx}",
  "./app/[[]**[]]/**/*.{js,ts,jsx,tsx,mdx}",
],
```

v4 : add an `@source` :

```css
@import "tailwindcss";
@source "./app/[[]**[]]/**/*.{js,ts,jsx,tsx,mdx}";
```

### Verification
Build and grep the output CSS for a class used only inside a catch-all
folder :

```bash
npm run build
grep -c "your-unique-class" .next/static/css/*.css
```

Result should be > 0.

## Trap 4 : Bare @theme With var() for next/font

### Symptom
Tailwind utilities resolve correctly to CSS, but the font family in the
browser is the fallback (`ui-sans-serif`) instead of the loaded Google
font. `--font-inter` is defined on `<html>` in DevTools, but
`font-family` on `font-sans` elements does not reference it.

### Root cause
Using bare `@theme` instead of `@theme inline` :

```css
@theme {
  --font-sans: var(--font-inter), ui-sans-serif, system-ui, sans-serif;
}
```

Bare `@theme` resolves `var(--font-inter)` at BUILD time. At build time,
next/font has not yet injected the variable onto any element, so
`var(--font-inter)` resolves to its CSS default (the empty string) and
the token compiles to `--font-sans: , ui-sans-serif, ...`. The leading
empty entry corrupts the entire family list.

### Fix
Use `@theme inline` :

```css
@theme inline {
  --font-sans: var(--font-inter), ui-sans-serif, system-ui, sans-serif;
}
```

`@theme inline` emits the literal `var(...)` text into the output CSS,
deferring resolution to the runtime browser where next/font has already
injected the variable on `<html>`.

### Verification
Inspect a `font-sans` element in DevTools. Computed `font-family` should
start with `Inter` (or whatever Google font you loaded). The rule's
source value should contain `var(--font-inter)` verbatim.

## Trap 5 : Adding "use client" Solely for Styling

### Symptom
Bundle size grows. Server Components silently become Client Components
across an entire subtree. Initial page weight increases.

### Root cause
Belief that Tailwind classes require client runtime. They do not.
Tailwind is a compile-time tool ; the output is plain CSS. Every Next.js
component (RSC, Client, layout, page) consumes the same stylesheet.

### Fix
Remove the `"use client"` directive when added only to apply Tailwind
classes. Keep `"use client"` only when the component genuinely needs :

- React hooks (`useState`, `useEffect`, `useRef`, `useReducer`,
  `useContext`)
- Browser APIs (`window`, `document`, `localStorage`)
- Event handlers (`onClick`, `onChange`, etc., that need real handlers
  not just to be passed down)

### Verification
Server-rendered component should render fully styled in
`view-source:`'s HTML, not just in the hydrated DOM.

## Trap 6 : Turbopack Arbitrary-Value Miss

### Symptom
Under `next dev --turbo` (Next 16) plus Tailwind v4.2.1, classes like
`aspect-[12/5]`, `z-[100]`, `h-[80vh]` show in the DOM but the
corresponding CSS rules are missing. Layout looks like the utility
never applied. Webpack `next dev` (no `--turbo`) compiles them
correctly.

### Root cause
Turbopack's Rust-based source scanner misses specific arbitrary-value
tokens during incremental builds, especially in files that also import
heavy third-party libraries (referenced by tailwindlabs/tailwindcss
issue 19825). The first build can catch them ; subsequent edits in the
same file drop them.

### Fix
Choose one :

1. Inline-style the arbitrary value :
   ```tsx
   <div style={{ aspectRatio: "12/5" }} />
   ```
2. Force-safelist the exact tokens via `@source inline` :
   ```css
   @source inline("aspect-[12/5] z-[100] h-[80vh]");
   ```
3. Remove `--turbo` from the `dev` script until upstream fix lands.

### Verification
Run a production build (`npm run build`) under both Turbopack and
webpack. Grep both output CSS bundles for the arbitrary-value selector :

```bash
grep "aspect-\[12/5\]" .next/static/css/*.css
```

Both bundles must contain the rule. If only one does, the build path
with the missing rule is buggy.

## Trap 7 : Mixing v3 Directives With v4 PostCSS Plugin

### Symptom
Build fails with `Unknown at-rule @tailwind` or similar. Or build
succeeds but no utilities render.

### Root cause
Migrating to v4 incompletely : installed `@tailwindcss/postcss` but
left the old `@tailwind base;`, `@tailwind components;`,
`@tailwind utilities;` lines in `globals.css`. v4 does not understand
`@tailwind` (it understands only `@import "tailwindcss"`).

### Fix
Replace the three `@tailwind` directives with one line :

```css
@import "tailwindcss";
```

Migration upgrade guide : https://tailwindcss.com/docs/upgrade-guide

### Verification
`globals.css` should contain no `@tailwind` directive anywhere.

## Trap 8 : Importing CSS in a Server Component Page Instead of Layout

### Symptom
Styles work on one page, missing on another. App Router. No build
errors.

### Root cause
Importing `globals.css` inside `app/page.tsx` instead of `app/layout.tsx`.
The import is associated with that specific route's CSS chunk, so other
routes that do not transitively import the same file get no CSS.

### Fix
Move the import to `app/layout.tsx` :

```tsx
import "./globals.css";
```

The root layout is the highest stable mount point ; importing global
CSS there guarantees every route gets it.

### Verification
Navigate between two different routes (`/`, `/about`). Both should be
styled identically. `view-source:` on both should reference the same
CSS bundle URL in the `<link>` tag.
