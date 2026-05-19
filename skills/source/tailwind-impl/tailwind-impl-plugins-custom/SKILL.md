---
name: tailwind-impl-plugins-custom
description: >
  Use when authoring a custom Tailwind CSS plugin to add utilities,
  components, base styles, or variants, regardless of whether the host
  project runs Tailwind v3 (JS plugin in tailwind-config-js) or v4 (JS
  plugin loaded via @plugin or the CSS-native @utility directive).
  Prevents the four most common plugin-authorship traps: registering
  utilities through addBase (where variants do not apply), forgetting
  the second-argument default theme so theme('tabSize') returns
  undefined, omitting the values map on matchUtilities causing zero
  classes to compile, and importing tailwindcss/plugin in a v4-only
  project where the package no longer exists at runtime. Covers the
  full plugin() signature (addUtilities, addComponents, addBase,
  addVariant, matchUtilities, matchComponents, matchVariant, theme,
  config, corePlugins, e), plugin-withOptions for parameterised
  plugins, shipping default theme values through the configFn, the
  v4-native @utility name-* with --value() and --modifier(), the
  v4 @custom-variant directive, the v4 @plugin loader for legacy JS
  plugins, theme path syntax (theme('colors.red.500'),
  theme('spacing.4')), variants arrays on addUtilities under v3, and
  packaging a plugin as a publishable npm module.
  Keywords: tailwind plugin, custom plugin, plugin function,
  addUtilities, addComponents, addBase, addVariant, matchUtilities,
  matchComponents, matchVariant, plugin.withOptions, plugin-withOptions,
  theme function, config function, corePlugins, e helper, tabSize
  matchUtilities, hocus variant, peer variant plugin, @utility v4,
  @utility name-*, --value(), --modifier(), --alpha(), --spacing(),
  @custom-variant, @plugin tailwind v4, load legacy plugin v4,
  theme('colors.red.500'), theme('spacing.4'), npm package tailwind
  plugin, publish tailwind plugin, my plugin classes not working,
  matchUtilities not generating, how do I add a custom utility,
  how do I write a tailwind plugin, where do plugin classes go,
  why doesn't my variant apply, tailwind-config-js plugins array.
license: MIT
compatibility: "Designed for Claude Code. Requires Tailwind CSS v3.4 or v4.0+."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# Tailwind CSS Custom Plugins

Tailwind exposes three authoring paths for custom styles : the JS
`plugin()` API (works in v3 and v4 for back-compat), v4's CSS-native
`@utility` directive, and v4's `@custom-variant` directive. Pick by
constraint, not by taste.

Companion skills :

- `tailwind-impl-config-v4` : `@utility`, `@custom-variant`, `@plugin`
  directives in depth
- `tailwind-impl-config-v3` : `tailwind.config.js` surface
- `tailwind-core-v3-vs-v4` : version-wide API differences
- `tailwind-syntax-functional-utilities` : v4 `--value()` / `--modifier()`

## Decision : Which Authoring Path

| Constraint | Path |
|------------|------|
| Project on v3 | JS `plugin()` only |
| Project on v4 only, no v3 back-compat | CSS `@utility` + `@custom-variant` |
| Project on v4 with `@plugin` loading legacy plugins | JS `plugin()` via `@plugin "./my-plugin.js"` |
| Plugin must support BOTH v3 and v4 | JS `plugin()` (works in both) |
| Plugin needs runtime config from caller | `plugin.withOptions(fn, configFn)` |
| Plugin only adds tokens, no utilities | `@theme { ... }` directly, no plugin |

## Quick Reference : JS plugin() Signature

```js
const plugin = require('tailwindcss/plugin')

module.exports = plugin(function ({
  addUtilities,
  addComponents,
  addBase,
  addVariant,
  matchUtilities,
  matchComponents,
  matchVariant,
  theme,
  config,
  corePlugins,
  e,
}) {
  // body
})
```

| Helper | Purpose | Variants apply | Theme access |
|--------|---------|----------------|--------------|
| `addUtilities` | Static utility classes | yes | via `theme()` arg |
| `addComponents` | Multi-utility component classes | yes (under v4) | via `theme()` arg |
| `addBase` | Resets, `@font-face`, element selectors | NO | via `theme()` arg |
| `addVariant(name, selector)` | Static variant | n/a | n/a |
| `matchUtilities(map, opts)` | Functional utilities with arbitrary-value support | yes | `opts.values` |
| `matchComponents(map, opts)` | Functional components | yes | `opts.values` |
| `matchVariant(name, fn, opts)` | Parameterised variants (`nth-*`, `data-*`) | n/a | `opts.values` |
| `theme(path)` | Read theme value (`'colors.red.500'`) | n/a | direct |
| `config(key)` | Read full config value | n/a | n/a |
| `corePlugins(name)` | Test whether a core plugin is enabled | n/a | n/a |
| `e(str)` | Escape a string for safe CSS class names | n/a | n/a |

## Pattern 1 : Static Utility (addUtilities)

```js
const plugin = require('tailwindcss/plugin')

module.exports = plugin(function ({ addUtilities }) {
  addUtilities({
    '.skew-10deg': { transform: 'skewY(-10deg)' },
    '.skew-15deg': { transform: 'skewY(-15deg)' },
    '.content-auto': { contentVisibility: 'auto' },
  })
})
```

Generated classes participate in the full variant pipeline
(`hover:skew-10deg`, `md:skew-15deg`, `dark:content-auto`).

## Pattern 2 : Component Class (addComponents)

```js
module.exports = plugin(function ({ addComponents, theme }) {
  addComponents({
    '.btn': {
      padding: '0.5rem 1rem',
      borderRadius: theme('borderRadius.md'),
      backgroundColor: theme('colors.zinc.900'),
      color: theme('colors.zinc.50'),
      '&:hover': {
        backgroundColor: theme('colors.zinc.700'),
      },
    },
  })
})
```

ALWAYS use `addComponents` (not `addUtilities`) for multi-utility
recipes. Component classes are emitted into the `components` layer so
user-applied utilities (`btn rounded-none`) override them.

## Pattern 3 : Base Styles (addBase)

```js
module.exports = plugin(function ({ addBase, theme }) {
  addBase({
    h1: { fontSize: theme('fontSize.2xl'), fontWeight: theme('fontWeight.bold') },
    h2: { fontSize: theme('fontSize.xl') },
    'html': { fontFamily: theme('fontFamily.sans').join(', ') },
  })
})
```

NEVER expect variants on `addBase` output. Base rules target raw
selectors, not class tokens ; `hover:h1` is meaningless.

## Pattern 4 : Static Variant (addVariant)

```js
module.exports = plugin(function ({ addVariant }) {
  addVariant('hocus', ['&:hover', '&:focus'])
  addVariant('optional', '&:optional')
  addVariant('group-hocus', [':merge(.group):hover &', ':merge(.group):focus &'])
})
```

Usage : `class="hocus:underline"` underlines on hover OR focus.

## Pattern 5 : Functional Utility (matchUtilities)

```js
module.exports = plugin(function ({ matchUtilities, theme }) {
  matchUtilities(
    {
      tab: (value) => ({ tabSize: value }),
    },
    {
      values: theme('tabSize'),
      type: ['number', 'length'],
      supportsNegativeValues: false,
    }
  )
})
```

`values` is the map of `{ classSuffix: value }`. Reading from
`theme('tabSize')` automatically picks up user overrides in
`tailwind.config.js` `theme.extend.tabSize`. Without a `values` map,
`matchUtilities` generates ZERO classes (silent failure).

Arbitrary-value support is automatic : `tab-[7]` works without
listing `7` in `values`.

## Pattern 6 : Parameterised Variant (matchVariant)

```js
module.exports = plugin(function ({ matchVariant }) {
  matchVariant(
    'nth',
    (value) => `&:nth-child(${value})`,
    { values: { 1: '1', 2: '2', odd: 'odd', even: 'even' } }
  )
})
```

Usage : `nth-1:underline`, `nth-[3n+1]:bg-zinc-50`.

## Pattern 7 : Plugin With User Options

```js
const plugin = require('tailwindcss/plugin')

module.exports = plugin.withOptions(
  function (options = {}) {
    const className = options.className ?? 'markdown'
    return function ({ addComponents, theme }) {
      addComponents({
        [`.${className}`]: {
          maxWidth: theme('maxWidth.prose'),
          color: theme('colors.zinc.700'),
        },
      })
    }
  },
  function (options = {}) {
    return {
      theme: {
        extend: {
          maxWidth: {
            prose: options.proseWidth ?? '65ch',
          },
        },
      },
    }
  }
)
```

User invokes with `require('./my-plugin')({ className: 'docs',
proseWidth: '72ch' })`.

## Pattern 8 : Shipping Default Theme Values

`plugin()` accepts a second argument with `theme.extend` defaults :

```js
module.exports = plugin(
  function ({ matchUtilities, theme }) {
    matchUtilities(
      { tab: (v) => ({ tabSize: v }) },
      { values: theme('tabSize') }
    )
  },
  {
    theme: {
      tabSize: { 1: '1', 2: '2', 4: '4', 8: '8' },
    },
  }
)
```

User-side `theme.extend.tabSize` overrides merge over the plugin
defaults.

## v4-Native : @utility Directive (CSS Authoring)

When you only target v4 and prefer CSS over JS :

```css
@utility content-auto {
  content-visibility: auto;
}

@utility skew-10deg {
  transform: skewY(-10deg);
}
```

Functional form with `--value()` reads tokens from `@theme` :

```css
@theme {
  --tab-size-1: 1;
  --tab-size-2: 2;
  --tab-size-4: 4;
}

@utility tab-* {
  tab-size: --value(--tab-size-*);
}
```

`tab-*` automatically supports bare integers (`tab-3`), arbitrary
values (`tab-[7]`), and theme keys (`tab-2`). See
`tailwind-syntax-functional-utilities` for `--modifier()`,
`--alpha()`, `--spacing()`.

## v4-Native : @custom-variant Directive

```css
@custom-variant theme-midnight (&:where([data-theme="midnight"] *));

@custom-variant any-hover {
  @media (any-hover: hover) {
    &:hover {
      @slot;
    }
  }
}
```

The shorthand form (parenthesised) is single-selector. The block form
supports media queries and nested rules via `@slot`.

## v4 : Loading a JS Plugin via @plugin

A v4 project can still load legacy JS plugins :

```css
@import "tailwindcss";

@plugin "@tailwindcss/typography";
@plugin "./plugins/my-local-plugin.js";
```

The path is resolved relative to the CSS file. The plugin file uses
the standard `plugin()` signature ; the v4 build loads it through the
same JS bridge that maintains v3 compatibility.

## Theme Path Syntax

The `theme()` helper accepts dot-notation paths into the resolved
theme object :

```js
theme('colors.red.500')         // '#ef4444'
theme('spacing.4')              // '1rem'
theme('fontSize.lg')            // '1.125rem' or [size, { lineHeight }]
theme('screens.md')             // '768px'
theme('fontFamily.sans')        // array of family names
theme('borderRadius.full')      // '9999px'
```

Paths trace `theme` after merging user `theme.extend`. Missing paths
return `undefined`, NOT an error : guard with `?? fallback`.

## Packaging a Plugin as an npm Module

`package.json` :

```json
{
  "name": "@my-org/tailwind-skew",
  "version": "1.0.0",
  "main": "./dist/index.js",
  "exports": {
    ".": "./dist/index.js"
  },
  "peerDependencies": {
    "tailwindcss": ">=3.4 <5"
  },
  "files": ["dist"]
}
```

`src/index.js` :

```js
const plugin = require('tailwindcss/plugin')

module.exports = plugin(function ({ addUtilities }) {
  addUtilities({
    '.skew-10deg': { transform: 'skewY(-10deg)' },
  })
})
```

User-side install :

```bash
npm install @my-org/tailwind-skew
```

v3 user :

```js
// tailwind.config.js
module.exports = {
  plugins: [require('@my-org/tailwind-skew')],
}
```

v4 user :

```css
@import "tailwindcss";
@plugin "@my-org/tailwind-skew";
```

Use `peerDependencies` for `tailwindcss`, NOT `dependencies`. A direct
dependency would install a duplicate Tailwind in user node_modules and
break version resolution.

## Verification Checklist

1. Plugin classes appear in compiled CSS. Grep `.next/static/css` or
   the Vite-emitted CSS file for one of your selectors.
2. Variants apply : `hover:my-utility` produces a `:hover` rule, not a
   raw rule. If not, you probably registered through `addBase`.
3. `matchUtilities` generates classes for every key in `values`. Count
   selectors in the output bundle.
4. Arbitrary values work : `my-util-[42]` produces a CSS rule.
5. Theme lookups resolve : log `theme('colors.brand')` from inside the
   plugin body during development.
6. v4 `@plugin` loading : the plugin file path resolves relative to
   the CSS file, NOT the project root.

## References

- `references/methods.md` : every helper signature, options, v3-only
  vs v3+v4 vs v4-only authoring matrix
- `references/examples.md` : ten full plugin source files with
  matching consumer-side usage
- `references/anti-patterns.md` : eight authorship traps with cause,
  symptom, fix

## Sources

- https://v3.tailwindcss.com/docs/plugins
- https://tailwindcss.com/docs/adding-custom-styles
- https://tailwindcss.com/docs/functions-and-directives
- https://github.com/tailwindlabs/tailwindcss/blob/main/packages/tailwindcss/src/plugin.ts
