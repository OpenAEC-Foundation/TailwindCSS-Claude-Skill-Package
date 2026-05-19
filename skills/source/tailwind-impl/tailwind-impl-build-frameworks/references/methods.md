# Reference : Framework Integration Methods

Exhaustive reference for per-framework install commands, config-file
shapes, entry-CSS conventions, and integration-package signatures.

## Astro

### v4

| Step              | Command / file |
| ----------------- | -------------- |
| Install           | `npm install tailwindcss @tailwindcss/vite` |
| Configure         | edit `astro.config.mjs` `vite.plugins` |
| Entry CSS         | `src/styles/global.css` (path is your choice) |
| Import entry      | `import "../styles/global.css"` in layout |

`astro.config.mjs` :

```js
import { defineConfig } from "astro/config"
import tailwindcss from "@tailwindcss/vite"

export default defineConfig({
  vite: {
    plugins: [tailwindcss()],
  },
})
```

The `vite` key in Astro's config is passed directly to Vite. Any Vite
plugin works there, including `@tailwindcss/vite`.

### v3

| Step              | Command / file |
| ----------------- | -------------- |
| Install + wire    | `npx astro add tailwind` |
| Generated config  | `tailwind.config.mjs` (auto-created) |
| Astro integration | `@astrojs/tailwind` (auto-added) |

The `astro add` command :

1. Prompts to install `tailwindcss` and `@astrojs/tailwind`.
2. Edits `astro.config.mjs` to add the integration.
3. Generates `tailwind.config.mjs` with Astro-friendly defaults.

NEVER configure v3 Astro manually with the PostCSS plugin. The
integration handles preflight injection and content path discovery
in an Astro-specific way.

## SvelteKit

### v4

| Step              | Command / file |
| ----------------- | -------------- |
| Install           | `npm install tailwindcss @tailwindcss/vite` |
| Configure Vite    | edit `vite.config.ts` |
| Entry CSS         | `src/app.css` |
| Import in layout  | `src/routes/+layout.svelte` |

`vite.config.ts` :

```ts
import { sveltekit } from "@sveltejs/kit/vite"
import { defineConfig } from "vite"
import tailwindcss from "@tailwindcss/vite"

export default defineConfig({
  plugins: [tailwindcss(), sveltekit()],
})
```

ALWAYS register `tailwindcss()` before `sveltekit()` so the CSS plugin
runs first in the build pipeline.

`+layout.svelte` :

```svelte
<script>
  import "../app.css"
  let { children } = $props()
</script>

{@render children()}
```

For Svelte 4 (pre-runes) :

```svelte
<script>
  import "../app.css"
</script>

<slot />
```

### v3

| Step              | Command / file |
| ----------------- | -------------- |
| Install           | `npm install -D tailwindcss@3 postcss autoprefixer` |
| Init              | `npx tailwindcss init -p` |
| Configure content | `tailwind.config.js` content paths |
| Configure Svelte  | `svelte.config.js` vitePreprocess |
| Entry CSS         | `src/app.css` (three @tailwind directives) |

`svelte.config.js` :

```js
import adapter from "@sveltejs/adapter-auto"
import { vitePreprocess } from "@sveltejs/kit/vite"

export default {
  preprocess: vitePreprocess(),
  kit: { adapter: adapter() },
}
```

`vitePreprocess` enables PostCSS for `<style>` blocks. Without it,
`@apply` inside Svelte components fails.

## Nuxt

### v4

| Step              | Command / file |
| ----------------- | -------------- |
| Install           | `npm install tailwindcss @tailwindcss/vite` |
| Configure         | edit `nuxt.config.ts` (both `vite.plugins` and `css`) |
| Entry CSS         | `app/assets/css/main.css` |

`nuxt.config.ts` :

```ts
import tailwindcss from "@tailwindcss/vite"

export default defineNuxtConfig({
  compatibilityDate: "2025-07-15",
  devtools: { enabled: true },
  css: ["./app/assets/css/main.css"],
  vite: {
    plugins: [tailwindcss()],
  },
})
```

The path in `css` MUST be relative to the project root, NOT to the
`app/` directory. Nuxt resolves it once at build time.

### v3

| Step              | Command / file |
| ----------------- | -------------- |
| Install           | `npm install -D @nuxtjs/tailwindcss` |
| Register module   | edit `nuxt.config.ts` `modules` |
| Optional config   | `tailwind.config.{js,ts,mjs}` |

`nuxt.config.ts` :

```ts
export default defineNuxtConfig({
  modules: ["@nuxtjs/tailwindcss"],
})
```

The module :

- Auto-installs `tailwindcss`, `postcss`, `autoprefixer`.
- Generates a default Tailwind config.
- Injects the stylesheet automatically (no `<link>` needed).
- Exposes Nuxt hooks `tailwindcss:config` and `tailwindcss:resolvedConfig`.

To override the config, create `tailwind.config.ts` at the project
root. The module merges your overrides with its defaults.

## Remix

Remix 2+ runs on Vite. The Vite plugin pattern is identical to the
bare-Vite case ; the only Remix-specific detail is `LinksFunction`.

### v4

| Step              | Command / file |
| ----------------- | -------------- |
| Install           | `npm install tailwindcss @tailwindcss/vite` |
| Configure Vite    | edit `vite.config.ts` |
| Entry CSS         | `app/tailwind.css` |
| Link in root      | `app/root.tsx` LinksFunction |

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

The `?url` query is REQUIRED. Without it, Vite returns the CSS as a
JS string instead of an asset URL, and Remix cannot link to it.

### v3

Same Vite-plugin and LinksFunction shape ; CSS uses the v3 directives.

`app/tailwind.css` :

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
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

ALWAYS use `init --ts -p` for TypeScript projects so the config file
gets proper types from `tailwindcss`.

## Cross-framework helpers

### Verifying the install

After install, drop a single utility class in the framework's hello-world
file :

```html
<h1 class="text-3xl font-bold underline">Hello Tailwind</h1>
```

If the text becomes large, bold, and underlined, the integration is wired.

### Checking the loaded stylesheet

Browser DevTools, Sources tab :

- Astro      : `src/styles/global.css` (dev) / hashed CSS asset (build)
- SvelteKit  : `src/app.css` (dev) / `_app/immutable/assets/*.css` (build)
- Nuxt       : virtual entry resolving to `main.css` (dev) / hashed asset (build)
- Remix      : `app/tailwind.css?url` (dev) / hashed asset (build)

If the stylesheet is missing from Sources, the import in the framework
entry is wrong. If present but empty, the scanner found no classes.

### Disabling the module / integration

To rip out the integration cleanly :

- Astro v3       : remove `@astrojs/tailwind` from `astro.config.mjs` integrations.
- Astro v4       : remove `tailwindcss()` from `vite.plugins`.
- SvelteKit v3   : delete `tailwindcss` from `postcss.config.js`.
- SvelteKit v4   : remove `tailwindcss()` from `vite.plugins`.
- Nuxt v3        : remove `@nuxtjs/tailwindcss` from `modules`.
- Nuxt v4        : remove `tailwindcss()` from `vite.plugins` AND `css` entry.
- Remix          : remove `tailwindcss()` from `vite.plugins` AND links entry.

ALWAYS remove the matching entry CSS file too, otherwise the framework
keeps loading raw CSS without Tailwind processing.
