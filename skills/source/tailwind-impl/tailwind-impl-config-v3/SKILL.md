---
name: tailwind-impl-config-v3
description: >
  Use when configuring a Tailwind CSS v3.4 project: writing the `tailwind.config.js`
  / `.mjs` / `.ts` file, setting up content scanning globs, extending or replacing
  the default theme, registering plugins, picking a dark-mode strategy, disabling
  core utilities, safelisting classes that string-concatenation produces, or
  applying a project-wide prefix. Use also when reading an existing v3 config and
  needing to know the precise semantics of `theme` vs `theme.extend`, what
  `corePlugins`, `safelist`, and `separator` do, and which options are REMOVED in
  v4 so a v3 to v4 migration plan can flag them in advance.
  Prevents (a) accidentally REPLACING the default theme by writing `theme: { colors: {...} }`
  when `theme.extend.colors` was intended, (b) leaking content scanning into
  `node_modules` via `./**/*.{js,html}` and tanking build times, (c) configuring
  `darkMode: 'class'` and forgetting to toggle the class on `<html>` (no dark
  styles ever fire), (d) using `corePlugins: { float: false }`, `safelist: [...]`,
  or `separator: '_'` in a v3 project that is about to migrate to v4 (all three
  are silently removed), (e) building Next.js catch-all routes `[...slug]` into
  the content glob with literal brackets that the glob engine treats as a
  character class (none of the files inside match), and (f) mixing CommonJS and
  ESM exports in the same config (`module.exports` vs `export default`).
  Covers the complete v3 configuration surface: file location and module-system
  variants (`tailwind.config.js` CJS, `tailwind.config.mjs` ESM,
  `tailwind.config.ts` TypeScript-typed), content scanning with glob patterns,
  monorepo paths, `{ relative: true, files: [...] }` and the `transform` and
  `extract` advanced options, `theme` (REPLACE everything) vs `theme.extend`
  (ADDITIVE) semantics across every namespace, `presets: []` for shared base
  configs, the `plugin()` factory with all eight helpers (`addUtilities`,
  `matchUtilities`, `addComponents`, `matchComponents`, `addBase`, `addVariant`,
  `matchVariant`, `theme`, `config`, `e`), `darkMode` strategies (`media`, `class`,
  selector array, fully custom variant), `corePlugins: { float: false }` and
  `corePlugins: ['preflight']` to disable individual or whole-core utilities,
  `safelist: ['bg-red-500', { pattern: /bg-(red|blue)-(100|200)/, variants: ['hover'] }]`,
  `blocklist: ['container']`, `prefix: 'tw-'`, `important: true | '#app'`,
  `separator: '_'`, and `future: { hoverOnlyWhenSupported: true }`. Explicitly
  flags every option that is REMOVED OR CHANGED in v4 so migration planning is
  unambiguous.
  Keywords: tailwind config js, tailwind v3 configuration, content glob, theme extend, presets, plugins array, darkMode class, darkMode media, darkMode selector, corePlugins disable float, safelist pattern, blocklist, prefix tw-, important selector, separator, future flags, hoverOnlyWhenSupported, why is my tailwind class not generating, content array not picking up files, Next-js catch-all glob, monorepo tailwind, addUtilities, matchUtilities, addVariant, plugin helpers, removed in v4, what changes in v4, ESM CJS tailwind config, TypeScript tailwind config, tailwind 3-4.
license: MIT
compatibility: "Designed for Claude Code. Requires Tailwind CSS v3.4."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# Tailwind CSS Impl: v3 Configuration (`tailwind.config.js`)

In v3.4 the entire Tailwind project is configured through a single JavaScript file (`tailwind.config.js` / `.mjs` / `.ts`). Every behaviour, every token, every plugin lives here. Mastering this file is mastering v3. NOTE: v4 moves most of this into CSS `@theme { ... }` blocks ; this skill is v3-ONLY and explicitly flags every option that is REMOVED in v4.

## Quick Reference

### Option matrix (v3.4 full surface)

| Option : purpose : default : v4 status |
|--|
| `content` : files to scan for class names : `[]` : moved to `@source "./path"` directives |
| `theme` : REPLACE all default tokens : `{}` (extend defaults) : moved to `@theme` block |
| `theme.extend` : ADD to default tokens : `{}` : moved to `@theme` (always additive) |
| `presets` : shared base configs : `[]` : `@import` chains in CSS |
| `plugins` : JS plugins (addUtilities etc.) : `[]` : still supported via `@plugin "./..."` |
| `darkMode` : dark-mode strategy : `'media'` : `@custom-variant dark (...)` in CSS |
| `corePlugins` : disable built-ins : enabled : **REMOVED in v4 (no replacement)** |
| `safelist` : force-generate classes : `[]` : **REMOVED in v4** (use `@source inline("...")`) |
| `blocklist` : skip false-positive classes : `[]` : moved to `@source not "path"` |
| `prefix` : class-name prefix : `''` : **CHANGED in v4** (variant-style `tw:flex` instead of `tw-flex`) |
| `important` : !important all utilities : `false` : moved to `!` trailing on utility (`flex!`) |
| `separator` : variant-utility separator : `':'` : **REMOVED in v4 (no replacement)** |
| `future` : opt into next-major behaviours : `{}` : behaviours adopted as defaults in v4 |

### Minimal v3 config skeleton

```js
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './index.html',
    './src/**/*.{js,ts,jsx,tsx,vue,svelte}',
  ],
  theme: {
    extend: {
      colors: {
        brand: { 500: '#3b82f6', 600: '#2563eb' },
      },
    },
  },
  plugins: [
    require('@tailwindcss/typography'),
    require('@tailwindcss/forms'),
  ],
}
```

### Removed in v4 (FLAG ON MIGRATION)

| v3 option | v4 replacement |
|-----------|----------------|
| `corePlugins: { float: false }` | NONE. The utility cannot be disabled in v4. |
| `corePlugins: ['preflight']` (allowlist) | NONE for individual disabling. Use `@source not "..."` to scope which utilities are emitted. |
| `safelist: ['bg-red-500']` | `@source inline("bg-red-500")` in main CSS. |
| `safelist: [{ pattern: /bg-.../ }]` | `@source inline("bg-{red,blue}-{100..900..100}")` brace expansion. |
| `separator: '_'` | NONE. The separator is fixed at `:` in v4. |
| `prefix: 'tw-'` | `prefix: 'tw'` in v4 produces `tw:flex` (variant-style), NOT `tw-flex`. |
| `important: true` | `@important` directive OR trailing `!` per utility. |

## Decision Trees

### Picking the config-file format

```
TypeScript-first project?
├── ALWAYS use `tailwind.config.ts` with `import type { Config } from 'tailwindcss'`
├── exports the typed config and `module.exports = config` (or `export default config` if ESM)

ESM-only project (`"type": "module"` in package.json)?
├── ALWAYS use `tailwind.config.mjs` (or `.ts`) with `export default { ... }`
├── NEVER mix CommonJS (`module.exports`) with ESM (`export default`) in the same file

Plain JS project?
├── ALWAYS use `tailwind.config.js` with `module.exports = { ... }` (CJS)
└── Add the JSDoc type comment `/** @type {import('tailwindcss').Config} */` for editor IntelliSense
```

### `theme` vs `theme.extend`

```
adding NEW values to defaults?
└── ALWAYS use `theme.extend.{namespace}.{key} = value` (additive)

REPLACING the entire default namespace (e.g. wiping the default colors)?
└── ALWAYS use `theme.{namespace} = { onlyMyValues }` (NO `.extend`)
    └── DANGER : you lose every default color. `bg-red-500`, `text-slate-900`,
        every default theme reference STOPS WORKING. Confirm before committing.

partial replacement (keep some defaults, override others)?
└── Use `theme.extend.{namespace}` with the same key as a default to override it
    └── e.g. `theme.extend.colors.red = { 500: '#custom-red' }` overrides ONLY red-500
```

### `corePlugins` mental model (v3-only ; gone in v4)

```
want to disable specific utilities?
├── few specific utilities : `corePlugins: { float: false, clear: false }`
├── most utilities : ALLOWLIST mode `corePlugins: ['preflight', 'container']`
│   └── lists only the utilities to KEEP ; everything else is disabled
└── note : v4 has NO equivalent. If migration to v4 is on the roadmap, document
    the disabled utilities as a project rule and remove the `corePlugins` block
    before upgrading.
```

### `safelist` mental model (v3-only ; gone in v4)

```
class name lands in DOM but no CSS is generated?
├── string-concatenation source (`bg-${color}`) ?
│   └── ROOT CAUSE : content scan needs literal token. Fix at source (preferred)
│       OR add to safelist (escape hatch).
├── CMS / server-side generated class names not in any source file ?
│   └── safelist : ['text-3xl', 'bg-red-500', ...]
└── pattern-based force-include ?
    └── safelist : [{ pattern: /bg-(red|blue|green)-(100|200|300|400|500)/, variants: ['hover', 'focus'] }]
```

### `darkMode` strategy

```
which dark-mode trigger?
├── follow OS preference (no opt-in) : `darkMode: 'media'` (default ; matches `prefers-color-scheme: dark`)
├── manual `.dark` class on `<html>` : `darkMode: 'class'`
├── custom attribute `data-theme="dark"` : `darkMode: ['class', '[data-theme="dark"]']` (v3.4.1+)
├── manual selector beyond `.dark` (e.g. `.dark` OR `[data-theme=dark]`) : ['selector', '[data-theme=dark]']
└── fully custom expression : `darkMode: ['variant', '&:not(.light *)']`
```

## Patterns

### Pattern 1: Content scanning (the most consequential block)

```js
module.exports = {
  content: [
    './index.html',
    './src/**/*.{js,ts,jsx,tsx,vue,svelte,mdx}',
    './node_modules/@my-org/ui-lib/dist/**/*.{js,mjs}', // include a published shared library
  ],
  // ...
}
```

Critical rules :

- ALWAYS list exact file extensions. `*.{js}` skips `.jsx` ; `*.{js,ts,jsx,tsx}` covers all.
- NEVER use `./**/*.{js,html}` from the project root : it scans `node_modules` and tanks build time.
- For Next.js / Remix projects with catch-all routes `[...slug]`, escape the brackets to prevent the glob engine from interpreting them as a character class :

```js
content: [
  './pages/[[]...slug[]]/*.{js,jsx,tsx}', // literal brackets via [[ ]]
]
```

For monorepos with workspaces, point at each workspace explicitly :

```js
content: [
  './apps/web/**/*.{js,ts,jsx,tsx}',
  './packages/ui/src/**/*.{js,ts,jsx,tsx}',
  './packages/marketing/src/**/*.{mdx,jsx,tsx}',
]
```

### Pattern 2: Advanced content (relative + transform)

```js
content: {
  relative: true,                              // paths resolve from THIS config file
  files: [
    './src/**/*.{js,jsx}',
    { raw: '<div class="hidden md:block">Markup pulled from runtime DB</div>' },
  ],
  transform: {
    pug: (content) => require('pug').render(content),
  },
  extract: {
    pug: (content) => content.match(/[a-z][a-z0-9-_:/]*/g) || [],
  },
}
```

`relative: true` ALWAYS makes paths resolve relative to the config file, not the CWD. Use this in monorepos where the build runs from the root but the config lives in a workspace.

### Pattern 3: `theme.extend` for additive customisations

```js
theme: {
  extend: {
    colors: {
      brand: {
        50:  '#eff6ff',
        500: '#3b82f6',
        600: '#2563eb',
        900: '#1e3a8a',
      },
    },
    spacing: {
      '128': '32rem',
      '144': '36rem',
    },
    fontSize: {
      'display': ['4.5rem', { lineHeight: '1.05', letterSpacing: '-0.02em' }],
    },
    screens: {
      '3xl': '120rem',
    },
    fontFamily: {
      sans: ['Inter', 'ui-sans-serif', 'system-ui'],
    },
    keyframes: {
      shimmer: {
        '0%': { backgroundPosition: '-100% 0' },
        '100%': { backgroundPosition: '100% 0' },
      },
    },
    animation: {
      shimmer: 'shimmer 2s linear infinite',
    },
  },
}
```

ALL the default tokens stay available ; the listed values are added or override the matching key.

### Pattern 4: Plugin authorship (all helpers)

```js
const plugin = require('tailwindcss/plugin')

module.exports = {
  plugins: [
    plugin(function ({
      addUtilities,    // static utility classes
      matchUtilities,  // parameterised utility (accepts arbitrary value)
      addComponents,   // static component classes
      matchComponents, // parameterised component classes
      addBase,         // raw base styles
      addVariant,      // static custom variant
      matchVariant,    // parameterised custom variant
      theme,           // access theme values from the config
      config,          // access whole user config
      e,               // escape a string for use as a class-name part
    }) {
      // 1. Static utility
      addUtilities({
        '.content-auto': { 'content-visibility': 'auto' },
        '.content-hidden': { 'content-visibility': 'hidden' },
      })

      // 2. Parameterised utility : tab-1, tab-2, tab-[13]
      matchUtilities(
        { tab: (value) => ({ tabSize: value }) },
        { values: theme('tabSize') },
      )

      // 3. Component (lower specificity than utility ; overridable)
      addComponents({
        '.btn-primary': {
          padding: '0.5rem 1rem',
          backgroundColor: theme('colors.blue.600'),
          color: theme('colors.white'),
        },
      })

      // 4. Base styles
      addBase({
        'h1': { fontSize: theme('fontSize.4xl'), fontWeight: theme('fontWeight.bold') },
        'h2': { fontSize: theme('fontSize.3xl'), fontWeight: theme('fontWeight.semibold') },
      })

      // 5. Custom variant
      addVariant('third', '&:nth-child(3)')
      addVariant('hocus', ['&:hover', '&:focus'])
      matchVariant('nth', (v) => `&:nth-child(${v})`)
    }),
  ],
}
```

### Pattern 5: Presets for shared base configs

```js
// packages/design-system/tailwind-preset.js
module.exports = {
  theme: {
    extend: {
      colors: { brand: { /* ... */ } },
      fontFamily: { sans: ['Inter', 'sans-serif'] },
    },
  },
  plugins: [require('@tailwindcss/typography')],
}

// apps/web/tailwind.config.js
module.exports = {
  presets: [require('@my-org/design-system/tailwind-preset')],
  content: ['./src/**/*.{js,tsx}'],
  // Local overrides ADD ON TOP of the preset
  theme: { extend: { fontSize: { display: ['5rem', { lineHeight: '1' }] } } },
}
```

Presets merge ADDITIVELY by default ; the local config extends what the preset provides.

### Pattern 6: Removed-in-v4 options (use sparingly if migration is planned)

```js
module.exports = {
  // REMOVED in v4 : disable specific utilities
  corePlugins: {
    float: false,
    container: false,
  },

  // REMOVED in v4 : force-include classes the scanner cannot detect
  safelist: [
    'bg-red-500',
    'text-3xl',
    {
      pattern: /bg-(red|green|blue)-(100|200|300|400|500|600|700|800|900)/,
      variants: ['hover', 'focus'],
    },
  ],

  // REMOVED in v4 : change the variant-utility separator
  separator: '_', // class="hover_bg-red-500" instead of "hover:bg-red-500"
}
```

ALWAYS document each of these decisions in CHANGELOG or DECISIONS records, because they need an explicit migration plan when upgrading to v4.

## Reference Links

- `references/methods.md` : complete option-by-option reference with every accepted value type, JSON schema, v4 status.
- `references/examples.md` : full working configs (small app, monorepo, Next.js with catch-all, TypeScript, ESM, preset-driven design system).
- `references/anti-patterns.md` : the theme-replace trap, broad content glob, Next.js catch-all glob, mixed CJS/ESM exports, `corePlugins` in projects planning v4 migration, mutating `darkMode: 'class'` without HTML toggle.

## Verified Sources

- https://v3.tailwindcss.com/docs/configuration (option list, defaults, file location)
- https://v3.tailwindcss.com/docs/content-configuration (content scanning, glob rules, relative paths, transform/extract)
- https://v3.tailwindcss.com/docs/theme (theme vs theme.extend semantics, every namespace)
- https://v3.tailwindcss.com/docs/plugins (plugin() factory, all helpers)
- https://v3.tailwindcss.com/docs/presets (preset merging behaviour)
- https://v3.tailwindcss.com/docs/dark-mode (darkMode strategies including v3.4.1 selector)
- https://tailwindcss.com/docs/upgrade-guide (v4 removal list for corePlugins, safelist, separator)

Last verified : 2026-05-19 (Tailwind CSS v3.4.x latest).
