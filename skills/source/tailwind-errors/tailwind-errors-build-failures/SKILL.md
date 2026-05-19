---
name: tailwind-errors-build-failures
description: >
  Use when a Tailwind build completes successfully but the compiled
  CSS is empty, partial, or stale, when classes appear in markup but
  styles never render in the browser, when a freshly added utility
  refuses to take effect, when an HMR update leaves the page styled
  with the previous build, or when a monorepo package's components
  ship un-styled because the scanner does not look at their files.
  Prevents the dynamic-class-name trap (string interpolation breaks
  detection in BOTH v3 content scanner and v4 Oxide scanner, the most
  common reason for "Tailwind is broken"), the too-narrow content
  glob (v3 content array misses new directories), the cached-build
  trap (Vite's node_modules/.vite cache + Next.js .next cache hold
  stale class output), the Turbopack arbitrary-value miss (Next.js
  16.x + Turbopack incremental build misses aspect-[12/5], z-[100]),
  the Astro v4.0.8 regression (pin to v4.0.7), the monorepo-import
  trap (importing a UI lib from node_modules whose classes the
  scanner never sees), and the v3-v4 directive-mix trap (using
  @tailwind base in v4 OR @import "tailwindcss" in v3 yields empty
  output).
  Covers v3 content array + safelist + blocklist + transform + extract,
  v4 auto-detect + @source + @source not + @source inline + @source none,
  brace-expansion safelist syntax, monorepo absolute-path patterns,
  Vite cache invalidation (rm -rf node_modules/.vite), Next.js cache
  invalidation (rm -rf .next), Turbopack workarounds, Astro version
  pinning, watch-mode-misses-new-directory restart procedure, and the
  static-class-name discipline that prevents 90% of all build failures.
  Keywords: tailwind build fails, no utility classes generated, empty
  output css, tailwind classes not applied, styles missing in build,
  jit not detecting class, content not scanned, @source not working,
  v4 source detection, v3 content array, safelist, blocklist,
  dynamic class names, string interpolation tailwind, monorepo
  tailwind, ui library not styled, node_modules tailwind, vite cache
  stale, node_modules/.vite, next cache, .next cache, turbopack
  arbitrary value, aspect-[12/5] not generated, z-[100] missing,
  next.js 16 turbopack, astro 4.0.8 broken, tailwind 4.0.7 pin,
  @tailwindcss/vite regression, @import tailwindcss missing utilities,
  watch mode not detecting new file, why is my tailwind class missing,
  my class is in html but no style, my padding class disappeared,
  tailwind not rebuilding, hot reload broken tailwind, brace expansion
  safelist, bg-red-{50,100,200,300}, force generate tailwind class,
  arbitrary values jit, monorepo @source absolute path, restart
  tailwind watcher
license: MIT
compatibility: "Designed for Claude Code. Requires Tailwind CSS v3.4 or v4.0+."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# Tailwind Build Failures : Why Your Classes Don't Render

The Tailwind compiler emits ONLY the utility classes it can DETECT in
your source files. Every "Tailwind is broken" report is one of :

1. The compiler did not see the file containing the class.
2. The compiler saw the file but could not parse the class as a
   complete unbroken string.
3. The class was detected, compiled, but a stale build cache served
   yesterday's CSS to the browser.
4. A version-specific regression in v4 or Turbopack swallowed certain
   class patterns silently.

This skill is the diagnostic flow for all four. Always go top to
bottom : root-cause-first.

Companion skills :

- `tailwind-impl-config-v3` : the `content` array surface
- `tailwind-impl-config-v4` : the `@source` directive surface
- `tailwind-impl-build-vite` : Vite-specific cache behaviour
- `tailwind-impl-build-postcss-cli` : PostCSS / CLI flow
- `tailwind-errors-runtime` : when classes ARE in CSS but not applied
- `tailwind-errors-v4-migration` : v3-to-v4 specific breakage

## Quick Reference : 60-Second Diagnostic

```
1. Inspect compiled CSS (open the file the browser loads).
   - File is EMPTY                   -> scanner found nothing
   - File has SOME classes, missing X -> X was not detected
   - File looks right but browser shows old styles -> cache stale
   - File is missing a recently added class -> watcher missed it

2. Search compiled CSS for the missing class :
   grep "p-4" dist/output.css

3. If absent : the SCANNER missed it. Go to "Scanner Diagnosis".
   If present : the BUILD is stale OR another rule wins. Go to
   "Cache Diagnosis" + tailwind-errors-runtime.
```

## Decision Tree : Scanner Missed the Class

```
Is the class an UNBROKEN STRING in source ?
  ├── NO  (string interpolation: `bg-${color}-500`)
  │   -> FIX: replace with static lookup (see "Static-Class Discipline")
  │
  └── YES
      ├── v3 : Is the file inside `content` globs ?
      │   ├── NO  -> add the glob OR widen existing one
      │   └── YES -> check next branch
      │
      └── v4 : Is the file inside auto-detect scope OR explicit @source ?
          ├── NO (file in node_modules, monorepo sibling, etc.)
          │   -> Add @source "../path/to/file"
          └── YES
              ├── Turbopack ? -> see "Turbopack Workaround"
              ├── Astro v4.0.8 ? -> pin to v4.0.7
              └── Otherwise -> restart watcher (new-directory case)
```

## Decision Tree : Stale Cache

```
Did the class appear in CSS yesterday but not today after a code change ?

Vite project ?
  -> rm -rf node_modules/.vite ; npm run dev

Next.js project ?
  -> rm -rf .next ; npm run dev

Webpack project ?
  -> Disable persistent cache OR rm -rf node_modules/.cache

Astro project ?
  -> rm -rf .astro dist node_modules/.vite ; npm run dev

CLI watch mode ?
  -> Ctrl-C, restart npx (@tailwindcss/cli OR tailwindcss)
```

## CRITICAL : Static-Class Discipline

90% of "Tailwind is broken" reports trace to ONE pattern.

### Wrong (works in browser console, fails in build)

```jsx
function Button({ color, size }) {
  return <button className={`bg-${color}-600 text-${size}`}>...</button>;
}
```

At compile time, Tailwind sees the strings `bg-`, `-600`, `text-` but
NEVER the complete `bg-blue-600`. The class never enters compiled CSS.

### Right (complete class strings)

```jsx
function Button({ color, size }: { color: "blue" | "red"; size: "sm" | "lg" }) {
  const colorClasses = {
    blue: "bg-blue-600 hover:bg-blue-700",
    red: "bg-red-600 hover:bg-red-700",
  };
  const sizeClasses = {
    sm: "text-sm",
    lg: "text-lg",
  };
  return <button className={`${colorClasses[color]} ${sizeClasses[size]}`}>...</button>;
}
```

Every class appears as a complete unbroken string somewhere in source.
The scanner finds them.

### Right (safelist when dynamic is unavoidable)

v3 :

```js
// tailwind.config.js
module.exports = {
  safelist: [
    "bg-red-500",
    {
      pattern: /bg-(red|green|blue)-(100|200|300|400|500)/,
      variants: ["hover", "focus"],
    },
  ],
};
```

v4 :

```css
@import "tailwindcss";
@source inline("{hover:,focus:,}bg-{red,green,blue}-{100,200,300,400,500}");
```

The brace expansion in `@source inline` is the v4 equivalent of
v3's safelist `pattern`.

## v3 Content Array : The Survival Kit

### Minimum

```js
// tailwind.config.js
module.exports = {
  content: ["./src/**/*.{html,js,jsx,ts,tsx,vue,svelte}"],
};
```

### Common mistakes (and fixes)

| Symptom | Cause | Fix |
|---------|-------|-----|
| New `app/admin/` directory unstyled | Pre-existing glob `./src/**` does not include `app/` | Add `"./app/**/*.{js,ts,jsx,tsx}"` |
| Classes from a CMS-rendered HTML missing | HTML files not in content | Add `"./public/**/*.html"` or wherever HTML lives |
| Classes in MDX files missing | `.mdx` not in extension list | Append `mdx` : `"./content/**/*.{md,mdx}"` |
| Tailwind picks up `node_modules` (build slow) | Glob too broad (`./**/*`) | Scope to specific dirs |
| Classes from imported `@my-org/ui` package missing | The package's source not in content | Add `"./node_modules/@my-org/ui/**/*.{js,jsx,ts,tsx}"` |

### Safelist for truly-dynamic strings

```js
safelist: [
  "bg-red-500", "bg-green-500", "bg-blue-500",
  { pattern: /^col-span-\d+$/, variants: ["sm", "md", "lg"] },
],
```

### Blocklist to suppress accidental classes

```js
blocklist: ["container"],   // strip 'container' if you defined your own
```

## v4 @source Directive : The Survival Kit

### Default auto-detection scope

- All files in current working directory, recursively.
- IGNORED : files matched by `.gitignore`, `node_modules`, binary
  files, CSS files, package-manager lock files.

### Adding a monorepo sibling

```css
@import "tailwindcss";
@source "../packages/ui/src";
@source "../packages/marketing/src";
```

### Adding a node_modules package

```css
@import "tailwindcss";
@source "../../node_modules/@my-org/ui";
```

### Setting a different base path

```css
@import "tailwindcss" source("../src");
```

### Excluding a path

```css
@source not "../src/legacy";
```

### Disabling auto-detection entirely

```css
@import "tailwindcss" source(none);
@source "../src/components";
@source "../src/pages";
```

Use this for monorepos where you want EXPLICIT control.

### Inline safelist with brace expansion

```css
@source inline("underline");
@source inline("{hover:,focus:,}underline");
@source inline("{hover:,}bg-red-{50,{100..900..100},950}");
```

The `{n..m..step}` range is a v4 feature. Pre-generates the entire
palette for dynamic class needs.

## CRITICAL : Stale Build Cache

Tailwind itself is fast and deterministic. The host bundler's cache
is what goes stale.

### Vite + Tailwind v4

The Vite dependency-cache lives in `node_modules/.vite`. When you
edit `tailwind.config.js` or `app.css`, Vite sometimes serves cached
PostCSS output. Symptom : you delete `bg-red-500` from source, it
still appears in the browser.

```bash
rm -rf node_modules/.vite
npm run dev
```

### Next.js (App Router + Pages Router)

Next.js caches PostCSS output in `.next/`. After upgrading Tailwind
or switching v3-v4, force a fresh build :

```bash
rm -rf .next
npm run dev
```

### Astro

Astro layers its own `.astro` cache on top of Vite's `.vite` cache :

```bash
rm -rf .astro dist node_modules/.vite
npm run dev
```

### Standalone CLI watch

Kill the watcher (Ctrl-C) and restart. The watcher's chokidar (v3) or
Oxide (v4) caches file lists at startup ; new-directory additions
need a restart.

## Known Version-Specific Regressions

### Turbopack arbitrary-value miss (Next.js 16.x + Tailwind v4.2.1)

Per issue 19825 : `aspect-[12/5]`, `z-[100]`, and similar arbitrary-value
classes are emitted in markup but NEVER reach compiled CSS when using
Next.js 16.1.6 with `--turbo`. Affects files that import certain
third-party libraries (e.g. Swiper.js).

Workarounds :

```bash
# Option 1 : disable Turbopack (use Webpack)
next dev    # no --turbo
```

```jsx
// Option 2 : inline style instead of arbitrary value
<div style={{ aspectRatio: "12/5" }}>
<div style={{ zIndex: 100 }}>
```

```jsx
// Option 3 : add the arbitrary class to @source inline so it
//           is always generated regardless of detection
```

```css
@source inline("aspect-[12/5] z-[100]");
```

### Astro v4.0.8 regression (issue 16733)

Per issue 16733 : Tailwind v4.0.8 with `@tailwindcss/vite` v4.0.8 and
Astro v5.3.0 silently drops styles from imported component packages.

Workaround : pin to v4.0.7 until upstream resolves.

```json
{
  "dependencies": {
    "tailwindcss": "4.0.7",
    "@tailwindcss/vite": "4.0.7"
  }
}
```

Lock both packages in `package-lock.json` / `yarn.lock` to prevent
auto-bump.

## CRITICAL : v3-v4 Directive Mix-Up

| Project | Entry CSS line | Result |
|---------|----------------|--------|
| v3 | `@import "tailwindcss";` | Empty CSS, browser fetches URL |
| v3 | `@tailwind base; @tailwind components; @tailwind utilities;` | Works |
| v4 | `@tailwind base; ...` | Deprecation warnings + partial output |
| v4 | `@import "tailwindcss";` | Works |

Detection : if the compiled CSS is `<2 KB`, the directive line is
probably wrong.

## CRITICAL : Watcher Misses New Directory

```bash
# v3
mkdir src/admin
# Watcher does NOT pick up files in src/admin if content glob is
# ./src/components/**/*  rather than  ./src/**/*

# v4
mkdir packages/widget
# Auto-detect picks it up if it's in cwd ; if it's a sibling outside
# cwd, ADD @source "../widget".
```

Fix : restart watcher AND widen content/@source scope.

## What This Skill Does NOT Cover

- Classes ARE in CSS but browser shows wrong styles -> see
  `tailwind-errors-runtime` (specificity, cascade, scope).
- v3-to-v4 migration breakage -> see `tailwind-errors-v4-migration`.
- Forms / plugin classes missing -> see
  `tailwind-impl-plugins-official` § "Verifying a Plugin Loaded".
- `cn`/`twMerge` returns unexpected output -> see
  `tailwind-impl-tailwind-merge`.
- HMR not updating React state (not a Tailwind issue) -> see the
  bundler's docs.

## References

- `references/methods.md` : every config option, every directive,
  every cache-invalidation command.
- `references/examples.md` : full diagnostic walk-throughs for the
  classic failure modes.
- `references/anti-patterns.md` : the traps catalogued by symptom.

## Official Source Links

- v4 source detection : https://tailwindcss.com/docs/detecting-classes-in-source-files
- v3 content : https://v3.tailwindcss.com/docs/content-configuration
- Turbopack issue 19825 : https://github.com/tailwindlabs/tailwindcss/issues/19825
- Astro v4.0.8 issue 16733 : https://github.com/tailwindlabs/tailwindcss/issues/16733
