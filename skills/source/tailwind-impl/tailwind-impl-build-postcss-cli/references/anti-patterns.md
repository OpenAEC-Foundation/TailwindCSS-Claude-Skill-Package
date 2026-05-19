# tailwind-impl-build-postcss-cli : Anti-Patterns

Every trap, the exact symptom, the root cause, and the surgical fix.

## Anti-Pattern 1 : Wrong PostCSS plugin name (v4)

### Symptom

```
Error: Loading PostCSS Plugin failed: Cannot find module 'tailwindcss/postcss'
Error: Cannot find module '@tailwindcss/postcss'
```

Or, more subtle : build succeeds but output CSS contains the raw
`@import "tailwindcss";` line and no utility classes.

### Wrong (v4 project)

```js
// postcss.config.mjs
export default {
  plugins: { tailwindcss: {} },   // WRONG : this is v3 plugin name
};
```

### Right (v4 project)

```js
// postcss.config.mjs
export default {
  plugins: { "@tailwindcss/postcss": {} },
};
```

### Root cause

v4 ships the PostCSS adapter as a SEPARATE npm package named
`@tailwindcss/postcss`. The bare `tailwindcss` package in v4 contains
only the core library + Vite plugin, not a PostCSS-compatible export.

ALWAYS verify which version is installed before writing postcss.config :

```bash
npm list tailwindcss
```

## Anti-Pattern 2 : Adding autoprefixer in a v4 project

### Symptom

Properties get DOUBLE vendor prefixes (`-webkit--webkit-transform`),
container queries break, `@layer` ordering scrambles, or selectors
like `:where()` get incorrectly rewritten.

### Wrong (v4)

```js
// postcss.config.mjs
export default {
  plugins: {
    "@tailwindcss/postcss": {},
    autoprefixer: {},   // WRONG : Lightning CSS already does this
  },
};
```

### Right (v4)

```js
// postcss.config.mjs
export default {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
```

### Root cause

v4 embeds Lightning CSS, which handles vendor prefixing, modern-CSS
lowering (e.g. `oklch()` to `rgb()` fallback when configured), and
`@import` inlining. Layering another prefixer on top produces
double-prefixed declarations the browser cannot parse and silently
discards.

## Anti-Pattern 3 : Removing autoprefixer from a v3 project

### Symptom

Layouts break on Safari < 14, older Edge, older Firefox. `display: flex`,
`transform`, `position: sticky` lack vendor prefixes. Container queries
fail entirely.

### Wrong (v3)

```js
// postcss.config.js
module.exports = {
  plugins: {
    tailwindcss: {},
    // autoprefixer removed because "browsers are modern now"
  },
};
```

### Right (v3)

```js
// postcss.config.js
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

### Root cause

v3's PostCSS plugin does NOT prefix vendor properties. The
`autoprefixer` plugin is a hard requirement for cross-browser
correctness in v3. ALWAYS keep it. v4 is the version where prefixing
moved inline ; v3 has no such replacement.

## Anti-Pattern 4 : Using `@tailwind base/components/utilities` in v4

### Symptom

Build runs without error but emits warnings :

```
warn: @tailwind directives are deprecated in v4
```

The output CSS contains only some utility classes (or partial preflight)
because the directives no longer map 1:1 to v4's loading model.

### Wrong (v4 entry CSS)

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### Right (v4 entry CSS)

```css
@import "tailwindcss";
```

### Root cause

In v4 the entire framework loads from a single `@import` statement
which the PostCSS plugin or Vite plugin replaces with the framework
output at compile time. The three `@tailwind` directives were a v3
convention tied to the legacy build model.

## Anti-Pattern 5 : Using `@import "tailwindcss"` in v3

### Symptom

Build succeeds. Output CSS contains the literal `@import "tailwindcss";`
URL or completely empty CSS depending on PostCSS chain. No utility
classes ever appear in browser.

### Wrong (v3 entry CSS)

```css
@import "tailwindcss";
```

### Right (v3 entry CSS)

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### Root cause

v3 has no `@import "tailwindcss"` handler. PostCSS treats the line as
a real CSS `@import` rule and either leaves it (the browser then tries
to fetch a stylesheet at the URL `tailwindcss`) or strips it
(`postcss-import` strips because the path is not a real file).

## Anti-Pattern 6 : Forgetting `npx tailwindcss init` in v3

### Symptom

```
Specified content paths empty. Tailwind will not detect any classes.
warn: No utility classes were detected in your source files
```

Output CSS contains only `preflight` (reset) and no utilities.

### Wrong (v3)

```bash
npm install -D tailwindcss@3 postcss autoprefixer
# straight to building, no tailwind.config.js exists
npx tailwindcss -i input.css -o output.css
```

### Right (v3)

```bash
npm install -D tailwindcss@3 postcss autoprefixer
npx tailwindcss init    # creates tailwind.config.js
# edit content paths, then build
npx tailwindcss -i input.css -o output.css
```

### Root cause

v3 requires `tailwind.config.js` to know which files to scan for class
names. Without it, the `content` array is empty and JIT generates
nothing. v4 has no equivalent requirement because content detection
runs automatically and is augmented by `@source` directives in the CSS.

## Anti-Pattern 7 : Wrong CLI binary name (v3 vs v4 mix-up)

### Symptom

```
npm error 404 Not Found - GET https://registry.npmjs.org/@tailwindcss%2fcli - Not found
```

OR :

```
error: unknown option '--input'
```

### Wrong

```bash
# v3 project trying v4 binary name
npx @tailwindcss/cli -i input.css -o output.css

# v4 project trying v3 binary name
npx tailwindcss -i input.css -o output.css
```

### Right

```bash
# v3 project
npx tailwindcss -i input.css -o output.css

# v4 project
npx @tailwindcss/cli -i input.css -o output.css
```

### Root cause

v3 publishes the CLI as part of the bare `tailwindcss` package. v4
extracted it to `@tailwindcss/cli`. The npx invocation must match the
package that owns the binary in the version you installed.

## Anti-Pattern 8 : `--minify` in watch mode

### Symptom

Watcher recompiles slowly on every save. Browser source maps point to
mangled minified CSS. Debugging is painful.

### Wrong

```bash
npx @tailwindcss/cli -i input.css -o output.css --watch --minify
```

### Right

```bash
# development
npx @tailwindcss/cli -i input.css -o output.css --watch
# production (separate command, separate file)
npx @tailwindcss/cli -i input.css -o output.css --minify
```

### Root cause

`--minify` runs an extra optimisation pass (cssnano-equivalent in v3,
Lightning CSS `-Os` in v4) that adds latency to each rebuild and
removes whitespace/comments, both of which hurt the development
feedback loop. Use it ONLY for production.

## Anti-Pattern 9 : Watch mode misses newly-added directory

### Symptom

You add a new directory `app/admin/` with `.tsx` files and apply
Tailwind classes inside. The classes never appear in the compiled CSS.
Restarting the watcher fixes it once but the next new directory
repeats the problem.

### Root cause

In v3, content globs are resolved ONCE at watcher startup. New files
inside an already-matched glob trigger rebuild ; new directories
outside the glob's recursive scope do not.

### Fix (v3)

Use recursive globs in `content` :

```js
// tailwind.config.js
module.exports = {
  content: ["./app/**/*.{js,ts,jsx,tsx}"],   // ** captures any depth
};
```

### Fix (v4)

Use `@source` with a recursive scope :

```css
@import "tailwindcss";
@source "./app";   /* all files under app/ at any depth */
```

## Anti-Pattern 10 : Content glob too broad

### Symptom

Watcher startup takes 10+ seconds. Each rebuild lags. Memory usage
balloons. Build picks up class-name-like strings inside `node_modules`,
inflating output CSS with utilities you never use.

### Wrong (v3)

```js
module.exports = {
  content: ["./**/*.{html,js,ts,jsx,tsx}"],   // includes node_modules!
};
```

### Right (v3)

```js
module.exports = {
  content: [
    "./src/**/*.{js,ts,jsx,tsx}",
    "./public/**/*.html",
  ],
};
```

### Root cause

A top-level recursive glob includes `node_modules`, hidden
directories, build outputs, and vendor code. Always scope content
globs to your application's source directories. v4 mitigates this by
defaulting to `.gitignore`-aware scanning, but explicit `@source`
scoping is still recommended.

## Anti-Pattern 11 : PostCSS version mismatch

### Symptom

```
Module build failed: Error: PostCSS plugin tailwindcss requires PostCSS 8
```

OR :

```
TypeError: Cannot read properties of undefined (reading 'process')
```

### Root cause

Tailwind v3 and v4 both require PostCSS 8.x. A transitive dependency
pinning PostCSS 7 (older Vue CLI, some Storybook setups, old
Laravel Mix) breaks the plugin chain.

### Fix

Upgrade PostCSS in the lockfile :

```bash
npm install postcss@^8 --force
# or for yarn
yarn add postcss@^8
```

If a tool that depends on PostCSS 7 is in the chain, upgrade the tool
or switch to the standalone CLI to bypass PostCSS entirely.

## Anti-Pattern 12 : Using standalone binary without pinning version

### Symptom

CI builds suddenly fail or produce different output after upstream
cuts a release. Locally everything works because the cached binary is
older.

### Wrong

```bash
curl -sLO https://github.com/tailwindlabs/tailwindcss/releases/latest/download/tailwindcss-linux-x64
```

### Right

```bash
curl -sLO https://github.com/tailwindlabs/tailwindcss/releases/download/v4.0.0/tailwindcss-linux-x64
```

### Root cause

`latest` is a moving target. Pin to a specific tag (`v4.0.0`,
`v3.4.0`) and update deliberately. Treat the binary like any other
dependency : it has a version, lock it.

## Anti-Pattern 13 : Mixing v3 and v4 in the same project

### Symptom

Some files use `@tailwind base/components/utilities`, others use
`@import "tailwindcss"`. PostCSS config lists BOTH `tailwindcss` and
`@tailwindcss/postcss`. Output is unpredictable.

### Wrong (postcss.config)

```js
module.exports = {
  plugins: {
    tailwindcss: {},               // v3
    "@tailwindcss/postcss": {},    // v4
  },
};
```

### Root cause

The two versions are not designed to coexist. Pick ONE per project.
For gradual migration, use `tailwind-impl-migration-v3-v4` to move
in a structured way ; the recommended bridge is `@config "./tailwind.config.js"`
inside a v4 entry CSS so the JS config still applies during transition.

## Anti-Pattern 14 : Forgetting to import compiled CSS in HTML/JS

### Symptom

CSS file builds fine, contains correct utilities, but page shows
unstyled content.

### Wrong (no link in HTML, no import in JS entry)

```html
<!doctype html>
<html>
  <head></head>
  <body><h1 class="text-3xl font-bold">Hello</h1></body>
</html>
```

### Right (HTML)

```html
<link rel="stylesheet" href="./dist/output.css" />
```

### Right (JS bundler)

```js
// src/main.ts
import "./styles/app.css";
```

### Root cause

The CLI/PostCSS step produces a static file. Nothing automatically
loads it into the page. ALWAYS link or import the compiled output
where it should take effect.

## Anti-Pattern 15 : Output path inside a directory the framework ignores

### Symptom

Build succeeds. File appears on disk. Browser receives a 404 for the
stylesheet URL.

### Common case (Next.js)

```bash
npx tailwindcss -i input.css -o ./pages/styles.css
```

Next.js does not serve files from `pages/` as static assets. The build
silently fails to surface them.

### Right

```bash
# Next.js
npx tailwindcss -i input.css -o ./public/styles.css

# Or let Next.js handle the chain via PostCSS plugin (preferred)
# In that case no CLI step at all.
```

### Root cause

Each framework has conventions about where build artefacts live.
Output to a directory that the framework EXPOSES (`public/`, `static/`,
`dist/`, `priv/static/`, `app/assets/builds/`). When unsure, route the
build through the framework's own pipeline (Vite, PostCSS).
