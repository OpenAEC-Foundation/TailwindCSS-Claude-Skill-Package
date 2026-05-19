# tailwind-errors-build-failures : Anti-Patterns

The traps that cause Tailwind builds to silently produce wrong output.

## Anti-Pattern 1 : String-interpolated class names

### Symptom

Class string is built at runtime ; scanner never sees the complete
token ; class is absent from compiled CSS.

### Wrong

```jsx
<button className={`bg-${color}-500 text-${size}`}>
```

```html
<div class="text-{{ color }}-600"></div>
```

### Right

```jsx
const colorMap = { red: "bg-red-500", blue: "bg-blue-500" };
<button className={colorMap[color]}>
```

### Root cause

Tailwind reads source files as text, extracting tokens that look like
complete class names. String interpolation produces FRAGMENTS that
never assemble. The classifier sees `bg-`, `-500` separately and
discards both. ALWAYS map prop values to complete class strings.

## Anti-Pattern 2 : Glob that doesn't include the file

### Symptom

A page renders unstyled even though the class is in source.

### Wrong (v3)

```js
content: ["./src/components/**/*.{js,jsx}"],
// page lives in src/pages/  -> never scanned
```

### Right

```js
content: ["./src/**/*.{js,jsx,ts,tsx}"],
```

### Root cause

Tailwind scans ONLY files matching the content array. A file outside
the glob does not exist for the compiler. ALWAYS keep the glob wide
enough to cover ALL directories with utility classes.

## Anti-Pattern 3 : Glob too broad (scans node_modules)

### Symptom

Build is slow (10+ seconds). Output CSS contains thousands of unused
utilities. Memory usage grows.

### Wrong

```js
content: ["./**/*.{html,js}"],     // includes node_modules
```

### Right

```js
content: [
  "./src/**/*.{html,js,jsx,ts,tsx}",
  "./public/**/*.html",
],
```

### Root cause

Recursive glob from project root picks up `node_modules/*`,
`.next/`, `dist/`, every vendor file. Class-name-like strings inside
vendor code inflate the output and slow scanning. ALWAYS scope to
your application directories.

## Anti-Pattern 4 : Believing the cache

### Symptom

Code change does NOT produce a corresponding CSS change. Old
selectors persist. Same after browser hard-refresh.

### Wrong

```bash
# Edit @theme tokens in app.css
# Save
# Reload browser
# See old output
# Conclude "Tailwind is broken"
```

### Right

```bash
rm -rf node_modules/.vite .next .astro
npm run dev
```

### Root cause

Vite, Next.js, Astro, Webpack all cache PostCSS output. After config
or @theme changes, the cache key may not invalidate. ALWAYS nuke the
relevant cache directory before reporting a Tailwind bug.

## Anti-Pattern 5 : Mixing v3 and v4 syntax

### Symptom

Build emits warnings + small CSS file. Or build emits empty CSS file.

### Wrong (v4 project, v3 entry)

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### Wrong (v3 project, v4 entry)

```css
@import "tailwindcss";
```

### Right

```
Tailwind v3 -> @tailwind base/components/utilities
Tailwind v4 -> @import "tailwindcss"
```

### Root cause

The two directive systems are incompatible. v3 ignores `@import "tailwindcss"`
(treats it as a literal URL import). v4 emits deprecation warnings for
`@tailwind` directives and partially honours them. ALWAYS match
directive to version. See `tailwind-impl-build-postcss-cli`.

## Anti-Pattern 6 : Forgetting to restart watcher after config change

### Symptom

You add a new content glob OR a new `@source` line. New file is still
not detected.

### Wrong

```js
// Edit tailwind.config.js
content: [..., "./new-dir/**/*"]   // saved
// Wait for HMR
// New-dir classes still missing
```

### Right

```bash
# Ctrl-C the watcher
# Restart
npm run dev
```

### Root cause

In v3, content array changes are picked up by the watcher in most
cases, but some bundler integrations (Vite, Next.js) cache the array
at startup. In v4, `@source` directives are read on watcher startup ;
new lines need a restart. ALWAYS restart after editing config.

## Anti-Pattern 7 : Safelist as a band-aid for dynamic classes

### Symptom

Safelist grows to thousands of entries. Output CSS is massive (200+
KB). The codebase has lots of `bg-${color}-${shade}` patterns.

### Wrong philosophy

```js
safelist: [
  // 300 patterns covering every possible color/shade combination
],
```

### Right philosophy

Fix the dynamic class strings to be static. Safelist only what is
TRULY runtime-chosen (user-stored data, third-party themes you don't
control). Static class lookups for component props.

### Root cause

Safelist generates classes regardless of usage. Used heavily, it
defeats Tailwind's main optimization (only ship what is used). ALWAYS
treat safelist as a LAST resort after refactoring source to be static.

## Anti-Pattern 8 : @source pointing at a non-existent path

### Symptom

```
warn: @source path does not exist: ../packages/ui/src
```

No error. Build continues. The expected files are NOT scanned.

### Wrong

```css
@source "../packages/ui/src";   /* but actual path is ../../packages/ui/src */
```

### Right

```css
@source "../../packages/ui/src";
```

### Root cause

`@source` paths resolve relative to the CSS file's location. Mis-
counting directory levels is common in monorepos. ALWAYS verify the
path exists from the perspective of the entry CSS.

```bash
# Sanity check
ls "$(dirname app.css)/../packages/ui/src"
```

## Anti-Pattern 9 : @source matching files but extensions excluded

### Symptom

```css
@source "../packages/ui";   /* dir exists, has .tsx files */
```

Yet classes from .tsx files are missing.

### Wrong

```css
@source "../packages/ui/dist";   /* only .js files in dist */
```

### Right

```css
@source "../packages/ui/src";    /* point at source .tsx */
```

### Root cause

The scanner reads file extensions for tokenisation. Compiled bundles
in `dist/` often have mangled class strings (minified, concatenated).
ALWAYS point @source at SOURCE files, not build output.

## Anti-Pattern 10 : Using `class` attribute in JSX

### Symptom

JSX file uses `class="bg-blue-500"` instead of `className`. Output
has the class in CSS but the element does NOT receive it.

### Wrong

```jsx
<div class="bg-blue-500">   /* class is the WRONG attribute in JSX */
```

### Right

```jsx
<div className="bg-blue-500">
```

### Root cause

This is NOT a Tailwind issue ; it is a React/JSX issue. The scanner
still detects `bg-blue-500` because it appears as a string. But the
DOM never receives it because `class` is invalid in JSX. ALWAYS use
`className` in React, `class` only in Vue/Svelte/HTML.

## Anti-Pattern 11 : Running v4 plugin in v3 PostCSS chain

### Symptom

```
Error: Cannot find module 'tailwindcss/postcss'
```

OR : build succeeds but output CSS contains the raw `@import
"tailwindcss";` line.

### Wrong

```js
// v3 project
module.exports = {
  plugins: { "@tailwindcss/postcss": {} },   // v4 plugin
};
```

### Right

```js
// v3 project
module.exports = {
  plugins: { tailwindcss: {}, autoprefixer: {} },
};
```

### Root cause

v4's PostCSS adapter is `@tailwindcss/postcss`. v3's adapter is bare
`tailwindcss`. Mixing them produces a chain that either errors or
silently passes raw CSS through. ALWAYS match plugin to Tailwind major.
See `tailwind-impl-build-postcss-cli`.

## Anti-Pattern 12 : Persistent watcher with infinite rebuild loop

### Symptom

Watcher rebuilds every 50ms. CPU at 100%. CSS file modification time
flickers constantly.

### Wrong

```js
content: ["./dist/**/*.{html,js}"],   // glob includes BUILD OUTPUT
```

The watcher writes `dist/output.css` -> dist changes -> watcher
triggers -> writes again -> infinite.

### Right

```js
content: ["./src/**/*.{html,js,jsx,ts,tsx}"],   // never include dist
```

### Root cause

Content globs that include the output destination create a feedback
loop. ALWAYS exclude `dist/`, `build/`, `out/`, `.next/`, `node_modules/`
from content scope.

## Anti-Pattern 13 : Pin tailwindcss but not @tailwindcss/vite

### Symptom

Astro v4.0.8 regression : pinned `tailwindcss: "4.0.7"` but issue
persists.

### Wrong

```json
{ "dependencies": { "tailwindcss": "4.0.7" } }
```

`@tailwindcss/vite` still resolves to `^4.0.0` and picks up 4.0.8.

### Right

```json
{
  "dependencies": {
    "tailwindcss": "4.0.7",
    "@tailwindcss/vite": "4.0.7"
  }
}
```

### Root cause

`@tailwindcss/vite`, `@tailwindcss/postcss`, `@tailwindcss/cli` are
SEPARATE npm packages with their own version ranges. Pin ALL Tailwind-
namespaced packages together when avoiding a specific release.

## Anti-Pattern 14 : Ignoring deprecation warnings

### Symptom

Build emits warnings ; developer ignores them ; production deploy
behaves differently from local.

### Right

ALWAYS read warnings during build. Tailwind v4 emits warnings for
EVERY deprecated v3 pattern (e.g. `@tailwind` directives, container-
queries plugin, aspect-ratio plugin). Each warning indicates an
upcoming break.

## Anti-Pattern 15 : Forgetting that Tailwind runs at BUILD time

### Symptom

Developer expects classes determined by an API response to render
correctly. The class names come from server state at request time.

### Wrong assumption

"If the class name is in the DOM at runtime, Tailwind will style it."

### Right

Tailwind compiles AT BUILD TIME. Classes not visible to the scanner
at build time are NOT in the CSS. Runtime-chosen classes require
either :

- A safelist that pre-generates them.
- Pre-known finite set (use static-lookup pattern).
- A different approach (CSS variables for dynamic values).

```jsx
// Anti-pattern : runtime class from API
<div className={apiResponse.color}>   /* not generated */

// Right : runtime VALUE via CSS variable
<div style={{ color: apiResponse.color }} className="text-[--my-color]">
```
