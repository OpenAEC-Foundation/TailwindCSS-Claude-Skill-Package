# tailwind-impl-config-v3 : Methods Reference

Complete API surface for the v3.4 `tailwind.config.js` configuration object.

## File Variants

| File | Module system | When to use |
|------|---------------|-------------|
| `tailwind.config.js` | CommonJS (`module.exports = {}`) | Default. Works with any Node project. |
| `tailwind.config.mjs` | ESM (`export default {}`) | When `package.json` has `"type": "module"`. |
| `tailwind.config.ts` | TypeScript with type import | TS-first projects ; requires `ts-node` or compiler. |

```ts
// tailwind.config.ts
import type { Config } from "tailwindcss";

export default {
  content: ["./src/**/*.{ts,tsx,html}"],
} satisfies Config;
```

## `content` Option

| Form | Example | Notes |
|------|---------|-------|
| Array of globs | `content: ["./src/**/*.{js,ts,jsx,tsx,html,vue,svelte}"]` | Most common. |
| Object form | `content: { files: ["./src/**/*.html"], relative: true, transform: { vue: c => c.replace(/<style>.*?<\/style>/gs, "") }, extract: { html: c => c.match(/\bclass="([^"]+)"/g) } }` | Advanced extractors. |
| Glob from outside CWD | `["../shared/**/*.tsx"]` | Monorepo. |
| node_modules path | `["../node_modules/@my-org/ui/**/*.{js,html}"]` | Explicitly allowed ; default scanner skips `node_modules`. |

Common patterns :

```js
// Next.js App Router + Pages Router
content: [
  "./app/**/*.{js,ts,jsx,tsx,mdx}",
  "./pages/**/*.{js,ts,jsx,tsx,mdx}",
  "./components/**/*.{js,ts,jsx,tsx}",
],

// Vue 3
content: [
  "./index.html",
  "./src/**/*.{vue,js,ts}",
],

// Astro
content: [
  "./src/**/*.{astro,html,md,mdx,js,ts,jsx,tsx,vue,svelte}",
],

// Monorepo workspace
content: [
  "./app/**/*.tsx",
  "../packages/ui/**/*.tsx",
],
```

## `theme` and `theme.extend`

`theme` REPLACES the entire default for that namespace. `theme.extend` ADDS to defaults.

```js
theme: {
  colors: { white: "#fff", brand: "#3b82f6" },  // DROPS all default colors
  extend: {
    colors: { brand: { 500: "#3b82f6" } },      // KEEPS defaults, adds brand
    spacing: { 128: "32rem" },                  // adds 128 alongside 0..96 defaults
    fontFamily: { display: ["Inter", "sans-serif"] },
  },
},
```

### Theme namespaces (full list)

| Namespace | Powers utilities |
|-----------|------------------|
| `colors` | `bg-*`, `text-*`, `border-*`, `from-*`, `via-*`, `to-*`, `ring-*`, `divide-*`, `outline-*`, `accent-*`, `caret-*`, `fill-*`, `stroke-*` |
| `spacing` | `p-*`, `m-*`, `gap-*`, `w-*`, `h-*`, `inset-*`, `space-x-*`, `space-y-*`, `translate-*` |
| `fontFamily` | `font-sans`, `font-serif`, `font-mono`, custom |
| `fontSize` | `text-xs` through `text-9xl` + custom |
| `fontWeight` | `font-thin` through `font-black` |
| `lineHeight` | `leading-*` |
| `letterSpacing` | `tracking-*` |
| `screens` | `sm:`, `md:`, `lg:`, `xl:`, `2xl:`, custom |
| `borderRadius` | `rounded-*` |
| `boxShadow` | `shadow-*` |
| `zIndex` | `z-*` |
| `opacity` | `opacity-*` |
| `transitionDuration` | `duration-*` |
| `transitionTimingFunction` | `ease-*` |
| `keyframes` + `animation` | `animate-*` |
| `aspectRatio` | `aspect-*` |
| `container` | `container` component config |

## `presets`

Shared base configs. Multiple presets merge in order ; later overrides earlier.

```js
const base = require("@my-org/tailwind-preset");
const brand = require("@my-org/tailwind-brand");

module.exports = {
  presets: [base, brand],
  theme: { extend: { /* project-specific overrides */ } },
};
```

## `plugins`

Array of plugin functions or factory results.

```js
const plugin = require("tailwindcss/plugin");

module.exports = {
  plugins: [
    require("@tailwindcss/typography"),
    require("@tailwindcss/forms"),
    plugin(({ addUtilities, addComponents, matchUtilities, theme, addVariant }) => {
      addUtilities({
        ".skew-10deg": { transform: "skewY(-10deg)" },
      });
      addComponents({
        ".btn": {
          padding: theme("spacing.2") + " " + theme("spacing.4"),
          borderRadius: theme("borderRadius.md"),
        },
      });
      addVariant("optional", "&:optional");
      matchUtilities(
        { tab: (value) => ({ tabSize: value }) },
        { values: theme("tabSize") }
      );
    }),
  ],
};
```

Plugin helpers (all signatures) :

| Helper | Purpose |
|--------|---------|
| `addUtilities(utilities, options?)` | Add static utilities |
| `matchUtilities(utilities, options?)` | Add value-bearing utilities (e.g. `tab-4`) |
| `addComponents(components, options?)` | Add component classes |
| `matchComponents(components, options?)` | Add value-bearing components |
| `addBase(styles)` | Add base styles |
| `addVariant(name, selector)` | Add a variant |
| `matchVariant(name, fn, options?)` | Add a value-bearing variant |
| `theme(path, defaultValue?)` | Read from resolved theme |
| `config(path, defaultValue?)` | Read from full config |
| `e(className)` | Escape a class name |
| `corePlugins(name)` | Check if a core plugin is enabled |

## `darkMode`

| Setting | Strategy |
|---------|----------|
| `'media'` (default) | Respect `prefers-color-scheme: dark` |
| `'class'` | Activate `dark:` when `.dark` is on any ancestor |
| `['class', '[data-theme="dark"]']` (v3.4.1+) | Custom selector (attribute, etc.) |
| `['variant', ['&:where(.dark, .dark *)', '&:where([data-theme=dark], [data-theme=dark] *)']]` | Multiple selectors |

```js
darkMode: ['class', '[data-theme="dark"]'],
```

## `corePlugins` (REMOVED in v4)

Disable individual built-in utilities.

```js
corePlugins: { float: false, clear: false }                // disable specific
corePlugins: ['preflight', 'container']                    // allowlist (only these)
```

## `safelist` (REMOVED in v4)

Force-include classes that the content scanner cannot find.

```js
safelist: [
  "bg-red-500",
  { pattern: /bg-(red|blue|green)-(100|500|900)/ },
  { pattern: /text-.*-(500|700|900)/, variants: ["hover", "focus"] },
],
```

## Other Options

| Option | Default | Effect |
|--------|---------|--------|
| `blocklist` | `[]` | Class names to SKIP even if found |
| `prefix` | `""` | Prefix every utility (`tw-flex`) |
| `important` | `false` | `true` or selector string `"#app"` |
| `separator` (REMOVED in v4) | `":"` | Variant-utility separator |
| `future` | `{}` | Opt-in to next-major behaviour (e.g. `hoverOnlyWhenSupported: true`) |
| `experimental` | `{}` | Unstable opt-in flags |

## Verified Sources

- https://v3.tailwindcss.com/docs/configuration
- https://v3.tailwindcss.com/docs/content-configuration
- https://v3.tailwindcss.com/docs/theme
- https://v3.tailwindcss.com/docs/plugins
- https://v3.tailwindcss.com/docs/dark-mode
- https://v3.tailwindcss.com/docs/presets

Last verified : 2026-05-19.
