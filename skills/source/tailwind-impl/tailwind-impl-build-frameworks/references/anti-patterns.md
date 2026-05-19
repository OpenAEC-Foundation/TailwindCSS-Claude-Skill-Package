# Anti-Patterns : Astro, SvelteKit, Nuxt, Remix

Every framework-specific wiring mistake, the symptom, the cause, and the fix.

---

## AP-1 : Using @astrojs/tailwind with v4

**Symptom :** Build error : `Unknown directive '@import "tailwindcss"'`,
or no utilities produced.

**Cause :** `@astrojs/tailwind` is a v3-only integration. It wires the
v3 PostCSS plugin and cannot parse v4 entry-CSS syntax.

**Fix :** Remove the integration, install `@tailwindcss/vite`, and
configure via `astro.config.mjs > vite.plugins` :

```diff
- import tailwind from "@astrojs/tailwind"
+ import tailwindcss from "@tailwindcss/vite"

  export default defineConfig({
-   integrations: [tailwind()],
+   vite: { plugins: [tailwindcss()] },
  })
```

---

## AP-2 : Running `npx astro add tailwind` for v4

**Symptom :** v3 utilities work but v4 directives (`@theme`, `@source`,
`@plugin`) raise build errors.

**Cause :** `astro add tailwind` installs the v3 `@astrojs/tailwind`
integration regardless of which Tailwind major you intended.

**Fix :** Configure v4 manually (see AP-1). Do NOT run `astro add tailwind`.

---

## AP-3 : Using @nuxtjs/tailwindcss with v4

**Symptom :** Module installs and dev server starts, but new v4
directives are silently ignored and `@theme` tokens never emit.

**Cause :** The Nuxt module wires the v3 PostCSS plugin. It does not
support v4 at the time of writing.

**Fix :** Remove the module and switch to `@tailwindcss/vite` :

```diff
  export default defineNuxtConfig({
-   modules: ["@nuxtjs/tailwindcss"],
+   css: ["./app/assets/css/main.css"],
+   vite: { plugins: [tailwindcss()] },
  })
```

ALSO create `app/assets/css/main.css` with `@import "tailwindcss"`.

---

## AP-4 : Missing `css` array entry in Nuxt v4

**Symptom :** `app/assets/css/main.css` exists with valid Tailwind
import, but no styles apply.

**Cause :** Nuxt does not auto-discover CSS files. Only files listed
in `nuxt.config.ts > css` are bundled.

**Fix :** Add the path :

```ts
export default defineNuxtConfig({
  css: ["./app/assets/css/main.css"],
  // ...
})
```

The path is relative to the project root, NOT the `app/` directory.

---

## AP-5 : Remix CSS import without `?url`

**Symptom :** Production build fails with `Cannot find module ~/tailwind.css`
or the linked stylesheet is empty.

**Cause :** Without `?url`, Vite returns the CSS as a JS module
(an object with a `default` export). Remix expects a URL string for
`LinksFunction`.

**Fix :** Add `?url` :

```diff
- import stylesheet from "~/tailwind.css"
+ import stylesheet from "~/tailwind.css?url"
```

---

## AP-6 : Importing CSS in Remix root without LinksFunction

**Symptom :** Styles apply in dev but not in production.

**Cause :** Remix splits client and server bundles. A side-effect
`import "./tailwind.css"` in `root.tsx` is dropped from the server
bundle, so the production HTML never includes the `<link>`.

**Fix :** ALWAYS use the LinksFunction pattern :

```tsx
import stylesheet from "~/tailwind.css?url"

export const links: LinksFunction = () => [
  { rel: "stylesheet", href: stylesheet },
]
```

---

## AP-7 : Missing `<Links />` in Remix root

**Symptom :** LinksFunction is defined but no `<link>` tag appears in
the rendered HTML.

**Cause :** Remix only emits link tags when the `<Links />` component
is rendered inside `<head>`.

**Fix :**

```tsx
import { Links } from "@remix-run/react"

export default function App() {
  return (
    <html>
      <head>
        <Links />
      </head>
      <body>...</body>
    </html>
  )
}
```

---

## AP-8 : Missing `vitePreprocess` in SvelteKit v3

**Symptom :** `@apply` inside `<style>` blocks errors with
"Cannot apply unknown utility class" or "Unknown at-rule @tailwind".

**Cause :** Without `vitePreprocess`, Svelte's compiler treats `<style>`
content as raw CSS and never sends it through PostCSS.

**Fix :** `svelte.config.js` :

```js
import { vitePreprocess } from "@sveltejs/kit/vite"

export default {
  preprocess: vitePreprocess(),
  // ...
}
```

ALSO add `lang="postcss"` to style blocks that use `@apply` :

```svelte
<style lang="postcss">
  .card { @apply rounded p-4; }
</style>
```

---

## AP-9 : SvelteKit v4 scoped `@apply` without `@reference`

**Symptom :** Build error : `Cannot apply unknown utility class: bg-blue-500`
inside a Svelte component `<style>` block.

**Cause :** v4 tokens are file-scoped. The scoped style block does not
see the global token registry.

**Fix :** Add `@reference` at the top of the scoped block :

```svelte
<style>
  @reference "../app.css";

  .card { @apply bg-blue-500; }
</style>
```

---

## AP-10 : SvelteKit `+layout.svelte` Svelte 5 syntax vs Svelte 4

**Symptom :** `Cannot read properties of undefined (reading 'children')`
or `<slot />` deprecation warning.

**Cause :** Svelte 5 uses `let { children } = $props()` + `{@render children()}`.
Svelte 4 uses `<slot />`. Mixing them breaks the layout.

**Fix :** Match the syntax to the Svelte major you installed :

Svelte 4 :

```svelte
<script>
  import "../app.css"
</script>

<slot />
```

Svelte 5 :

```svelte
<script>
  import "../app.css"
  let { children } = $props()
</script>

{@render children()}
```

Check `package.json` for `svelte` version : 4.x or 5.x.

---

## AP-11 : Forgetting to import `app.css` in SvelteKit layout

**Symptom :** Files exist, Vite config is correct, but no Tailwind
classes apply anywhere.

**Cause :** SvelteKit only bundles CSS that is imported from a
component tree. `+layout.svelte` is the canonical place because every
route renders inside the root layout.

**Fix :** `src/routes/+layout.svelte` :

```svelte
<script>
  import "../app.css"
</script>

<slot />
```

NEVER import `app.css` from individual pages : duplicate Tailwind
emission per route.

---

## AP-12 : Astro layout file does not import the entry CSS

**Symptom :** v4 entry CSS exists at `src/styles/global.css`, Vite
plugin is registered, but no styles apply.

**Cause :** Astro does not auto-include CSS files. They MUST be
imported from a layout or page.

**Fix :** In `src/layouts/Layout.astro` :

```astro
---
import "../styles/global.css"
---
```

If no layout is in use, import directly from each page :

```astro
---
import "../styles/global.css"
---
<h1 class="text-3xl">...</h1>
```

---

## AP-13 : Nuxt v3 module + manual postcss config

**Symptom :** Duplicate styles, conflicting config sources, or
HMR-related stylesheet flicker.

**Cause :** `@nuxtjs/tailwindcss` already wires PostCSS. Adding a
manual `postcss.config.js` or installing `tailwindcss` separately
creates two pipelines.

**Fix :** Remove the manual setup. Let the module own the pipeline :

```bash
rm postcss.config.js
npm uninstall tailwindcss postcss autoprefixer
```

Keep only `@nuxtjs/tailwindcss` in `devDependencies` ; the module
manages transitive deps.

---

## AP-14 : Nuxt v4 with `<style scoped>` `@apply` without `@reference`

**Symptom :** Same as AP-9 but in Vue SFCs : `Cannot apply unknown
utility class`.

**Fix :** Same pattern :

```vue
<style scoped>
@reference "~/assets/css/main.css";

.card { @apply bg-blue-500; }
</style>
```

The `~/` alias resolves to the Nuxt app root.

---

## AP-15 : Mixing v3 and v4 in the same project

**Symptom :** Some utilities work, others do not. Errors about unknown
directives or "Cannot apply unknown utility class".

**Cause :** Two Tailwind majors are resolved (e.g. `tailwindcss@3` in
`devDependencies` and `tailwindcss@4` pulled in transitively by
`@tailwindcss/vite`).

**Fix :** Inspect with :

```bash
npm ls tailwindcss
```

ALWAYS resolve to a single major. Remove the stale one :

```bash
npm uninstall tailwindcss
npm install tailwindcss@4   # or @3
```

---

## AP-16 : Astro v3 manual install (skipping `astro add`)

**Symptom :** Integration not loaded, base styles missing, content
paths not picked up.

**Cause :** Manually adding `@astrojs/tailwind` to `astro.config.mjs`
without running `astro add` skips the post-install step that wires
Astro's content-discovery hooks.

**Fix :** ALWAYS use `npx astro add tailwind` for v3. If you already
manually added it, run `astro add tailwind` again to re-trigger
the wiring step.

---

## AP-17 : Remix Vite plugin position

**Symptom :** Remix HMR breaks, or CSS is not injected on dev reload.

**Cause :** `tailwindcss()` and `remix()` order matters for Remix's
internal asset pipeline.

**Fix :** Per official Remix docs, register Tailwind FIRST :

```ts
plugins: [tailwindcss(), remix()]
```

The reverse order can leave Tailwind's emitted asset out of Remix's
manifest.

---

## AP-18 : Loading two CSS frameworks side by side

**Symptom :** Conflicting resets, layout shifts, double margin / padding
on default elements.

**Cause :** Tailwind's preflight + Bootstrap / Bulma reset both run.

**Fix :** Disable Tailwind's preflight in v3 (`corePlugins: { preflight: false }`)
or scope it in v4 :

```css
@layer base {
  /* opt out of preflight selectively */
}
```

Better : pick ONE framework. NEVER ship two reset stylesheets in production.

---

## AP-19 : Stale dev server after framework config edit

**Symptom :** Edits to `astro.config.mjs`, `nuxt.config.ts`,
`svelte.config.js`, or `vite.config.ts` do not take effect.

**Cause :** Framework config files load once at server start. HMR does
not propagate config changes.

**Fix :** Restart the dev server :

```bash
# Ctrl-C
npm run dev
```

This is expected behavior, NOT a Tailwind bug.

---

## AP-20 : Browser caches old CSS in production

**Symptom :** Deployed site shows old styles after a Tailwind config
change ; local dev shows the new ones.

**Cause :** CDN cache or service worker holds the previous CSS asset.

**Fix :** Frameworks emit hashed asset filenames in production
(`assets/main.abc123.css`), so a fresh build invalidates the cache.
If hashing is disabled, re-enable it. If a service worker is
registered, bump its version. Hard-reload (Ctrl-Shift-R) to confirm
the issue is cache-only.
