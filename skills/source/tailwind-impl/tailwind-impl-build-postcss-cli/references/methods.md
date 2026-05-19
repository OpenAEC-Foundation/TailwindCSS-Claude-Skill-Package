# tailwind-impl-build-postcss-cli : Methods Reference

Complete API surface for the PostCSS plugin and the standalone CLI,
for both Tailwind v3 and v4.

## Section 1 : Package Names

| Tailwind version | npm package(s) | PostCSS plugin name | CLI binary name |
|------------------|----------------|---------------------|-----------------|
| v3.x | `tailwindcss@3`, `postcss`, `autoprefixer` | `tailwindcss` | `npx tailwindcss` |
| v4.x | `tailwindcss`, `@tailwindcss/postcss` (for PostCSS), `@tailwindcss/cli` (for CLI) | `@tailwindcss/postcss` | `npx @tailwindcss/cli` |

ALWAYS verify which version you are targeting BEFORE installing. The
v3 and v4 plugin names are not interchangeable.

## Section 2 : Install Commands

### v4 PostCSS

```bash
npm install tailwindcss @tailwindcss/postcss postcss
```

`postcss` is a peer dependency. `autoprefixer` and `postcss-import` are
NOT needed because Lightning CSS handles both internally.

### v4 CLI

```bash
npm install tailwindcss @tailwindcss/cli
```

`postcss` is NOT needed for the CLI path.

### v3 PostCSS

```bash
npm install -D tailwindcss@3 postcss autoprefixer
npx tailwindcss init
```

The `init` command creates a default `tailwind.config.js`. Pass
`--full` for a config file pre-populated with all defaults, or `-p` to
also create a default `postcss.config.js`.

### v3 CLI

```bash
npm install -D tailwindcss@3
npx tailwindcss init
```

## Section 3 : postcss.config Files

### v4 (postcss.config.mjs, ESM)

```js
export default {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
```

### v4 (postcss.config.cjs, CommonJS)

```js
module.exports = {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
```

### v3 (postcss.config.js)

```js
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

### v3 with extra plugins (full pipeline)

```js
module.exports = {
  plugins: {
    "postcss-import": {},
    "tailwindcss/nesting": {},
    tailwindcss: {},
    autoprefixer: {},
    ...(process.env.NODE_ENV === "production" ? { cssnano: {} } : {}),
  },
};
```

## Section 4 : CSS Entry Points

### v4

```css
/* src/app.css */
@import "tailwindcss";
```

Partial imports for advanced cases :

```css
@import "tailwindcss/preflight";   /* reset only */
@import "tailwindcss/utilities";   /* utilities only */
@import "tailwindcss/theme";       /* theme variables only */
```

### v3

```css
/* src/app.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Order matters : `base` then `components` then `utilities` so the
cascade resolves utility-overrides-component correctly.

## Section 5 : CLI Flags

### Full flag table

| Long | Short | Argument | Default | Purpose |
|------|-------|----------|---------|---------|
| `--input` | `-i` | path | (none, stdin) | Path to source CSS file |
| `--output` | `-o` | path | (stdout) | Path to compiled CSS file |
| `--watch` | `-w` | none | off | Recompile on file change |
| `--minify` | `-m` | none | off | Compress output for production |
| `--config` | `-c` | path | `./tailwind.config.js` (v3) | Path to JS config (v3) ; v4 ignores, use `@config` directive |
| `--cwd` | none | path | `process.cwd()` (v3) | Override scan root (v3 only) |
| `--postcss` | none | path or `false` | auto | Path to postcss.config.js (v3 only) |
| `--content` | none | comma-list of globs | (none) | Override content paths (v3 only) |
| `--no-autoprefixer` | none | none | (v3 default-on) | Disable autoprefixer in pipeline (v3 only) |
| `--help` | `-h` | none | n/a | Print usage and exit |

### v4 example invocations

Build once (production) :

```bash
npx @tailwindcss/cli -i ./src/input.css -o ./dist/output.css --minify
```

Watch (development) :

```bash
npx @tailwindcss/cli -i ./src/input.css -o ./dist/output.css --watch
```

Stdout to a piped consumer :

```bash
npx @tailwindcss/cli -i ./src/input.css --minify > ./dist/output.css
```

### v3 example invocations

Build once :

```bash
npx tailwindcss -i ./src/input.css -o ./dist/output.css --minify
```

Watch :

```bash
npx tailwindcss -i ./src/input.css -o ./dist/output.css --watch
```

Custom config path :

```bash
npx tailwindcss -i ./src/input.css -o ./dist/output.css -c ./config/tailwind.cjs --minify
```

Content paths from CLI (no config file) :

```bash
npx tailwindcss --content './src/**/*.{html,js,ts,jsx,tsx}' -i ./src/input.css -o ./dist/output.css
```

## Section 6 : Standalone Binary (no Node)

The standalone executable is published per platform on the GitHub
releases page : https://github.com/tailwindlabs/tailwindcss/releases

### Asset naming

| Platform | Asset name pattern |
|----------|---------------------|
| Linux x64 (glibc) | `tailwindcss-linux-x64` |
| Linux arm64 | `tailwindcss-linux-arm64` |
| macOS Intel | `tailwindcss-macos-x64` |
| macOS Apple Silicon | `tailwindcss-macos-arm64` |
| Windows x64 | `tailwindcss-windows-x64.exe` |

v4 names the asset `tailwindcss-<platform>-<arch>` ; v3 names match.

### Download + install (Linux/macOS)

```bash
curl -sLO https://github.com/tailwindlabs/tailwindcss/releases/latest/download/tailwindcss-linux-x64
chmod +x tailwindcss-linux-x64
mv tailwindcss-linux-x64 ./bin/tailwindcss
./bin/tailwindcss -i input.css -o output.css --watch
```

### Download + install (Windows PowerShell)

```powershell
Invoke-WebRequest -Uri https://github.com/tailwindlabs/tailwindcss/releases/latest/download/tailwindcss-windows-x64.exe -OutFile tailwindcss.exe
.\tailwindcss.exe -i input.css -o output.css --watch
```

### Pinning a specific version

```bash
curl -sLO https://github.com/tailwindlabs/tailwindcss/releases/download/v4.0.0/tailwindcss-linux-x64
```

ALWAYS pin a version in production. The `latest` redirect can change
overnight when upstream cuts a release.

## Section 7 : tailwind.config.js Shape (v3 only)

Minimal :

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./src/**/*.{html,js,jsx,ts,tsx,vue,svelte}"],
  theme: { extend: {} },
  plugins: [],
};
```

Common extensions :

```js
module.exports = {
  content: [
    "./src/**/*.{html,js,jsx,ts,tsx}",
    "./components/**/*.{js,jsx,ts,tsx}",
    "./public/index.html",
  ],
  darkMode: "class",        // or "media"
  theme: {
    extend: {
      colors: { brand: "#0070f3" },
      fontFamily: { sans: ["Inter", "sans-serif"] },
    },
  },
  plugins: [
    require("@tailwindcss/typography"),
    require("@tailwindcss/forms"),
  ],
};
```

v4 has NO `tailwind.config.js` by default. Use the v4 CSS-first
directives instead (`@theme`, `@source`, `@plugin`, `@variant`). To
load a legacy v3 config in v4, add `@config "./tailwind.config.js"` to
the entry CSS.

## Section 8 : Framework Integration Snippets

### Next.js (v4 PostCSS)

```js
// postcss.config.mjs
export default { plugins: { "@tailwindcss/postcss": {} } };
```

```css
/* app/globals.css */
@import "tailwindcss";
```

```tsx
// app/layout.tsx
import "./globals.css";
```

### Next.js (v3 PostCSS)

```js
// postcss.config.js
module.exports = { plugins: { tailwindcss: {}, autoprefixer: {} } };
```

```css
/* styles/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### Remix (v4)

```js
// postcss.config.cjs (Remix uses CJS by default)
module.exports = { plugins: { "@tailwindcss/postcss": {} } };
```

### Astro (v4)

Use the Vite plugin, not PostCSS :

```js
// astro.config.mjs
import { defineConfig } from "astro/config";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  vite: { plugins: [tailwindcss()] },
});
```

### Astro (v3)

```js
// astro.config.mjs
import tailwind from "@astrojs/tailwind";
export default defineConfig({ integrations: [tailwind()] });
```

### Webpack (any version)

Add `postcss-loader` to your loader chain. Tailwind itself is
declared in `postcss.config.js`, not in webpack config.

```js
// webpack.config.js (excerpt)
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader", "postcss-loader"],
      },
    ],
  },
};
```

## Section 9 : Watch-Mode Internals

v4 watch uses Oxide's Rust file-watching layer. v3 uses chokidar.

What both detect :
- Changes to the input CSS file.
- Changes to files matched by content globs (v3) or @source scopes (v4).
- Changes to `tailwind.config.js` (v3 only ; restart not needed).
- Changes to any `@import`-ed CSS partial.

What neither detects :
- New files in directories NOT covered by content/source scope. Restart
  the watcher when adding such directories.
- Changes to `node_modules` content (ignored by default for performance).
- Changes to files mentioned in a `safelist` only at the regex level.

## Section 10 : Programmatic API (v3 only)

v3 exposes a Node API for advanced use :

```js
const tailwindcss = require("tailwindcss");
const postcss = require("postcss");
const fs = require("fs");

const css = fs.readFileSync("./src/input.css", "utf8");
postcss([tailwindcss("./tailwind.config.js")])
  .process(css, { from: "./src/input.css" })
  .then((result) => fs.writeFileSync("./dist/output.css", result.css));
```

v4 does NOT expose a stable Node API in the same shape. Use the
`@tailwindcss/postcss` plugin or the `@tailwindcss/cli` binary instead.

## Section 11 : Output Behaviour Across Modes

| Mode | Selectors emitted | Vendor prefixes | Whitespace | Comments |
|------|-------------------|-----------------|------------|----------|
| dev (default) | all detected | v3 via autoprefixer ; v4 via Lightning CSS | preserved | preserved |
| dev `--watch` | all detected | same | preserved | preserved |
| `--minify` | all detected | same | stripped | stripped (except license `/*! */`) |
| `--minify` + v4 | all detected | same | stripped | stripped, plus dead-code elimination |

Output is deterministic : same input + same config + same content =
same byte-for-byte CSS.
