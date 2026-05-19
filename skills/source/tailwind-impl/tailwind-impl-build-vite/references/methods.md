# Reference : Tailwind CSS + Vite Methods and Options

Exhaustive reference for the Vite-side wiring of Tailwind, both
`@tailwindcss/vite` (v4) and the v3 PostCSS-plugin path.

## v4 : @tailwindcss/vite

### Install

```bash
npm install tailwindcss @tailwindcss/vite
# or
pnpm add tailwindcss @tailwindcss/vite
# or
yarn add tailwindcss @tailwindcss/vite
```

Both `tailwindcss` (the engine) and `@tailwindcss/vite` (the integration)
MUST be installed. The plugin re-exports nothing useful without the engine.

### Plugin signature

```ts
import tailwindcss from "@tailwindcss/vite"

tailwindcss(): Plugin[]
```

The plugin export is a function with **no required arguments**. All
configuration in v4 lives in the entry CSS via `@import "tailwindcss"`,
`@theme`, `@source`, `@plugin`, etc. There is no plugin-options object
in v4 Vite integration.

### Where to register

In `vite.config.ts` (or `vite.config.js`), inside `plugins`. Order does
not affect Tailwind's CSS pipeline, but framework plugins that transform
single-file components (`@vitejs/plugin-vue`, `@sveltejs/vite-plugin-svelte`)
SHOULD run before `tailwindcss()` so component-scoped styles are
extracted before the CSS plugin runs.

```ts
plugins: [
  vue(),        // or react(), svelte(), solid(), qwikVite()
  tailwindcss(),
]
```

### Entry CSS directives that the v4 plugin honors

| Directive             | Purpose |
| --------------------- | ------- |
| `@import "tailwindcss"` | Loads preflight + utilities + theme. ALWAYS the first line. |
| `@theme { ... }`      | Defines design tokens (CSS variables) that produce utilities. |
| `@theme inline { ... }`| Like @theme but resolves var refs at build time. |
| `@theme static { ... }`| Forces token output even if unused. |
| `@source "path"`      | Adds a glob to the template scanner. Use for monorepo paths. |
| `@source not "path"`  | Excludes a glob from scanning. |
| `@source inline "{...}"` | Brace-expansion safelist for dynamic class names. |
| `@source none`        | Disables automatic scanning entirely. |
| `@plugin "name"`      | Loads a legacy JS plugin (e.g. `@tailwindcss/forms`). |
| `@utility name`       | Declares a custom utility (with --value/--modifier helpers). |
| `@variant name`       | Uses a variant inline in CSS. |
| `@custom-variant name` | Declares a new variant. |
| `@reference`          | Brings tokens into scoped styles (Vue/Svelte/CSS modules). |
| `@config "path"`      | Opt-in JS-config bridge for migrating from v3. |
| `@apply`              | Inlines utilities into a component class. |
| `@layer base|components|utilities` | Layered insertions. |

Full directive semantics live in `tailwind-impl-config-v4`. The Vite
plugin's job is to drive the engine that interprets them.

### Automatic content detection

v4 with `@tailwindcss/vite` scans :

- Every file under the Vite root (`vite.config.ts`'s `root`, default `process.cwd()`).
- Skipping anything listed in the project's `.gitignore`.
- Skipping binary file types.

Files outside the Vite root, or symlinked workspace packages, are NOT
auto-scanned. ALWAYS declare them with `@source "../path/**/*.{ts,tsx}"`
in the entry CSS.

### HMR behavior

The v4 plugin participates in Vite's dependency graph. Editing :

- The entry CSS triggers a CSS-only HMR swap.
- A template file (matched by the scanner) triggers an incremental
  re-scan and a CSS swap.
- An `@source`-referenced file triggers the same incremental flow.
- A `@config`-loaded JS config triggers a full module invalidation.

NEVER add a manual `server.watch.include` entry for Tailwind files :
the plugin already registers its own watchers.

## v3 : tailwindcss PostCSS plugin

### Install

```bash
npm install -D tailwindcss@3 postcss autoprefixer
```

Pin to `tailwindcss@3` when you intentionally want v3 ; otherwise `npm
install -D tailwindcss` may resolve to v4 and silently break v3
expectations.

### Init

```bash
npx tailwindcss init -p
```

Flags :

| Flag                | Effect |
| ------------------- | ------ |
| (no flag)           | Generates `tailwind.config.js` only. |
| `-p`                | Also generates `postcss.config.js`. |
| `--esm`             | Emits ES-module syntax (default in projects with `"type": "module"`). |
| `--ts`              | Emits a TypeScript config file. |
| `--full`            | Includes the full default theme inline. |

ALWAYS use `init -p` for Vite : Vite reads `postcss.config.js`
automatically with no extra config.

### tailwind.config.js options the Vite path cares about

```js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx,vue,svelte}",
  ],
  darkMode: "class",       // or "media"
  theme: { extend: {} },
  plugins: [],
  safelist: [
    "bg-red-500",
    { pattern: /bg-(red|green|blue)-(100|500|900)/ },
  ],
  corePlugins: { preflight: true },
}
```

ALWAYS list every extension you author classes in inside `content`.
The JIT engine never sees files outside that list.

### postcss.config.js

```js
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

Order matters : `tailwindcss` first, `autoprefixer` second. NEVER add
`postcss-import` after `tailwindcss` : the v3 plugin already handles
`@tailwind`, `@apply`, and `@layer` ; `postcss-import` MUST run before
`tailwindcss` if you need it at all.

### Entry CSS directives (v3)

| Directive                  | Purpose |
| -------------------------- | ------- |
| `@tailwind base;`          | Emits preflight + base styles. |
| `@tailwind components;`    | Layer placeholder for component classes. |
| `@tailwind utilities;`     | Emits all generated utilities. |
| `@layer base { ... }`      | Adds CSS to the base layer. |
| `@layer components { ... }`| Adds CSS to the components layer. |
| `@layer utilities { ... }` | Adds custom utilities. |
| `@apply ...`               | Inlines utilities into a component class. |
| `@screen sm { ... }`       | Wraps in a responsive media query (deprecated, use breakpoint variants). |
| `@variants ... { ... }`    | Generates variant utilities (deprecated in 3.x). |

NEVER write `@import "tailwindcss"` in a v3 stylesheet. That is v4 syntax.

### HMR behavior (v3)

Vite watches files in the `content` array via PostCSS dependency
reporting. Editing :

- The entry CSS triggers a CSS-only HMR swap.
- A `content`-matched template file triggers a JIT rescan + CSS swap.
- `tailwind.config.js` triggers a full server restart in most setups
  (the JIT engine reads the config eagerly).

If `tailwind.config.js` edits do not reload, restart the dev server.
NEVER paper over with a watcher plugin : the config-reload is a
known v3 limitation.

## CLI commands

| Command                                      | Purpose |
| -------------------------------------------- | ------- |
| `npx tailwindcss init`                       | v3 : generate tailwind.config.js. |
| `npx tailwindcss init -p`                    | v3 : generate tailwind.config.js + postcss.config.js. |
| `npx tailwindcss -i src/index.css -o dist/out.css --watch` | v3 standalone CLI (not used with Vite plugin). |
| `npx @tailwindcss/upgrade`                   | v3-to-v4 codemod (NOT Vite-specific). |
| `npm run dev`                                | Standard Vite dev server. |
| `npm run build`                              | Standard Vite production build. |

The `tailwindcss` CLI is independent of the Vite plugin. NEVER mix
CLI-driven generation with the Vite plugin in the same project ; pick one.

## Framework integrations

### React (`@vitejs/plugin-react`)

Standard. Add `tailwindcss()` to `plugins`. Reference the entry CSS
from `main.tsx` :

```ts
import "./style.css"
```

### Vue (`@vitejs/plugin-vue`)

In `<style scoped>` blocks, `@apply` requires `@reference "../style.css"`
in v4 to resolve tokens. v3 does not need this.

### Svelte (`@sveltejs/vite-plugin-svelte`)

Same as Vue : scoped styles need `@reference` in v4 for `@apply`.

### SolidJS (`vite-plugin-solid`)

Standard. No scoped-style quirks.

### Astro (`@astrojs/vite`)

Astro's Vite integration auto-discovers `@tailwindcss/vite` when
listed in `plugins`. ALWAYS pin Tailwind to a patched 4.x release (NOT
4.0.8) to avoid the issue 16733 regression.

### Qwik (`@builder.io/qwik/optimizer`)

Standard. Tailwind plugin runs after Qwik's optimizer.

## Verifying the install

```bash
# Dev
npm run dev
# Browse to the local URL, inspect a div with `class="text-3xl font-bold"`
# Confirm computed style shows font-size: 1.875rem and font-weight: 700.

# Prod
npm run build
ls dist/assets/*.css   # confirm a CSS file emitted
```

If `dist/assets/*.css` is missing or near-empty, the scanner found no
classes. Re-check content/source scope.
