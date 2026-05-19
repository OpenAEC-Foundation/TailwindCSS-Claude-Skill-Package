# Reference : Tailwind + Vite Examples

Complete, copy-paste-ready setups. Each example shows the **minimum**
files needed for a working Tailwind + Vite project.

Every example assumes you have already run `npm create vite@latest`
with the relevant template.

---

## Example 1 : Vanilla Vite + Tailwind v4

`package.json` (dependencies excerpt) :

```json
{
  "devDependencies": {
    "vite": "^5.0.0"
  },
  "dependencies": {
    "tailwindcss": "^4.0.0",
    "@tailwindcss/vite": "^4.0.0"
  }
}
```

`vite.config.ts` :

```ts
import { defineConfig } from "vite"
import tailwindcss from "@tailwindcss/vite"

export default defineConfig({
  plugins: [tailwindcss()],
})
```

`src/style.css` :

```css
@import "tailwindcss";
```

`index.html` :

```html
<!doctype html>
<html>
  <head>
    <link rel="stylesheet" href="/src/style.css">
  </head>
  <body>
    <h1 class="text-3xl font-bold underline">Hello Tailwind</h1>
  </body>
</html>
```

Run : `npm run dev`.

---

## Example 2 : Vanilla Vite + Tailwind v3

`package.json` excerpt :

```json
{
  "devDependencies": {
    "vite": "^5.0.0",
    "tailwindcss": "^3.4.0",
    "postcss": "^8.4.0",
    "autoprefixer": "^10.4.0"
  }
}
```

Generated via `npx tailwindcss init -p` :

`tailwind.config.js` :

```js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts}",
  ],
  theme: { extend: {} },
  plugins: [],
}
```

`postcss.config.js` :

```js
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

`src/index.css` :

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

`src/main.ts` :

```ts
import "./index.css"
```

---

## Example 3 : React + Vite + Tailwind v4

`vite.config.ts` :

```ts
import { defineConfig } from "vite"
import react from "@vitejs/plugin-react"
import tailwindcss from "@tailwindcss/vite"

export default defineConfig({
  plugins: [react(), tailwindcss()],
})
```

`src/index.css` :

```css
@import "tailwindcss";
```

`src/main.tsx` :

```tsx
import React from "react"
import ReactDOM from "react-dom/client"
import App from "./App"
import "./index.css"

ReactDOM.createRoot(document.getElementById("root")!).render(<App />)
```

---

## Example 4 : React + Vite + Tailwind v3

`vite.config.ts` :

```ts
import { defineConfig } from "vite"
import react from "@vitejs/plugin-react"

export default defineConfig({
  plugins: [react()],
})
```

`tailwind.config.js` :

```js
export default {
  content: ["./index.html", "./src/**/*.{ts,tsx}"],
  theme: { extend: {} },
  plugins: [],
}
```

`postcss.config.js` and `src/index.css` : same as Example 2.

`src/main.tsx` :

```tsx
import "./index.css"
```

---

## Example 5 : Vue 3 + Vite + Tailwind v4 (scoped @apply)

`vite.config.ts` :

```ts
import { defineConfig } from "vite"
import vue from "@vitejs/plugin-vue"
import tailwindcss from "@tailwindcss/vite"

export default defineConfig({
  plugins: [vue(), tailwindcss()],
})
```

`src/style.css` :

```css
@import "tailwindcss";
```

`src/App.vue` :

```vue
<template>
  <button class="btn-primary">Click</button>
</template>

<style scoped>
@reference "./style.css";

.btn-primary {
  @apply bg-blue-500 text-white px-4 py-2 rounded;
}
</style>
```

The `@reference` line is REQUIRED in v4 for `@apply` inside scoped
styles. Without it, the design tokens are not in scope and `@apply`
errors with "Cannot apply unknown utility class".

---

## Example 6 : Svelte + Vite + Tailwind v4

`vite.config.ts` :

```ts
import { defineConfig } from "vite"
import { svelte } from "@sveltejs/vite-plugin-svelte"
import tailwindcss from "@tailwindcss/vite"

export default defineConfig({
  plugins: [svelte(), tailwindcss()],
})
```

`src/app.css` :

```css
@import "tailwindcss";
```

`src/App.svelte` :

```svelte
<script>
  // ...
</script>

<h1 class="text-3xl font-bold">Hello</h1>

<style>
  @reference "./app.css";

  h1 {
    @apply text-blue-600;
  }
</style>
```

`src/main.ts` :

```ts
import "./app.css"
import App from "./App.svelte"
new App({ target: document.getElementById("app")! })
```

---

## Example 7 : SolidJS + Vite + Tailwind v4

```ts
// vite.config.ts
import { defineConfig } from "vite"
import solid from "vite-plugin-solid"
import tailwindcss from "@tailwindcss/vite"

export default defineConfig({
  plugins: [solid(), tailwindcss()],
})
```

```css
/* src/index.css */
@import "tailwindcss";
```

```tsx
/* src/index.tsx */
import { render } from "solid-js/web"
import App from "./App"
import "./index.css"

render(() => <App />, document.getElementById("root")!)
```

---

## Example 8 : Astro + Tailwind v4 (Astro 5.x)

`package.json` excerpt :

```json
{
  "dependencies": {
    "astro": "^5.3.0",
    "tailwindcss": "4.0.7",
    "@tailwindcss/vite": "4.0.7"
  }
}
```

ALWAYS pin to 4.0.7 (or a release past the issue 16733 fix) to avoid
the 4.0.8 Astro regression where component-package utilities stopped applying.

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

`src/styles/global.css` :

```css
@import "tailwindcss";
```

`src/layouts/Layout.astro` :

```astro
---
import "../styles/global.css"
---
<html>
  <body>
    <slot />
  </body>
</html>
```

---

## Example 9 : Qwik + Vite + Tailwind v4

```ts
// vite.config.ts
import { defineConfig } from "vite"
import { qwikVite } from "@builder.io/qwik/optimizer"
import tailwindcss from "@tailwindcss/vite"

export default defineConfig({
  plugins: [qwikVite(), tailwindcss()],
})
```

```css
/* src/global.css */
@import "tailwindcss";
```

```tsx
/* src/root.tsx */
import "./global.css"
```

---

## Example 10 : pnpm monorepo + Tailwind v4

Workspace layout :

```
repo/
├── pnpm-workspace.yaml
├── apps/
│   └── web/                  (Vite app)
│       ├── vite.config.ts
│       └── src/style.css
└── packages/
    └── ui/                   (shared components)
        └── src/Button.tsx
```

`apps/web/vite.config.ts` :

```ts
import { defineConfig } from "vite"
import tailwindcss from "@tailwindcss/vite"

export default defineConfig({
  plugins: [tailwindcss()],
})
```

`apps/web/src/style.css` :

```css
@import "tailwindcss";
@source "../../../packages/ui/src/**/*.{ts,tsx}";
```

Without the `@source` line, classes used inside `packages/ui` are
NEVER scanned. The v4 plugin restricts auto-detection to the Vite root.

---

## Example 11 : pnpm monorepo + Tailwind v3

`apps/web/tailwind.config.js` :

```js
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{ts,tsx}",
    "../../packages/ui/src/**/*.{ts,tsx}",
  ],
  theme: { extend: {} },
  plugins: [],
}
```

v3 uses `content` paths instead of `@source`. Both work the same in
practice : list every directory that authors Tailwind classes.

---

## Example 12 : Library mode + Tailwind v4

`vite.config.ts` :

```ts
import { defineConfig } from "vite"
import tailwindcss from "@tailwindcss/vite"

export default defineConfig({
  plugins: [tailwindcss()],
  build: {
    lib: {
      entry: "src/index.ts",
      formats: ["es"],
      fileName: "index",
    },
    rollupOptions: {
      external: ["react"],
    },
    cssCodeSplit: false,
  },
})
```

`src/index.ts` :

```ts
import "./styles.css"
export * from "./components"
```

`src/styles.css` :

```css
@import "tailwindcss";
```

`package.json` exports :

```json
{
  "exports": {
    ".": "./dist/index.js",
    "./style.css": "./dist/index.css"
  }
}
```

Consumers import the stylesheet themselves :

```ts
import "@your-lib/style.css"
```

---

## Example 13 : v4 with safelist for dynamic class names

`src/style.css` :

```css
@import "tailwindcss";

@source inline "{bg-red-500,bg-green-500,bg-blue-500}";
```

This guarantees `bg-red-500`, `bg-green-500`, `bg-blue-500` are
emitted even if the JIT scanner does not see them as literals.

Useful for class names assembled at runtime :

```ts
const color = "red"
const className = `bg-${color}-500`  // scanner cannot see this
```

---

## Example 14 : v3 safelist equivalent

`tailwind.config.js` :

```js
export default {
  content: ["./src/**/*.{ts,tsx}"],
  safelist: [
    "bg-red-500",
    "bg-green-500",
    "bg-blue-500",
  ],
  theme: { extend: {} },
  plugins: [],
}
```

For a pattern :

```js
safelist: [
  { pattern: /bg-(red|green|blue)-(100|500|900)/ },
]
```

---

## Example 15 : Verifying the build

After install, drop this snippet into your entry HTML or root component
to verify all major scanner paths work :

```html
<div class="text-3xl font-bold underline">utility</div>
<div class="text-[#ff00aa] h-[42vh]">arbitrary value</div>
<div class="md:text-lg dark:bg-gray-800">variant</div>
<div class="aspect-[12/5] z-[100]">complex arbitrary</div>
```

If all four render with the intended styles, the entire chain works.
If only some render, see `anti-patterns.md` to diagnose which scanner
boundary is wrong.
