---
name: tailwind-impl-build-postcss-cli
description: >
  Use when wiring Tailwind CSS into a non-Vite build : the PostCSS plugin
  (Next.js, Remix, Astro pre-Vite, Nuxt v3, Webpack, Rollup, esbuild,
  Parcel, Laravel Mix, Symfony Encore, any postcss.config.js consumer),
  or the standalone CLI binary for Rails, Laravel Blade, Django,
  PHP, Go, Rust, Elixir, .NET, or any non-Node project that has no JS
  bundler. Prevents the wrong-plugin-name trap (v4 ships
  @tailwindcss/postcss as a separate package, NOT tailwindcss directly),
  the autoprefixer-duplication trap (v4 includes Lightning CSS so
  adding autoprefixer in postcss.config breaks the cascade), the
  forgot-init trap (v3 requires npx tailwindcss init to create
  tailwind.config.js before content scanning works), the CLI-binary-vs-npx
  trap (CLI runs as npx @tailwindcss/cli in v4 but npx tailwindcss in v3),
  and the @tailwind-vs-@import trap (v3 uses three @tailwind directives,
  v4 uses one @import "tailwindcss").
  Covers postcss.config.mjs/cjs/js for both versions, every CLI flag
  (--input/-i, --output/-o, --watch, --minify, --config, --cwd),
  framework-specific integration (Next.js, Astro, Remix, Nuxt,
  SvelteKit, plain HTML/PHP/Rails), the v3 to v4 plugin rename,
  standalone executable usage without Node, and the autoprefixer
  pairing rule (v3 needs it explicitly, v4 never needs it).
  Keywords: tailwind postcss, @tailwindcss/postcss, postcss config mjs,
  postcss config js, postcss plugin, tailwind cli, @tailwindcss/cli,
  npx tailwindcss, npx @tailwindcss/cli, --input flag, --output flag,
  --watch flag, --minify flag, -i flag, -o flag, autoprefixer tailwind,
  lightning css, oxide engine, tailwind without node, tailwind
  standalone binary, tailwind rails, tailwind laravel, tailwind django,
  tailwind php, tailwind go, tailwind nextjs postcss, tailwind astro
  postcss, tailwind nuxt, tailwind webpack, tailwind rollup, tailwind
  esbuild, tailwind parcel, my postcss config not working, plugin
  @tailwindcss/postcss not found, Cannot find module tailwindcss,
  tailwindcss init missing, how do I build tailwind without webpack,
  how to watch tailwind, how to minify tailwind output, npm install
  tailwind postcss, tailwind binary download, tailwind cli flags
license: MIT
compatibility: "Designed for Claude Code. Requires Tailwind CSS v3.4 or v4.0+."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# Tailwind CSS PostCSS + Standalone CLI Build

Two non-Vite ways to compile Tailwind : the **PostCSS plugin** (drops into
any bundler that runs PostCSS) and the **standalone CLI** (a Node script
or a single binary, used when there is no bundler at all).

This skill covers BOTH Tailwind v3 and v4 because the package names,
config-file shapes, and CSS entry-point syntax all differ between
versions. Pick the right combination or compilation silently produces
empty CSS.

Companion skills :

- `tailwind-impl-build-vite` : the Vite plugin path (preferred for SPAs)
- `tailwind-impl-config-v3` : the v3 JS-config surface
- `tailwind-impl-config-v4` : the v4 CSS-first config surface
- `tailwind-core-v3-vs-v4` : every breaking change across the API
- `tailwind-errors-build` : when builds fail or output is empty

## Quick Reference : Which Path Do You Need

```
Have a Vite project ?              -> tailwind-impl-build-vite
Have any other JS bundler ?         -> PostCSS plugin (this skill)
   (Webpack, Rollup, esbuild, Parcel, Laravel Mix, Encore)
Have Next.js (any version) ?        -> PostCSS plugin (this skill)
Have Remix, Nuxt, Astro (pre-v4) ?  -> PostCSS plugin (this skill)
No JS bundler at all ?              -> Standalone CLI (this skill)
   (Rails, Laravel Blade, Django, Flask, Go html/template,
    Phoenix, plain HTML, static sites)
No Node.js installed ?              -> Standalone CLI binary (this skill)
```

## Quick Reference : Minimum v4 PostCSS Setup

```bash
npm install tailwindcss @tailwindcss/postcss postcss
```

```js
// postcss.config.mjs
export default {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
```

```css
/* src/app.css */
@import "tailwindcss";
```

No `tailwind.config.js`. No `autoprefixer`. No `postcss-import`. The
v4 plugin embeds Lightning CSS, which handles vendor prefixing,
`@import` inlining, and modern-CSS lowering internally.

## Quick Reference : Minimum v3 PostCSS Setup

```bash
npm install -D tailwindcss@3 postcss autoprefixer
npx tailwindcss init
```

```js
// postcss.config.js
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

```js
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./src/**/*.{html,js,jsx,ts,tsx,vue,svelte}"],
  theme: { extend: {} },
  plugins: [],
};
```

```css
/* src/app.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Note : v3 plugin name is bare `tailwindcss`. v4 plugin name is
`@tailwindcss/postcss`. Mixing these (e.g. `tailwindcss: {}` in a v4
project) yields PostCSS error `Cannot find module 'tailwindcss/postcss'`.

## Quick Reference : Minimum v4 CLI Setup

```bash
npm install tailwindcss @tailwindcss/cli
```

```css
/* src/input.css */
@import "tailwindcss";
```

```bash
npx @tailwindcss/cli -i ./src/input.css -o ./dist/output.css --watch
```

Production build (one-shot, minified) :

```bash
npx @tailwindcss/cli -i ./src/input.css -o ./dist/output.css --minify
```

## Quick Reference : Minimum v3 CLI Setup

```bash
npm install -D tailwindcss@3
npx tailwindcss init
```

```css
/* src/input.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

```bash
npx tailwindcss -i ./src/input.css -o ./dist/output.css --watch
```

Production build :

```bash
npx tailwindcss -i ./src/input.css -o ./dist/output.css --minify
```

Note : v3 CLI runs as `npx tailwindcss`. v4 CLI runs as
`npx @tailwindcss/cli`. The binary name is different because v4 split
the CLI into its own npm package.

## Decision Tree : Picking PostCSS vs CLI

```
Are you using a JS bundler that ALREADY runs PostCSS ?
  ├── YES (Next.js, Webpack, Rollup, Parcel, esbuild-with-postcss-plugin,
  │        Laravel Mix, Symfony Encore, Astro pre-Vite, Nuxt v3)
  │   -> Use PostCSS plugin. Add it to the existing postcss.config.
  │
  └── NO bundler at all, or non-Node backend
      ├── Have Node.js available ?
      │   ├── YES -> Use npx CLI from package.json scripts
      │   └── NO  -> Download standalone executable from GitHub releases
      │              (see `references/methods.md` § Standalone Binary)
      ├── Rails / Laravel / Django / Phoenix / Go / .NET ?
      │   -> Standalone CLI is the official recommendation
      └── Static HTML site, plain CSS workflow ?
          -> Standalone CLI with --watch in development
```

## Decision Tree : Picking v3 vs v4

```
New project, no existing Tailwind code ?
  -> v4 (CSS-first config, faster builds, modern CSS features)

Existing v3 project ?
  ├── Critical browsers IE 11 or Safari < 16.4 ?
  │   -> Stay on v3 (v4 targets Safari 16.4+, Chrome 111+, Firefox 128+)
  ├── Heavy JS-plugin ecosystem (typography custom variants, complex
  │   theme functions in tailwind.config.js) ?
  │   -> Stay on v3 OR migrate gradually with @config back-compat
  └── Otherwise
      -> Migrate to v4 (see `tailwind-impl-migration-v3-v4`)
```

## CLI Flags : Complete Reference

Both v3 and v4 CLIs share the same core flags. Listed long form first,
short form second where present.

| Flag | Short | Purpose | v3 | v4 |
|------|-------|---------|----|----|
| `--input` | `-i` | Path to source CSS file | yes | yes |
| `--output` | `-o` | Path to compiled CSS file | yes | yes |
| `--watch` | `-w` | Rebuild on source change | yes | yes |
| `--minify` | `-m` | Compress output (production) | yes | yes |
| `--config` | `-c` | Path to tailwind.config.js | yes | v4 via `@config` only |
| `--cwd` | none | Override scan root | v3 only | v4 uses `@source` |
| `--postcss` | none | Path to postcss.config.js | v3 only | n/a |
| `--content` | none | Override content paths (comma-list) | v3 only | v4 uses `@source` |
| `--help` | `-h` | Print usage | yes | yes |

In v4 the CLI auto-detects sources in the current working directory and
in directories referenced by `@source` directives in the entry CSS.
There is no `--content` flag because v4 has no JS config file by default.

## Decision Tree : `@import "tailwindcss"` vs `@tailwind` Trio

```
Tailwind v4 project ?
  -> Use ONE @import directive : @import "tailwindcss";

Tailwind v3 project ?
  -> Use THREE @tailwind directives :
       @tailwind base;
       @tailwind components;
       @tailwind utilities;
```

Mixing these is the most common cause of "Tailwind compiles but no
utility classes appear". v3 ignores `@import "tailwindcss"`. v4 emits
warnings for `@tailwind base` and produces partial output.

## Autoprefixer Rule (CRITICAL)

| Tailwind version | autoprefixer in postcss.config | Reason |
|------------------|-------------------------------|--------|
| v3 | REQUIRED | v3 does not prefix vendor properties itself |
| v4 | NEVER | Lightning CSS handles prefixing internally |

Adding `autoprefixer: {}` to a v4 PostCSS pipeline DOUBLE-PREFIXES
properties and breaks `:where()`, `:is()`, and container query output.
Removing it from v3 strips `-webkit-`, `-moz-`, `-ms-` from
`display: flex`, `transform`, `position: sticky` and crashes layouts
on Safari < 14, older Edge, and older Firefox.

## Framework Integration Cheatsheet

| Framework | Recommended v4 path | Recommended v3 path |
|-----------|---------------------|---------------------|
| Vite | `@tailwindcss/vite` plugin | `tailwindcss` PostCSS plugin |
| Next.js (App or Pages router) | `@tailwindcss/postcss` | `tailwindcss` PostCSS plugin |
| Astro | `@tailwindcss/vite` (Astro uses Vite) | `@astrojs/tailwind` integration |
| Remix | `@tailwindcss/postcss` | `tailwindcss` PostCSS plugin |
| Nuxt | `@tailwindcss/vite` | `@nuxtjs/tailwindcss` module |
| SvelteKit | `@tailwindcss/vite` | `tailwindcss` PostCSS plugin |
| Laravel Mix | `@tailwindcss/postcss` | `tailwindcss` PostCSS plugin |
| Symfony Encore | `@tailwindcss/postcss` | `tailwindcss` PostCSS plugin |
| Rails (Sprockets/Propshaft) | Standalone CLI | Standalone CLI |
| Django / Flask | Standalone CLI | Standalone CLI |
| Go html/template | Standalone CLI | Standalone CLI |
| Phoenix LiveView | Standalone CLI | Standalone CLI |
| Plain HTML | Standalone CLI | Standalone CLI |
| Webpack (no Vite) | `@tailwindcss/postcss` | `tailwindcss` PostCSS plugin |

Astro-with-v4 pin a known-good release : per upstream issue 16733 some
Astro patch versions regress on Vite plugin compatibility.

## When to Use the Standalone CLI Binary (no Node)

The standalone executable is a single self-contained binary published
on GitHub releases. Use it when :

- You have no Node.js installed in your environment.
- You ship a Docker image where adding Node + node_modules is undesired
  (a Rails image with a single binary is smaller than one with Node).
- Your CI pipeline can download a binary faster than running
  `npm install`.
- Your team is Ruby/PHP/Python/Go/Rust and Node is not an existing
  dependency.

Both v3 and v4 publish standalone binaries. Download URL pattern :
`https://github.com/tailwindlabs/tailwindcss/releases/latest` and pick
the platform-matching asset (linux-x64, linux-arm64, macos-x64,
macos-arm64, windows-x64). Make it executable and invoke it the same
way as `npx tailwindcss` (v3) or `npx @tailwindcss/cli` (v4) but with
the binary name : `./tailwindcss -i input.css -o output.css --watch`.

See `references/methods.md` § Standalone Binary for the exact download
and install steps for Linux, macOS, Windows.

## Build-Script Patterns (package.json)

```json
{
  "scripts": {
    "build:css": "npx @tailwindcss/cli -i src/input.css -o dist/output.css --minify",
    "dev:css":   "npx @tailwindcss/cli -i src/input.css -o dist/output.css --watch"
  }
}
```

For v3, swap `@tailwindcss/cli` for bare `tailwindcss`.

For a non-Node project (Rails/Django/PHP), drive the binary directly
from the project's build tool. Example Rake task :

```ruby
task :tailwind do
  sh "./bin/tailwindcss -i app/assets/stylesheets/application.tailwind.css " \
     "-o app/assets/builds/application.css --minify"
end
```

See `references/examples.md` for Rails, Django, Laravel Blade, and Go
end-to-end patterns.

## Watch Mode Mechanics

`--watch` does TWO things :

1. Recompiles when the input CSS file changes.
2. Re-scans content files for class names and recompiles when they
   change.

In v4 the watcher is the Oxide engine (Rust). In v3 it is chokidar
plus the JIT compiler. Both are fast enough for sub-second incremental
rebuilds on typical projects (under 200 ms in v4, 100-500 ms in v3).

CRITICAL : watch mode does NOT detect new files outside the content
paths configured in v3 (`content` array in `tailwind.config.js`) or in
v4 (the default scan + `@source` directives). If a new `.html` or
`.jsx` file appears in a previously-uncovered directory, restart the
watcher.

## Minify Mode

`--minify` strips whitespace, comments, and uses shortest-form syntax.
Output size reduction is typically 50-80% of unminified for a
production-ready bundle. ALWAYS combine `--minify` with the production
build, NEVER use it for `--watch` development.

In v4 minification ALSO triggers Lightning CSS's `-Os` mode :
- merges duplicate selectors
- normalises shorthand properties
- removes dead CSS

In v3 minification is cssnano-equivalent. Both versions produce
deterministic output (same input -> same byte-for-byte output).

## PostCSS Plugin Ordering (v3)

When the v3 PostCSS pipeline includes other plugins, ordering matters :

```js
module.exports = {
  plugins: {
    "postcss-import": {},     // 1. inline @import statements
    "tailwindcss/nesting": {}, // 2. resolve nested rules (Tailwind's nesting plugin)
    tailwindcss: {},          // 3. expand @tailwind / utility classes
    autoprefixer: {},         // 4. add vendor prefixes
    cssnano: {},              // 5. minify in production
  },
};
```

v4 has NO ordering concerns because Lightning CSS replaces all of
these steps internally. The v4 PostCSS config has exactly ONE plugin :
`"@tailwindcss/postcss": {}`.

## Common First-Run Errors (Quick Diagnosis)

| Error message | Cause | Fix |
|---------------|-------|-----|
| `Cannot find module '@tailwindcss/postcss'` | v4 plugin not installed | `npm install @tailwindcss/postcss` |
| `Cannot find module 'tailwindcss'` (v3 project) | Tailwind not installed | `npm install -D tailwindcss@3` |
| `error: unknown option '-i'` | Used v4 binary name in v3 project | Use `npx tailwindcss` not `npx @tailwindcss/cli` |
| Output CSS is empty | `content` paths wrong (v3) or no `@source` (v4) | See `tailwind-errors-build` |
| Build hangs in watch mode | Too-broad content glob | Narrow `content` array (v3) or scope `@source` (v4) |
| `Module build failed: Error: PostCSS plugin tailwindcss requires PostCSS 8` | PostCSS 7 in the chain | Upgrade to `postcss@^8` |

Full anti-pattern catalogue in `references/anti-patterns.md`.

## What This Skill Does NOT Cover

- Vite plugin setup (use `tailwind-impl-build-vite`)
- CSS-first config syntax (`@theme`, `@source`, `@utility` directives)
  belong in `tailwind-impl-config-v4`.
- JS config syntax (`tailwind.config.js` shape) belongs in
  `tailwind-impl-config-v3`.
- Why builds produce wrong output (debugging belongs in
  `tailwind-errors-build`).
- Migration from v3 to v4 (use `tailwind-impl-migration-v3-v4`).

## References

- `references/methods.md` : Every CLI flag, every PostCSS config shape,
  framework-specific code, standalone binary install.
- `references/examples.md` : End-to-end working setups for Next.js,
  Remix, Astro, SvelteKit, Rails, Laravel Blade, Django, Phoenix,
  Go, plain HTML.
- `references/anti-patterns.md` : The traps : wrong plugin name,
  autoprefixer double-prefix, @tailwind vs @import mix-up, missing
  init, watch-mode pitfalls.

## Official Source Links

- v4 PostCSS : https://tailwindcss.com/docs/installation/using-postcss
- v4 CLI : https://tailwindcss.com/docs/installation/tailwind-cli
- v3 PostCSS : https://v3.tailwindcss.com/docs/installation/using-postcss
- v3 install : https://v3.tailwindcss.com/docs/installation
- Standalone CLI releases : https://github.com/tailwindlabs/tailwindcss/releases
