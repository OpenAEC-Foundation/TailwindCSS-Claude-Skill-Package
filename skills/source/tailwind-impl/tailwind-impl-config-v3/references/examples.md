# tailwind-impl-config-v3 : Examples

Concrete `tailwind.config.js` recipes for common project shapes.

## Bare Minimum (Vite + plain HTML)

```js
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./index.html", "./src/**/*.{js,ts,jsx,tsx}"],
  theme: { extend: {} },
  plugins: [],
};
```

## Next.js 14+ (App + Pages Router)

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./app/**/*.{js,ts,jsx,tsx,mdx}",
    "./pages/**/*.{js,ts,jsx,tsx,mdx}",
    "./components/**/*.{js,ts,jsx,tsx}",
  ],
  theme: { extend: {} },
  plugins: [require("@tailwindcss/typography"), require("@tailwindcss/forms")],
};
```

## ESM + TypeScript

```ts
// tailwind.config.ts
import type { Config } from "tailwindcss";
import typography from "@tailwindcss/typography";

const config: Config = {
  content: ["./src/**/*.{ts,tsx,vue}"],
  darkMode: ["class", '[data-theme="dark"]'],
  theme: {
    extend: {
      colors: {
        brand: {
          50: "#eff6ff",
          500: "#3b82f6",
          900: "#1e3a8a",
        },
      },
    },
  },
  plugins: [typography],
};

export default config;
```

## Monorepo Workspace

```js
// apps/web/tailwind.config.js
const sharedPreset = require("@my-org/tailwind-preset");

/** @type {import('tailwindcss').Config} */
module.exports = {
  presets: [sharedPreset],
  content: [
    "./src/**/*.{ts,tsx}",
    "../../packages/ui/src/**/*.{ts,tsx}",
    "../../packages/shared/src/**/*.{ts,tsx}",
  ],
};
```

```js
// packages/tailwind-preset/index.js (the shared preset)
/** @type {import('tailwindcss').Config} */
module.exports = {
  theme: {
    extend: {
      colors: { brand: { 500: "#3b82f6" } },
      fontFamily: { sans: ["Inter Variable", "sans-serif"] },
    },
  },
  plugins: [require("@tailwindcss/forms")],
};
```

## Vue 3 (Vite)

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./index.html", "./src/**/*.{vue,js,ts}"],
  theme: { extend: {} },
  plugins: [],
};
```

## Astro

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./src/**/*.{astro,html,md,mdx,js,ts,jsx,tsx,vue,svelte}"],
  theme: { extend: {} },
  plugins: [],
};
```

## SvelteKit

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./src/**/*.{html,js,ts,svelte}"],
  theme: { extend: {} },
  plugins: [],
};
```

## Dark Mode : Class Strategy (Manual Toggle)

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./src/**/*.{ts,tsx}"],
  darkMode: "class",
};
```

```ts
// dark-mode-toggle.ts
function toggleDark() {
  document.documentElement.classList.toggle("dark");
  localStorage.theme = document.documentElement.classList.contains("dark") ? "dark" : "light";
}

// Initial load
if (localStorage.theme === "dark" || (!("theme" in localStorage) && window.matchMedia("(prefers-color-scheme: dark)").matches)) {
  document.documentElement.classList.add("dark");
}
```

## Dark Mode : Attribute Selector (v3.4.1+)

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  darkMode: ["class", '[data-theme="dark"]'],
};
```

## Custom Plugin (Component Class)

```js
const plugin = require("tailwindcss/plugin");

/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./src/**/*.tsx"],
  theme: {
    extend: {
      colors: { brand: { 500: "#3b82f6" } },
    },
  },
  plugins: [
    plugin(({ addComponents, theme }) => {
      addComponents({
        ".btn-primary": {
          padding: `${theme("spacing.2")} ${theme("spacing.4")}`,
          backgroundColor: theme("colors.brand.500"),
          color: theme("colors.white"),
          borderRadius: theme("borderRadius.md"),
          "&:hover": {
            backgroundColor: theme("colors.brand.600", theme("colors.brand.500")),
          },
        },
      });
    }),
  ],
};
```

## Custom Plugin (Value-Bearing Utility)

```js
const plugin = require("tailwindcss/plugin");

module.exports = {
  plugins: [
    plugin(({ matchUtilities, theme }) => {
      matchUtilities(
        { tab: (value) => ({ tabSize: value }) },
        { values: theme("tabSize") }
      );
    }),
  ],
  theme: {
    tabSize: { 1: "1", 2: "2", 4: "4", 8: "8" },
  },
};
```

## Safelist with Pattern + Variants

```js
module.exports = {
  content: ["./src/**/*.tsx"],
  safelist: [
    "bg-red-500",
    "bg-blue-500",
    { pattern: /bg-(red|green|blue)-(100|500|900)/ },
    { pattern: /text-.+-(500|700|900)/, variants: ["hover", "dark"] },
  ],
};
```

Mostly used when class names are built from runtime data and the scanner cannot detect them.

## Disabling Core Utilities

```js
module.exports = {
  corePlugins: {
    preflight: false,            // disable the CSS reset
    float: false,                // disable float-* utilities
    container: false,            // disable .container component
  },
};
```

## Prefix + Important

```js
module.exports = {
  prefix: "tw-",                   // tw-flex, tw-bg-red-500, ...
  important: true,                 // all utilities get !important
  // OR scope important to a selector :
  // important: "#app",
};
```

## Future Flags

```js
module.exports = {
  future: {
    hoverOnlyWhenSupported: true,  // hover: only fires on devices with hover
  },
};
```

In v4 this became the default.

## Verified Sources

- https://v3.tailwindcss.com/docs/configuration
- https://v3.tailwindcss.com/docs/content-configuration
- https://v3.tailwindcss.com/docs/plugins
- https://v3.tailwindcss.com/docs/presets

Last verified : 2026-05-19.
