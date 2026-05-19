# tailwind-errors-build-failures : Methods Reference

Complete control surface for source detection, safelisting, and cache
invalidation.

## Section 1 : v3 content Array Reference

### Top-level shape (array)

```js
// tailwind.config.js
module.exports = {
  content: [
    "./src/**/*.{html,js,jsx,ts,tsx}",
    "./public/index.html",
  ],
};
```

### Top-level shape (object with options)

```js
module.exports = {
  content: {
    files: ["./src/**/*.{html,js,jsx,ts,tsx,md,mdx}"],
    relative: true,                            // resolve relative to config
    transform: {
      mdx: (raw) => raw.replace(/\{.*?\}/g, ""), // strip JSX expressions
    },
    extract: {
      wtf: (raw) => raw.match(/[^<>"'`\s]*/g),   // custom token regex
    },
  },
};
```

### Glob syntax

| Pattern | Matches |
|---------|---------|
| `*` | any chars EXCEPT `/` and hidden files |
| `**` | zero or more directory levels |
| `{a,b,c}` | one of a, b, c |
| `?` | exactly one char |
| `[abc]` | one of a, b, c |
| `!pattern` | negation (inside files array) |

### safelist

```js
safelist: [
  // literal class
  "bg-red-500",

  // pattern object
  {
    pattern: /bg-(red|green|blue)-(100|200|300|400|500)/,
    variants: ["hover", "focus", "lg"],
  },
],
```

Patterns match BASE class names. Variants are an OPT-IN array.

### blocklist

```js
blocklist: ["container", "collapse"],   // strings only, no regex
```

## Section 2 : v4 @source Directive Reference

### Full syntax

```css
@import "tailwindcss";

/* Add a path */
@source "../node_modules/@my-org/ui";

/* Exclude a path */
@source not "../src/legacy";

/* Inline safelist */
@source inline("underline");
@source inline("{hover:,focus:,}underline");
@source inline("bg-red-{50,{100..900..100},950}");

/* Inline anti-safelist */
@source not inline("bg-red-{50,{100..900..100},950}");

/* Replace base path */
@import "tailwindcss" source("../src");

/* Disable auto-detection */
@import "tailwindcss" source(none);
@source "../src/components";
```

### Brace expansion syntax

| Pattern | Expands to |
|---------|------------|
| `{a,b,c}` | a, b, c |
| `{1..5}` | 1, 2, 3, 4, 5 |
| `{100..500..100}` | 100, 200, 300, 400, 500 |
| `{hover:,focus:,}underline` | hover:underline, focus:underline, underline |

### Path resolution

- Relative paths resolve from the CSS file's location.
- ABSOLUTE paths (`/abs/path`) are supported but discouraged.
- Globs work : `@source "../packages/*/src"`.

### Default ignored paths

- `.gitignore` matches
- `node_modules/` (unless explicitly @source'd)
- `*.css` files
- Binary files (images, fonts, videos, archives)
- Lock files (`package-lock.json`, `yarn.lock`, etc.)

## Section 3 : Inspecting What Gets Detected

### v3 : run with `--verbose`

```bash
DEBUG=tailwindcss:* npx tailwindcss -i input.css -o output.css
```

Logs every file the scanner reads.

### v4 : check output size

```bash
npx @tailwindcss/cli -i input.css -o output.css --minify
wc -c output.css
```

A v4 build of a real application should be 5-50 KB minified. If
output is `<2 KB` something is wrong.

### Grep the output

```bash
grep -c "bg-blue-500" dist/output.css
# 0 = class not detected
# >0 = class detected and emitted
```

## Section 4 : Cache Invalidation Commands

### Vite

```bash
rm -rf node_modules/.vite
npm run dev
```

### Next.js

```bash
rm -rf .next
npm run dev
```

For Next.js with persistent cache :

```bash
rm -rf .next node_modules/.cache
```

### Astro

```bash
rm -rf .astro dist node_modules/.vite
npm run dev
```

### Webpack (custom config)

```bash
rm -rf node_modules/.cache
# or set in webpack.config.js : cache: false (dev only)
```

### Turbopack

Turbopack has no documented cache-clear command. Restart with
`--turbo` after deleting `.next` :

```bash
rm -rf .next
next dev --turbo
```

### Standalone CLI

No cache. Restart kills any state.

## Section 5 : Common @source Patterns

### Monorepo (turborepo, nx, lerna)

```css
@import "tailwindcss";

/* All apps and packages under the root */
@source "../../apps/*/src";
@source "../../packages/*/src";

/* node_modules-shipped UI lib */
@source "../../node_modules/@my-org/ui/dist";
```

### Pin auto-detection root

```css
@import "tailwindcss" source("../src");
```

Equivalent to setting cwd before `tailwindcss` runs.

### Explicit-only mode

```css
@import "tailwindcss" source(none);
@source "./src/components";
@source "./src/pages";
@source "./src/layouts";
```

ALWAYS pair with explicit `@source` lines.

## Section 6 : Safelist Recipes

### v3 : all utility variants of one base

```js
safelist: [
  {
    pattern: /^bg-blue-\d+$/,
    variants: ["hover", "focus", "active", "lg", "dark"],
  },
],
```

### v3 : a full palette + numeric scale

```js
safelist: [
  {
    pattern: /^(bg|text|border|ring)-(red|green|blue|amber|emerald)-(100|200|300|400|500|600|700|800|900)$/,
    variants: ["hover", "focus", "dark"],
  },
],
```

### v4 : same palette via inline

```css
@source inline("{hover:,focus:,dark:,}{bg,text,border,ring}-{red,green,blue,amber,emerald}-{100..900..100}");
```

ONE line replaces a v3 safelist regex.

### v4 : safelist column-span 1 through 12

```css
@source inline("{sm:,md:,lg:,}col-span-{1..12}");
```

### v4 : exclude classes you don't want

```css
@source not inline("bg-amber-{100..900..100}");
```

## Section 7 : Diagnostic Commands

### Count classes in output

```bash
grep -oE '\.[a-z0-9_-]+' dist/output.css | sort -u | wc -l
```

A real project usually has 200-2000 unique selectors. Below 50 means
the scanner missed most of your source.

### List all class groups present

```bash
grep -oE '^\.[a-z]+' dist/output.css | sort -u | head -30
```

Useful for "is the typography plugin emitting prose-* selectors ?".

### Find which file mentions a class

```bash
grep -rn "bg-emerald-600" src/
```

If grep finds the string but compiled CSS does NOT have the class,
detection failed despite the file being scanned.

## Section 8 : Forcing a Rebuild From Scratch

### Aggressive nuke (kills all caches and lock state)

```bash
# Stop dev server first (Ctrl-C)
rm -rf node_modules .next .astro .vite dist out node_modules/.vite node_modules/.cache
rm -f package-lock.json yarn.lock pnpm-lock.yaml
npm install
npm run dev
```

ALWAYS use this as a LAST resort. Faster diagnoses are in Section 4.

## Section 9 : Watcher Restart Triggers

The watcher MUST be restarted when :

- You add a NEW directory not covered by content/@source.
- You change `tailwind.config.js` (v3) or the entry CSS @plugin / @source / @theme.
- You change a `node_modules` plugin (after `npm install` of new
  Tailwind plugin).
- HMR delivers stale CSS twice in a row.

The watcher does NOT need restart for :

- New file in a directory already covered by glob/source.
- New class in a file already covered.
- Edits to `tailwind.config.js` content array (v3 auto-restarts).

## Section 10 : Programmatic Verification

A unit test that confirms a critical class survives detection :

```js
// scripts/verify-class.js
const fs = require("fs");
const css = fs.readFileSync("dist/output.css", "utf8");

const required = ["bg-blue-600", "hover:bg-blue-700", "rounded-md"];
const missing = required.filter((c) => !css.includes(`.${c}`));

if (missing.length) {
  console.error("Missing required classes:", missing);
  process.exit(1);
}
console.log("All required classes present.");
```

Run as a CI step after build : fast smoke test against detection
regressions.
