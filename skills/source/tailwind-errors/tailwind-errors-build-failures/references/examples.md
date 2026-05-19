# tailwind-errors-build-failures : Examples

Full diagnostic walk-throughs for the most common build failures.

## Example 1 : Dynamic Class Name (Most Common)

### Reported symptom

"I pass a color prop and apply `bg-${color}-500` but the button is
unstyled."

### Diagnosis

```bash
grep -rn "bg-blue-500" src/   # found
grep "bg-blue-500" dist/output.css   # NOT FOUND
```

Class is in source as `bg-${color}-500` not as a complete string.
Scanner sees `bg-` and `-500` but cannot assemble them.

### Fix

```jsx
// Before
function Button({ color, children }) {
  return <button className={`bg-${color}-500 hover:bg-${color}-600`}>{children}</button>;
}

// After
const colorMap = {
  blue:   "bg-blue-500 hover:bg-blue-600",
  red:    "bg-red-500 hover:bg-red-600",
  green:  "bg-green-500 hover:bg-green-600",
};
function Button({ color, children }) {
  return <button className={colorMap[color]}>{children}</button>;
}
```

### Verify

```bash
npm run build
grep "bg-blue-500" dist/output.css   # found now
```

## Example 2 : New Directory Not in v3 Content Glob

### Reported symptom

"I added `app/admin/` with new pages. They render with no styles."

### Diagnosis

```js
// tailwind.config.js (existing)
module.exports = {
  content: ["./pages/**/*.{js,ts,jsx,tsx}"],
};
```

`app/` is NOT in the glob. The whole new directory is invisible to
the scanner.

### Fix

```js
module.exports = {
  content: [
    "./pages/**/*.{js,ts,jsx,tsx}",
    "./app/**/*.{js,ts,jsx,tsx}",     // ADD
    "./components/**/*.{js,ts,jsx,tsx}",
  ],
};
```

### Verify

Restart `npm run dev` (config change requires restart in v3). Open
the admin page. Styles render.

## Example 3 : Monorepo Package Not Detected (v4)

### Reported symptom

"Importing `<Button />` from our internal UI package shows it
unstyled even though the package uses Tailwind classes."

### Diagnosis

The UI package lives in `packages/ui/src/`. The current app is at
`apps/web/`. Auto-detect only sees the `apps/web/` tree.

### Fix

```css
/* apps/web/src/app.css */
@import "tailwindcss";
@source "../../../packages/ui/src";
```

### Alternative : pin source root to monorepo root

```css
@import "tailwindcss" source("../../../");
```

ALWAYS prefer the explicit `@source` line ; broader scope risks
scanning unrelated dirs.

## Example 4 : Vite Cache Stale After Config Change

### Reported symptom

"I added a new color in @theme but it doesn't show up in compiled
CSS even after rebuilding."

### Diagnosis

```bash
grep -c "text-brand" dist/assets/index-*.css   # 0
```

But the class IS in source AND `@theme` has `--color-brand: #...`.

The Vite dep cache is serving stale PostCSS output.

### Fix

```bash
rm -rf node_modules/.vite
npm run dev
```

### Verify

```bash
grep -c "text-brand" dist/assets/index-*.css   # > 0
```

## Example 5 : Next.js Cache Stale After Upgrade

### Reported symptom

"I upgraded from Tailwind v3 to v4. Build succeeds but old styles
persist."

### Diagnosis

`.next/` cached the v3 PostCSS output. The new v4 compilation never
runs because Next.js sees the cache hit.

### Fix

```bash
rm -rf .next
npm run dev
```

### Verify

Hot reload now serves v4-style output (look for `@layer` directives in
DevTools and confirm Lightning CSS prefixes).

## Example 6 : Turbopack Arbitrary-Value Miss

### Reported symptom

"`<div className="aspect-[12/5] z-[100]">` renders without aspect
ratio or stacking context. Same code works fine with `next dev` (no
turbo)."

### Diagnosis

Per issue 19825 : Next.js 16.1.6 + Turbopack + Tailwind v4.2.1 misses
certain arbitrary values in incremental builds, especially in files
that import third-party libraries.

### Fix (option 1) : drop --turbo

```json
// package.json
{
  "scripts": {
    "dev": "next dev"
  }
}
```

### Fix (option 2) : inline style

```jsx
<div style={{ aspectRatio: "12/5", zIndex: 100 }}>
```

### Fix (option 3) : force-include via @source inline

```css
@import "tailwindcss";
@source inline("aspect-[12/5] z-[100]");
```

The arbitrary values are now generated regardless of detection.

## Example 7 : Astro 4.0.8 Regression

### Reported symptom

"After running `npm update` my Astro+Tailwind project lost half its
styles."

### Diagnosis

```bash
cat package.json | grep tailwind
# "tailwindcss": "4.0.8"
# "@tailwindcss/vite": "4.0.8"
```

Issue 16733 confirms v4.0.8 breaks Astro component packages.

### Fix : pin to v4.0.7

```bash
npm install tailwindcss@4.0.7 @tailwindcss/vite@4.0.7
```

Then in `package.json` :

```json
{
  "dependencies": {
    "tailwindcss": "4.0.7",
    "@tailwindcss/vite": "4.0.7"
  }
}
```

### Verify

```bash
rm -rf .astro dist node_modules/.vite
npm run dev
```

## Example 8 : @tailwind Directives in v4 Project

### Reported symptom

"Build emits warnings about deprecated @tailwind directives, and the
output CSS is only 1 KB."

### Diagnosis

```css
/* src/app.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

This is v3 syntax. v4 ignores it and emits a tiny header-only CSS.

### Fix

```css
/* src/app.css */
@import "tailwindcss";
```

### Verify

```bash
wc -c dist/output.css   # now 5-50 KB
```

## Example 9 : Custom Utility Class Detection

### Reported symptom

"I defined `@utility ring-glow-sm` in app.css but `ring-glow-sm` does
not work in markup."

### Diagnosis

```bash
grep -c "ring-glow-sm" dist/output.css   # 0
```

The custom utility IS defined in CSS but the scanner has not seen
`ring-glow-sm` in source (e.g. the class is in a third-party UI lib).

### Fix : add to inline safelist

```css
@import "tailwindcss";
@utility ring-glow-sm {
  box-shadow: 0 0 4px rgba(0,122,255,0.5);
}
@source inline("ring-glow-sm");
```

### Verify

```bash
grep -c "ring-glow-sm" dist/output.css   # > 0
```

## Example 10 : Watch Mode Misses Newly Created File

### Reported symptom

"Created `src/components/NewModal.jsx`. Classes inside don't show up
until I restart the dev server."

### Diagnosis

The watcher's directory list was cached at startup. A new file in an
existing directory is picked up ; a new file in a brand-new
sub-directory beyond the existing glob/scope may not be.

### Fix

Stop dev server (Ctrl-C). Restart :

```bash
npm run dev
```

### Prevention

ALWAYS use recursive globs : `./src/**/*` not `./src/components/*`.

## Example 11 : node_modules-shipped Package Without @source

### Reported symptom

"Using a Tailwind-styled component library from npm. Components
render with no styles."

### Diagnosis (v3)

```js
// tailwind.config.js
content: ["./src/**/*"]   // does NOT include node_modules
```

### Fix (v3)

```js
content: [
  "./src/**/*.{js,ts,jsx,tsx}",
  "./node_modules/@vendor/ui/**/*.{js,jsx,ts,tsx}",
],
```

### Diagnosis (v4)

Auto-detect SKIPS `node_modules` for performance.

### Fix (v4)

```css
@import "tailwindcss";
@source "../node_modules/@vendor/ui";
```

## Example 12 : Safelist for User-Generated Colors

### Reported symptom

"Users pick a brand color in a form. The color stored in DB is then
used as a Tailwind class at runtime, but the class never renders."

### Diagnosis

Class chosen at runtime cannot be statically detected. Tailwind
compiles at build time only.

### Fix (v3)

```js
safelist: [
  {
    pattern: /^(bg|text|border)-(red|orange|amber|yellow|lime|green|emerald|teal|cyan|sky|blue|indigo|violet|purple|fuchsia|pink|rose|slate|gray|zinc|neutral|stone)-(100|200|300|400|500|600|700|800|900)$/,
    variants: ["hover", "focus", "dark"],
  },
],
```

### Fix (v4)

```css
@source inline("{hover:,focus:,dark:,}{bg,text,border}-{red,orange,amber,yellow,lime,green,emerald,teal,cyan,sky,blue,indigo,violet,purple,fuchsia,pink,rose,slate,gray,zinc,neutral,stone}-{100..900..100}");
```

Both pre-generate the full palette. Output CSS gets bigger (~80 KB
extra in v3, less in v4 due to compression) ; accept the tradeoff
when classes are truly runtime-chosen.

## Example 13 : CI Build Smoke Test

A guard against detection regressions in CI :

```js
// scripts/verify-detection.js
const fs = require("fs");
const path = require("path");

const cssFile = path.resolve("dist/output.css");
const css = fs.readFileSync(cssFile, "utf8");

const requiredClasses = [
  "bg-blue-600",
  "hover:bg-blue-700",
  "rounded-md",
  "prose",
];

const missing = requiredClasses.filter(
  (c) => !css.includes(`.${c.replace(":", "\\:")}`)
);

if (missing.length) {
  console.error(`FAILED: ${missing.length} class(es) missing:`);
  missing.forEach((c) => console.error(`  - ${c}`));
  process.exit(1);
}

console.log(`OK: all ${requiredClasses.length} required classes present.`);
```

```yaml
# .github/workflows/verify.yml
- run: npm run build
- run: node scripts/verify-detection.js
```
