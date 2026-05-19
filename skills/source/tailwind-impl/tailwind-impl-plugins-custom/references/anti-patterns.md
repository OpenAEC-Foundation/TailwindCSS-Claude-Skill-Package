# Anti-Patterns : Custom Plugin Authorship

Each trap : symptom, root cause, fix, verification.

## Trap 1 : Registering Utilities Through addBase

### Symptom
The class registers and produces CSS, but variants like `hover:my-class`
do nothing. Inspecting the bundle shows the rule exists at the
top level (no `:hover` variant generated).

### Root cause
`addBase` emits to the `base` layer, which is intended for element-
selector resets. Tailwind does NOT run the variant pipeline against
`base` rules because variants are class-token-based.

### Fix
Use `addUtilities` for single-purpose classes, `addComponents` for
multi-utility recipes :

```js
// Wrong
addBase({ '.skew-10deg': { transform: 'skewY(-10deg)' } })

// Right
addUtilities({ '.skew-10deg': { transform: 'skewY(-10deg)' } })
```

### Verification
Grep the compiled CSS for `:hover.my-class` or `.hover\\:my-class`.
With `addUtilities`, the hover variant appears. With `addBase`, only
the base rule exists.

## Trap 2 : Missing values Map on matchUtilities

### Symptom
`matchUtilities` registers without error. Build succeeds. ZERO classes
produced in output CSS. No warning, no log line.

### Root cause
`matchUtilities` ONLY generates classes for keys present in
`options.values`. Without a `values` map it has nothing to enumerate.
Arbitrary values (`my-util-[42]`) still work, but bare classes do not.

```js
// Wrong, silent zero-output
matchUtilities({ tab: (v) => ({ tabSize: v }) })

// Wrong, undefined map silently treated as empty
matchUtilities(
  { tab: (v) => ({ tabSize: v }) },
  { values: theme('tabSize') }   // tabSize not in theme
)
```

### Fix
Provide a non-empty `values` map. ALWAYS ship defaults via the
`plugin(fn, config)` second argument so users do not need to
configure the plugin to get classes :

```js
module.exports = plugin(
  function ({ matchUtilities, theme }) {
    matchUtilities(
      { tab: (v) => ({ tabSize: v }) },
      { values: theme('tabSize') }
    )
  },
  { theme: { tabSize: { 1: '1', 2: '2', 4: '4' } } }
)
```

### Verification
Grep for `.tab-1`, `.tab-2`, `.tab-4` in the compiled CSS. All three
selectors should appear.

## Trap 3 : Forgetting Theme Defaults in plugin() Second Arg

### Symptom
Plugin works for the author's project (their `theme.extend` matches),
breaks on every consumer install. Consumer reports : "plugin installed,
no classes appear."

### Root cause
The plugin reads `theme('myNamespace')` but ships no defaults for it.
The consumer's config does not extend `myNamespace`, so the lookup
returns `undefined`, and `matchUtilities({}, { values: undefined })`
emits nothing.

### Fix
ALWAYS ship the default theme values your plugin reads :

```js
module.exports = plugin(
  function ({ matchUtilities, theme }) {
    matchUtilities(
      { spin: (v) => ({ animation: `spin ${v} linear infinite` }) },
      { values: theme('spinDuration') }
    )
  },
  {
    theme: {
      spinDuration: {
        slow: '4s',
        DEFAULT: '2s',
        fast: '0.5s',
      },
    },
  }
)
```

### Verification
Install the plugin in a clean project with zero `theme.extend`. Classes
should still appear in the build.

## Trap 4 : importing tailwindcss/plugin in a v4-Only Project

### Symptom
Build fails : `Cannot find module 'tailwindcss/plugin'` or similar.
Project uses Tailwind v4 exclusively, has no v3 dependency.

### Root cause
v4 ships through `@tailwindcss/postcss` (PostCSS plugin) or
`@tailwindcss/vite` (Vite plugin). The `tailwindcss` npm package in v4
still exposes the legacy `plugin` re-export for back-compat, but some
build setups (especially monorepos with deduped packages) trip over
the resolution.

### Fix
Two options :

1. Keep `require('tailwindcss/plugin')` and add `tailwindcss` as a
   direct dependency. The shim works.
2. Migrate to CSS-native authoring with `@utility` and `@custom-variant`
   if the plugin only does what those directives cover :
   ```css
   @utility skew-10deg {
     transform: skewY(-10deg);
   }
   ```

### Verification
Build succeeds. The compiled output contains the plugin's selectors.

## Trap 5 : @plugin Path Resolved From Wrong Base

### Symptom
v4 build fails : `Cannot find module './plugins/my-plugin.js'`. The
file exists.

### Root cause
`@plugin "./path"` resolves the path relative to the CSS file the
directive lives in, NOT the project root or the importer. A common
mistake : the CSS file is `src/styles/app.css` and the plugin lives
in `src/plugins/my-plugin.js`. Writing `@plugin "./plugins/my-plugin.js"`
fails because the resolver looks for `src/styles/plugins/my-plugin.js`.

### Fix
Use a path relative to the CSS file :

```css
/* src/styles/app.css */
@plugin "../plugins/my-plugin.js";
```

Or use an absolute or package-qualified path :

```css
@plugin "@my-org/tailwind-skew";
```

### Verification
The build error names the exact resolved path. Compare it to the
actual on-disk location.

## Trap 6 : Stale Variants Array in v3 addUtilities Options

### Symptom
Tailwind v3.x build prints a deprecation notice : `the variants option
has been removed in v3` (or silently ignored under newer minor versions).

### Root cause
Legacy v2 code passed variants as a second argument :

```js
addUtilities(
  { '.scrollbar-hidden': { /* ... */ } },
  ['responsive', 'hover']
)
```

In v3 the variant system runs by default ; the array is redundant and
deprecated.

### Fix
Drop the second argument :

```js
addUtilities({
  '.scrollbar-hidden': {
    '&::-webkit-scrollbar': { display: 'none' },
  },
})
```

### Verification
No deprecation warning in `npm run build` output. Variants
(`hover:scrollbar-hidden`) still apply.

## Trap 7 : Confusing @utility Name-* Pattern With Static Name

### Symptom
v4 plugin author writes :

```css
@utility tab-* {
  tab-size: 4;
}
```

expecting `tab-4`, `tab-2`, etc. Result : only `tab-*` literal selector
(which never matches a real class), no `tab-N` classes appear.

### Root cause
`@utility name-* { ... }` is the FUNCTIONAL form. It requires
`--value(...)` to read the suffix as a value :

```css
@utility tab-* {
  tab-size: --value(--tab-size-*);  /* or --value(integer) */
}
```

Without `--value(...)`, the wildcard never expands.

### Fix
Either declare static utilities (no `*`) :

```css
@utility tab-2 { tab-size: 2; }
@utility tab-4 { tab-size: 4; }
```

Or use the functional form with `--value()` :

```css
@theme {
  --tab-size-2: 2;
  --tab-size-4: 4;
}
@utility tab-* {
  tab-size: --value(--tab-size-*);
}
```

### Verification
`tab-2` and `tab-4` both appear in compiled CSS with correct
`tab-size` values.

## Trap 8 : Depending on tailwindcss Instead of peerDependencies

### Symptom
Consumer reports duplicate Tailwind installs. `npm ls tailwindcss`
shows two versions in node_modules. Build picks the plugin's bundled
version instead of the consumer's, leading to mismatched APIs and
silent feature gaps.

### Root cause
The plugin's `package.json` lists `"tailwindcss"` under
`"dependencies"` instead of `"peerDependencies"`. npm dedupes only
when versions are compatible ; an exact pin in dependencies forces
a duplicate install.

### Fix
ALWAYS declare `tailwindcss` as a peer dependency :

```json
{
  "peerDependencies": {
    "tailwindcss": ">=3.4 <5"
  }
}
```

Optionally add `peerDependenciesMeta` if your plugin works without
Tailwind installed at install time (unusual) :

```json
{
  "peerDependenciesMeta": {
    "tailwindcss": { "optional": false }
  }
}
```

### Verification
`npm ls tailwindcss` in the consumer project shows exactly one copy.
The plugin's classes appear in the consumer's CSS bundle.
