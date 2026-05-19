# Anti-Patterns : Tailwind + Vite

Every known wiring mistake, the symptom you see, the cause, and the fix.

---

## AP-1 : Mixing @tailwindcss/vite and @tailwindcss/postcss in v4

**Symptom :** Slow dev start, occasional double-processing of at-rules,
mysterious `[postcss]` errors about unknown directives.

**Cause :** Both plugins transform CSS. They are mutually exclusive in
a Vite project. The Vite-native plugin already integrates with PostCSS
where needed.

**Fix :** Pick one. ALWAYS prefer `@tailwindcss/vite` for v4 :

```bash
npm uninstall @tailwindcss/postcss
```

Remove `tailwindcss` and `@tailwindcss/postcss` from `postcss.config.js`.
For v4 + Vite, you do not need a `postcss.config.js` at all.

---

## AP-2 : Using @tailwind base/components/utilities in v4

**Symptom :** No utilities are emitted. The CSS file produced is empty
or contains only reset styles.

**Cause :** The three `@tailwind` directives are v3 syntax. v4 ignores
them silently.

**Fix :** Replace all three with a single `@import "tailwindcss"` :

```diff
- @tailwind base;
- @tailwind components;
- @tailwind utilities;
+ @import "tailwindcss";
```

---

## AP-3 : Using @import "tailwindcss" in v3

**Symptom :** PostCSS leaves the `@import` line as-is in output. No
utilities are produced. Browser may try to load `tailwindcss.css` as
a file and 404.

**Cause :** v3 has no `tailwindcss` import target. That syntax was
introduced in v4.

**Fix :** Use the three v3 directives :

```diff
- @import "tailwindcss";
+ @tailwind base;
+ @tailwind components;
+ @tailwind utilities;
```

---

## AP-4 : v3 missing or wrong `content` paths

**Symptom :** Utility classes appear in the HTML but no CSS is
generated for them. Inspecting computed styles shows the property
is unset.

**Cause :** The v3 JIT engine ONLY scans files matched by `content`.
Anything outside is invisible.

**Fix :** Add every template directory and extension you use :

```js
content: [
  "./index.html",
  "./src/**/*.{js,ts,jsx,tsx,vue,svelte,astro,mdx}",
]
```

If you use `.html` partials, `.md` files, or any other extension that
authors classes, list it explicitly.

---

## AP-5 : v4 monorepo without @source

**Symptom :** Classes used in workspace packages (`packages/ui`,
`libs/shared`) work locally inside the package's own dev environment
but disappear when consumed from the app.

**Cause :** v4 auto-scans only files inside the Vite `root`. Workspace
packages, even when symlinked, are outside that root.

**Fix :** Add an `@source` directive in the consuming app's entry CSS :

```css
@import "tailwindcss";
@source "../../packages/ui/src/**/*.{ts,tsx}";
```

The path is relative to the entry CSS file.

---

## AP-6 : Dynamic class name construction

**Symptom :** Class names assembled from variables (`bg-${color}-500`)
do not produce CSS.

**Cause :** The scanner is a static analyzer. It sees raw strings, not
runtime concatenations.

**Fix (v3) :** Add to `safelist` :

```js
safelist: ["bg-red-500", "bg-green-500", "bg-blue-500"]
```

Or with a pattern :

```js
safelist: [{ pattern: /bg-(red|green|blue)-500/ }]
```

**Fix (v4) :** Use `@source inline` brace expansion :

```css
@source inline "{bg-red-500,bg-green-500,bg-blue-500}";
```

Better : write the full class names somewhere the scanner can see them,
e.g. an object literal :

```ts
const colors = {
  red: "bg-red-500",
  green: "bg-green-500",
  blue: "bg-blue-500",
}
```

The scanner sees each value as a literal string.

---

## AP-7 : @apply inside Vue/Svelte scoped styles in v4

**Symptom :** Build error : `Cannot apply unknown utility class: bg-blue-500`.

**Cause :** v4 tokens are file-scoped. Inside a `<style scoped>` block,
the scoped CSS is processed in isolation and cannot see the global
token registry.

**Fix :** Add `@reference` to the entry CSS at the top of the scoped block :

```vue
<style scoped>
@reference "../style.css";

.btn { @apply bg-blue-500; }
</style>
```

v3 does NOT have this issue ; `@apply` resolves against the global
PostCSS context.

---

## AP-8 : `postcss-import` order trap (v3)

**Symptom :** Custom CSS imports fail with `Tailwind functions cannot be
resolved` or similar.

**Cause :** `postcss-import` MUST run before `tailwindcss`. If listed
after, Tailwind processes raw `@import` syntax and bails.

**Fix :** Order in `postcss.config.js` :

```js
export default {
  plugins: {
    "postcss-import": {},   // first
    tailwindcss: {},
    autoprefixer: {},       // last
  },
}
```

---

## AP-9 : Forgetting to import the entry CSS

**Symptom :** Browser DevTools shows the HTML but no stylesheet linked.
No Tailwind classes apply.

**Cause :** The entry CSS file exists but is never `import`-ed into the
JS entry, and `index.html` does not `<link>` it.

**Fix :** Either link in HTML :

```html
<link rel="stylesheet" href="/src/style.css">
```

Or import in JS entry :

```ts
// main.ts (vanilla / vue / svelte / solid) or main.tsx (react)
import "./style.css"
```

---

## AP-10 : Pinning to broken v4.0.8 in Astro

**Symptom :** After bumping to `tailwindcss@4.0.8`, utilities used in
a private component package stop applying. Issue 16733.

**Cause :** Regression in 4.0.8 affecting how component-package CSS is parsed.

**Fix :** Pin to 4.0.7 OR upgrade past the patched release :

```json
"dependencies": {
  "tailwindcss": "4.0.7",
  "@tailwindcss/vite": "4.0.7"
}
```

ALWAYS run `npm run build` after bumping to confirm utilities still emit.

---

## AP-11 : Chasing Turbopack issue 19825 in Vite

**Symptom :** Arbitrary-value classes like `aspect-[12/5]`, `z-[100]`,
`h-[80vh]` do not produce CSS, and you find the Turbopack bug ticket.

**Cause :** Issue 19825 is exclusive to Next.js + Turbopack. Vite users
are NOT affected by that bug.

**Fix :** Look elsewhere :

1. v4 in a monorepo : check `@source` directives.
2. v3 : check `content` covers the file with the arbitrary value.
3. The arbitrary value syntax may be invalid : `aspect-[12 / 5]` (with
   spaces) is rejected. Use `aspect-[12/5]`.

---

## AP-12 : Wrong vite.config plugin position (Vue, Svelte)

**Symptom :** `@apply` in scoped styles produces unparseable output, or
component styles are duplicated.

**Cause :** `tailwindcss()` runs before the framework plugin extracted
the scoped style block.

**Fix :** ALWAYS register the framework plugin first :

```ts
plugins: [
  vue(),         // or svelte()
  tailwindcss(),
]
```

Order matters only when frameworks rewrite CSS blocks (Vue SFC, Svelte
SFC). For React, SolidJS, Qwik, Astro, order does not matter.

---

## AP-13 : Setting `optimizeDeps.exclude: ["tailwindcss"]`

**Symptom :** Dev start is slower than expected, or HMR feels sluggish.

**Cause :** Someone copy-pasted advice from a v2-era guide. Tailwind
is not a runtime dependency in v3 or v4 ; excluding it from
optimizeDeps is meaningless.

**Fix :** Remove the entry :

```diff
- optimizeDeps: {
-   exclude: ["tailwindcss"],
- },
```

---

## AP-14 : Generating CSS via the standalone `tailwindcss` CLI alongside the Vite plugin

**Symptom :** Two `.css` files in `dist/`, duplicated utilities, or
the dev server serves a stale build.

**Cause :** Running `npx tailwindcss -o dist/extra.css --watch` in
parallel with the Vite dev server produces conflicting outputs.

**Fix :** Pick one path. For Vite projects, the plugin replaces the
CLI completely. Remove any `tailwindcss -i ... -o ...` scripts from
`package.json`.

---

## AP-15 : Editing `tailwind.config.js` and expecting HMR (v3)

**Symptom :** New theme tokens, content paths, or plugins do not take
effect until you restart the dev server.

**Cause :** v3 loads `tailwind.config.js` once at engine start. HMR is
limited to template files and entry CSS.

**Fix :** Restart the dev server after editing the config :

```bash
# Ctrl-C
npm run dev
```

This is expected v3 behavior, not a bug. v4 with `@config` re-invalidates
the module on change.

---

## AP-16 : Mixing v3 and v4 documentation

**Symptom :** Copy-paste from one doc set produces silent failures
(empty CSS, parse errors).

**Cause :** v3 and v4 share almost no install-time syntax. Mixing the
two is the most common cause of "Tailwind broke after upgrade" reports.

**Fix :** ALWAYS verify the doc URL :

- v4 : `tailwindcss.com` (current main site)
- v3 : `v3.tailwindcss.com` (versioned subdomain)

When in doubt, read the package.json `tailwindcss` version first,
then open the matching doc set.

---

## AP-17 : Adding both `@tailwind` and `@import "tailwindcss"`

**Symptom :** Parse errors, or v4 ignores the file entirely.

**Cause :** Someone migrated a v3 project but kept the old directives
"to be safe."

**Fix :** Pick the version. Remove the directives that do not match :

```diff
- @tailwind base;
- @tailwind components;
- @tailwind utilities;
  @import "tailwindcss";
```

---

## AP-18 : Putting `content` in v4

**Symptom :** No error, but the v4 scanner appears to ignore the entries.

**Cause :** v4 stopped reading `tailwind.config.js` by default. The
`content` field is silently ignored.

**Fix :** Either drop `tailwind.config.js` entirely, or opt-in via
`@config` in the entry CSS :

```css
@import "tailwindcss";
@config "./tailwind.config.js";
```

This is only useful for migrating legacy v3 configs. New v4 projects
SHOULD use `@source` instead.

---

## AP-19 : Forgetting `darkMode` strategy mismatch

**Symptom :** `dark:` variants do not toggle when adding/removing a
`dark` class on the root.

**Cause (v3) :** Default `darkMode` is `media`. The variant follows the
OS preference, not a class.

**Fix (v3) :** Switch to class strategy in `tailwind.config.js` :

```js
export default {
  darkMode: "class",
  // ...
}
```

**v4 equivalent :** Use the `@custom-variant` directive in CSS :

```css
@import "tailwindcss";
@custom-variant dark (&:where(.dark, .dark *));
```

See `tailwind-syntax-dark-mode` for the full pattern.

---

## AP-20 : Browser caches old CSS

**Symptom :** Edits to the entry CSS appear not to take effect.

**Cause :** Service worker, CDN, or aggressive browser caching is
serving the previous build.

**Fix :** Hard-reload (Ctrl-Shift-R / Cmd-Shift-R). If a service worker
is registered for dev, unregister it. Vite emits hashed filenames in
production builds, so this is rarely a build-time issue.
