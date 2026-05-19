---
name: tailwind-impl-build-frameworks
description: >
  Use when wiring Tailwind CSS into Astro, SvelteKit, Nuxt, or Remix:
  choosing the right install path per framework + Tailwind version,
  picking the correct config file (astro.config.mjs, svelte.config.js,
  nuxt.config.ts, vite.config.ts), placing the global CSS in the
  framework-specific location, importing it from the right entry
  (layout, root, app.vue), and handling SSR/CSR specifics. Prevents
  the wrong-integration trap (using @astrojs/tailwind with v4 instead
  of @tailwindcss/vite, using @nuxtjs/tailwindcss module with v4
  instead of the Vite plugin), the missing-import trap (global CSS
  file exists but never imported in layout/root, so utilities silently
  miss), the SSR-flash trap (CSS not in critical path because
  imported in client-only branch), the v3-v4 mix-up (npx astro add
  tailwind installs the v3 integration even in a v4 project), and
  the wrong-CSS-path trap (Nuxt expects ./app/assets/css/main.css,
  Remix expects ./app/tailwind.css with ?url query). Covers Astro
  4+/5+ (v4 via Vite plugin in astro.config, v3 via @astrojs/tailwind),
  SvelteKit (Vite plugin + +layout.svelte import for both versions),
  Nuxt 3+/4+ (v4 via @tailwindcss/vite + css array entry, v3 via
  @nuxtjs/tailwindcss module), Remix (Vite plugin + LinksFunction in
  root.tsx with ?url stylesheet import), per-framework gotchas
  (Svelte :global, Nuxt assets path, Astro Vite-config bridge,
  Remix stylesheet URL import).
  Keywords: tailwind astro install, astro tailwind v4, npx astro
  add tailwind, @astrojs/tailwind, tailwind sveltekit, sveltekit
  tailwind v4, sveltekit tailwind layout, tailwind nuxt v4, nuxt
  @tailwindcss/vite, @nuxtjs/tailwindcss module, nuxt config css
  array, tailwind remix, remix tailwind links function,
  remix tailwind v4, app/tailwind.css remix, root.tsx links
  stylesheet, +layout.svelte tailwind import, app.css sveltekit,
  global.css astro, main.css nuxt, framework guides tailwind,
  ssr tailwind flash, my classes not working astro, tailwind not
  loading sveltekit, nuxt tailwind module vs plugin, remix v3 vs
  v4 tailwind, how to install tailwind nuxt, how to install
  tailwind sveltekit, how to install tailwind astro, how to
  install tailwind remix, framework-specific tailwind setup,
  tailwind 3-4 framework guide.
license: MIT
compatibility: "Designed for Claude Code. Requires Tailwind CSS v3.4 or v4.0+."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# Tailwind CSS Build Integration for Astro, SvelteKit, Nuxt, Remix

These four meta-frameworks all run on Vite under the hood, but each
hides Vite behind its own config layer and entry contract. Tailwind
wiring is therefore framework-specific, and v3 and v4 take different
paths inside each one.

ALWAYS pick the integration that matches the Tailwind major version :

| Framework  | v3 integration              | v4 integration            |
| ---------- | --------------------------- | ------------------------- |
| Astro      | `@astrojs/tailwind`         | `@tailwindcss/vite`       |
| SvelteKit  | `tailwindcss` PostCSS plugin| `@tailwindcss/vite`       |
| Nuxt       | `@nuxtjs/tailwindcss` module| `@tailwindcss/vite`       |
| Remix      | `tailwindcss` PostCSS plugin| `@tailwindcss/vite`       |

The v4 path converges across all four : add `@tailwindcss/vite` to
the framework's Vite config and import a single CSS file with
`@import "tailwindcss";`. The v3 paths differ because v3 ships
framework-specific integrations.

Companion skills :

- `tailwind-impl-build-vite` : underlying Vite plugin mechanics
- `tailwind-impl-config-v3` : v3 JS-config surface
- `tailwind-impl-config-v4` : v4 CSS-first config surface

## Quick Reference

### Astro (v4)

```bash
npm install tailwindcss @tailwindcss/vite
```

`astro.config.mjs` :

```js
import { defineConfig } from "astro/config"
import tailwindcss from "@tailwindcss/vite"

export default defineConfig({
  vite: { plugins: [tailwindcss()] },
})
```

`src/styles/global.css` :

```css
@import "tailwindcss";
```

Import in the layout :

```astro
---
import "../styles/global.css"
---
```

### Astro (v3)

```bash
npx astro add tailwind
```

This installs `tailwindcss` + `@astrojs/tailwind` and writes
`tailwind.config.mjs` and updates `astro.config.mjs` automatically.
ALWAYS accept the auto-edit ; manual setup is unsupported.

### SvelteKit (v4)

```bash
npm install tailwindcss @tailwindcss/vite
```

`vite.config.ts` :

```ts
import { sveltekit } from "@sveltejs/kit/vite"
import { defineConfig } from "vite"
import tailwindcss from "@tailwindcss/vite"

export default defineConfig({
  plugins: [tailwindcss(), sveltekit()],
})
```

`src/app.css` :

```css
@import "tailwindcss";
```

`src/routes/+layout.svelte` :

```svelte
<script>
  import "../app.css"
  let { children } = $props()
</script>

{@render children()}
```

### SvelteKit (v3)

```bash
npm install -D tailwindcss@3 postcss autoprefixer
npx tailwindcss init -p
```

`tailwind.config.js` :

```js
export default {
  content: ["./src/**/*.{html,js,svelte,ts}"],
  theme: { extend: {} },
  plugins: [],
}
```

`src/app.css` :

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

`src/routes/+layout.svelte` :

```svelte
<script>
  import "../app.css"
</script>

<slot />
```

ALSO configure `vitePreprocess` in `svelte.config.js` so `<style>` blocks
get PostCSS treatment.

### Nuxt (v4)

```bash
npm install tailwindcss @tailwindcss/vite
```

`nuxt.config.ts` :

```ts
import tailwindcss from "@tailwindcss/vite"

export default defineNuxtConfig({
  css: ["./app/assets/css/main.css"],
  vite: { plugins: [tailwindcss()] },
})
```

`app/assets/css/main.css` :

```css
@import "tailwindcss";
```

### Nuxt (v3)

```bash
npm install -D @nuxtjs/tailwindcss
```

`nuxt.config.ts` :

```ts
export default defineNuxtConfig({
  modules: ["@nuxtjs/tailwindcss"],
})
```

The module wires PostCSS, generates a default config, and exposes
hooks for `tailwind.config.{js,ts,mjs}`. No `app.vue` import needed ;
the module injects the stylesheet automatically.

### Remix (v4, Vite-based Remix 2+)

```bash
npm install tailwindcss @tailwindcss/vite
```

`vite.config.ts` :

```ts
import { defineConfig } from "vite"
import { vitePlugin as remix } from "@remix-run/dev"
import tailwindcss from "@tailwindcss/vite"

export default defineConfig({
  plugins: [tailwindcss(), remix()],
})
```

`app/tailwind.css` :

```css
@import "tailwindcss";
```

`app/root.tsx` :

```tsx
import type { LinksFunction } from "@remix-run/node"
import stylesheet from "~/tailwind.css?url"

export const links: LinksFunction = () => [
  { rel: "stylesheet", href: stylesheet },
]
```

The `?url` query is REQUIRED so Vite returns the asset URL rather
than inlining the CSS as a JS module.

### Remix (v3)

```bash
npm install -D tailwindcss@3 postcss autoprefixer
npx tailwindcss init --ts -p
```

`tailwind.config.ts` :

```ts
import type { Config } from "tailwindcss"

export default {
  content: ["./app/**/*.{js,jsx,ts,tsx}"],
  theme: { extend: {} },
  plugins: [],
} satisfies Config
```

`app/tailwind.css` :

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

`app/root.tsx` : same `LinksFunction` pattern as v4.

## Decision Trees

### Which integration to use ?

```
Framework + Tailwind version ?
├── Astro + v4    : @tailwindcss/vite in astro.config.vite.plugins
├── Astro + v3    : npx astro add tailwind (auto-wires @astrojs/tailwind)
├── SvelteKit + v4: @tailwindcss/vite in vite.config plugins
├── SvelteKit + v3: tailwindcss PostCSS via postcss.config.js
├── Nuxt + v4     : @tailwindcss/vite in nuxt.config.vite.plugins
├── Nuxt + v3     : @nuxtjs/tailwindcss module in nuxt.config.modules
├── Remix + v4    : @tailwindcss/vite in vite.config plugins
└── Remix + v3    : tailwindcss PostCSS + LinksFunction
```

### Where does the entry CSS go ?

```
Framework ?
├── Astro     : src/styles/global.css        (free choice, import from layout)
├── SvelteKit : src/app.css                  (canonical)
├── Nuxt v4   : app/assets/css/main.css      (referenced in nuxt.config css array)
├── Nuxt v3   : (module-managed, no file needed)
└── Remix     : app/tailwind.css             (canonical, referenced via LinksFunction)
```

### My classes do not apply : where do I check ?

```
1. Did you install both `tailwindcss` and the framework integration ?
   ├── No  : reinstall both
   └── Yes : continue
2. Is the integration registered in the framework's config ?
   ├── No  : add it
   └── Yes : continue
3. Does the entry CSS file actually exist at the expected path ?
   ├── No  : create it
   └── Yes : continue
4. Is the entry CSS imported / referenced by the framework's entry ?
   ├── Astro     : import "../styles/global.css" in layout
   ├── SvelteKit : import "../app.css" in +layout.svelte
   ├── Nuxt v4   : "./app/assets/css/main.css" in nuxt.config.css
   ├── Nuxt v3   : (handled by module)
   └── Remix     : LinksFunction in root.tsx
5. Browser DevTools : is the stylesheet loaded ?
   ├── No  : check the import path
   └── Yes : check content/source scope (per tailwind-impl-build-vite)
```

## Patterns

### Pattern : Nuxt module vs Vite plugin

v3 SHOULD use `@nuxtjs/tailwindcss` because the module wires PostCSS,
config discovery, and dev-server reloads correctly with Nuxt's nitro
pipeline. v4 MUST use `@tailwindcss/vite` because the Nuxt module
has not adopted v4 at the time of writing.

NEVER install both. The module owns config discovery ; adding the Vite
plugin alongside it double-emits CSS.

### Pattern : Astro v3 auto-install vs v4 manual

For v3 ALWAYS run `npx astro add tailwind`. The integration package
configures `astro.config.mjs` and emits `tailwind.config.mjs` with
content paths the Astro engine understands.

For v4 ALWAYS configure manually via `@tailwindcss/vite` in
`astro.config.mjs > vite.plugins`. The `@astrojs/tailwind` integration
does NOT understand v4 directives and produces broken output if used
with v4.

### Pattern : Svelte scoped styles with @apply (v4)

In SvelteKit + v4, `@apply` inside `<style>` blocks requires
`@reference` to resolve tokens :

```svelte
<style>
  @reference "../app.css";

  h1 {
    @apply text-3xl font-bold;
  }
</style>
```

Without `@reference`, build errors with "Cannot apply unknown utility class".

### Pattern : Remix LinksFunction is mandatory

Remix renders `<link>` tags from the `links` export, not from
JS-side `import "./styles.css"`. ALWAYS use `LinksFunction` :

```tsx
export const links: LinksFunction = () => [
  { rel: "stylesheet", href: stylesheet },
]
```

Importing the CSS in `root.tsx` without the LinksFunction works in
dev but fails in production builds because Remix splits client and
server bundles. The `?url` import ensures Vite emits the asset and
returns its URL.

### Pattern : Nuxt CSS array vs <style>

Nuxt v4 expects the entry stylesheet in the `css` array of
`nuxt.config.ts`. NEVER put `@import "tailwindcss"` inside an
`app.vue` `<style>` block ; Nuxt only treats files listed in `css`
as global. Component `<style scoped>` blocks need `@reference` like Svelte.

## Anti-Patterns

NEVER use `@astrojs/tailwind` with v4. It is a v3-only integration.

NEVER use `@nuxtjs/tailwindcss` with v4. Use `@tailwindcss/vite` instead.

NEVER skip the `?url` query on the Remix CSS import. Vite will inline
the CSS as a JS module string instead of emitting it as an asset.

NEVER mix `@tailwind base/components/utilities` (v3 directives) with
`@import "tailwindcss"` (v4 directive) in the same entry CSS.

NEVER assume `npx astro add tailwind` works for v4 ; the integration
it installs is v3-pinned.

See `references/anti-patterns.md` for the full catalog with quoted
error symptoms and fixes.

## Reference Links

- `references/methods.md` : per-framework install commands, config-file
  shapes, entry-CSS locations, framework-plugin signatures
- `references/examples.md` : copy-paste starter projects for each
  framework, both v3 and v4
- `references/anti-patterns.md` : known wiring mistakes per framework
  with symptoms, causes, and fixes

## Sources

- v4 Astro guide : https://tailwindcss.com/docs/installation/framework-guides/astro
- v4 SvelteKit guide : https://tailwindcss.com/docs/installation/framework-guides/sveltekit
- v4 Nuxt guide : https://tailwindcss.com/docs/installation/framework-guides/nuxt
- v3 Astro guide : https://v3.tailwindcss.com/docs/guides/astro
- v3 SvelteKit guide : https://v3.tailwindcss.com/docs/guides/sveltekit
- v3 Remix guide : https://v3.tailwindcss.com/docs/guides/remix
- v3 Nuxt guide : https://v3.tailwindcss.com/docs/guides/nuxtjs
- @nuxtjs/tailwindcss module : https://tailwindcss.nuxtjs.org
- Remix Vite docs : https://remix.run/docs/en/main/future/vite
