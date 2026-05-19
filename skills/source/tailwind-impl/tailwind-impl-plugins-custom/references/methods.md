# Methods : Plugin API Reference

Per-helper deep dive. JS API works in both v3 and v4. CSS-native v4
alternatives noted where relevant.

## addUtilities(utilities, options?)

### Signature

```js
addUtilities(
  utilities: Record<string, CSSDeclaration>,
  options?: { variants?: string[], respectPrefix?: boolean, respectImportant?: boolean }
)
```

### v3 behaviour

In v3, the second argument controls which variants apply. Modern
Tailwind v3.x respects all configured variants by default, so
`options.variants` is rarely needed. Legacy v2 syntax :

```js
addUtilities(
  { '.skew-10deg': { transform: 'skewY(-10deg)' } },
  ['responsive', 'hover']
)
```

### v4 behaviour

`addUtilities` registers classes that participate in the full variant
pipeline automatically. No options needed for variant gating.

### CSS-native v4 equivalent

```css
@utility skew-10deg {
  transform: skewY(-10deg);
}
```

## addComponents(components)

### Signature

```js
addComponents(components: Record<string, CSSDeclaration>)
```

### Purpose

Register multi-utility recipes that user-side utilities can override
through cascade order. Components emit into the `components` layer ;
utilities emit into the `utilities` layer (higher specificity for
overrides, lower CSS source order in v4).

### Example

```js
addComponents({
  '.card': {
    backgroundColor: theme('colors.white'),
    borderRadius: theme('borderRadius.lg'),
    padding: theme('spacing.6'),
    boxShadow: theme('boxShadow.xl'),
  },
  '.card-compact': {
    padding: theme('spacing.3'),
  },
})
```

User can override : `<div class="card rounded-none">` keeps the card's
shadow and padding but drops the rounding.

### CSS-native v4 equivalent

```css
@layer components {
  .card {
    background-color: var(--color-white);
    border-radius: var(--radius-lg);
    padding: --spacing(6);
    box-shadow: var(--shadow-xl);
  }
}
```

## addBase(rules)

### Signature

```js
addBase(rules: Record<string, CSSDeclaration>)
```

### Purpose

Register element-selector defaults : resets, `@font-face`, typography
defaults. Output emits to the `base` layer, BELOW utilities and
components.

### Example

```js
addBase({
  '@font-face': {
    fontFamily: 'Inter',
    src: 'url(/fonts/inter.woff2)',
    fontDisplay: 'swap',
  },
  h1: { fontSize: theme('fontSize.2xl'), fontWeight: '700' },
  h2: { fontSize: theme('fontSize.xl'), fontWeight: '600' },
  'a:where(:not([class]))': {
    color: theme('colors.blue.600'),
    textDecoration: 'underline',
  },
})
```

NEVER expect variants. `hover:h1` cannot exist ; variants are
class-token-based and `h1` is an element selector.

### CSS-native v4 equivalent

```css
@layer base {
  h1 { font-size: var(--text-2xl); font-weight: 700; }
  h2 { font-size: var(--text-xl); font-weight: 600; }
}
```

## addVariant(name, selector | selector[])

### Signature

```js
addVariant(name: string, selector: string | string[])
```

### Single selector

```js
addVariant('optional', '&:optional')
```

The `&` placeholder is replaced by the target selector.

### Multiple selectors (compounds into multiple rules)

```js
addVariant('hocus', ['&:hover', '&:focus'])
```

Generates two rules : one for hover, one for focus. Both share the same
declarations.

### Group selectors

```js
addVariant('group-hocus', [
  ':merge(.group):hover &',
  ':merge(.group):focus &',
])
```

`:merge(.group)` deduplicates the group selector when multiple
group-* variants stack.

### CSS-native v4 equivalent

```css
@custom-variant hocus {
  &:hover { @slot; }
  &:focus { @slot; }
}
```

## matchUtilities(map, options)

### Signature

```js
matchUtilities(
  map: Record<string, (value: string, extra: { modifier: string | null }) => CSSDeclaration>,
  options: {
    values: Record<string, string>,
    type?: string | string[],
    supportsNegativeValues?: boolean,
    modifiers?: Record<string, string> | 'any',
  }
)
```

### Basic

```js
matchUtilities(
  { tab: (value) => ({ tabSize: value }) },
  { values: theme('tabSize') }
)
```

Generates `.tab-1`, `.tab-2`, etc., from `theme('tabSize')` keys, AND
supports `.tab-[arbitrary]` automatically.

### Negative values

```js
matchUtilities(
  { 'rotate-y': (v) => ({ transform: `rotateY(${v})` }) },
  {
    values: theme('rotate'),
    supportsNegativeValues: true,
  }
)
```

Generates `.-rotate-y-45` (negative version) alongside `.rotate-y-45`.

### Type validation

```js
matchUtilities(
  { aspect: (v) => ({ aspectRatio: v }) },
  { values: theme('aspectRatio'), type: ['ratio', 'number'] }
)
```

Arbitrary values are validated against the type list ; invalid types
do not emit a class.

### Modifiers (the `/` separator)

```js
matchUtilities(
  {
    text: (value, { modifier }) => ({
      fontSize: value,
      lineHeight: modifier ?? 'normal',
    }),
  },
  {
    values: theme('fontSize'),
    modifiers: theme('lineHeight'),
  }
)
```

Usage : `text-lg/6` resolves to `{ fontSize: '1.125rem', lineHeight: 1.5 }`.

### CSS-native v4 equivalent

```css
@utility tab-* {
  tab-size: --value(--tab-size-*);
}

@utility text-* {
  font-size: --value(--text-*, [length]);
  line-height: --modifier(--leading-*, [length], [*]);
}
```

## matchComponents(map, options)

Identical signature to `matchUtilities`, but emits to the `components`
layer. Used for functional component classes :

```js
matchComponents(
  {
    'tag': (value) => ({
      display: 'inline-block',
      backgroundColor: value,
      padding: '0.125rem 0.5rem',
      borderRadius: '0.25rem',
      color: 'white',
    }),
  },
  { values: theme('colors') }
)
```

## matchVariant(name, fn, options)

### Signature

```js
matchVariant(
  name: string,
  fn: (value: string, extra: { modifier: string | null }) => string | string[],
  options?: { values?: Record<string, string>, sort?: (a, b) => number }
)
```

### Example

```js
matchVariant(
  'nth',
  (value) => `&:nth-child(${value})`,
  {
    values: {
      1: '1',
      2: '2',
      odd: 'odd',
      even: 'even',
    },
  }
)
```

Usage : `nth-1:underline`, `nth-odd:bg-zinc-50`, `nth-[3n+1]:font-bold`.

### Custom sort

```js
matchVariant(
  'min',
  (value) => `@media (min-width: ${value})`,
  {
    sort(a, z) {
      return parseInt(a.value) - parseInt(z.value)
    },
  }
)
```

Sort controls the order in the output bundle for stacking determinism.

## theme(path, defaultValue?)

### Signature

```js
theme(path: string, defaultValue?: unknown): unknown
```

### Dot-notation paths

```js
theme('colors.red.500')         // '#ef4444'
theme('spacing.4')              // '1rem'
theme('fontSize.lg')            // ['1.125rem', { lineHeight: '1.75rem' }]
theme('screens.md')             // '768px'
theme('fontFamily.sans')        // ['ui-sans-serif', 'system-ui', ...]
```

### With default

```js
theme('colors.brand.primary', '#000000')
```

Returns the default if the path is missing. Without the default,
missing paths return `undefined`.

### Inside a CSS value string

```js
addUtilities({
  '.card': {
    backgroundColor: theme('colors.zinc.100'),
    boxShadow: `0 4px 6px ${theme('colors.zinc.900')}33`,
  },
})
```

## config(key, defaultValue?)

Reads the full Tailwind config object, not just `theme` :

```js
config('darkMode')           // 'media', 'class', or 'selector'
config('important')          // boolean or string
config('separator')          // ':' (default)
```

## corePlugins(name)

```js
if (corePlugins('preflight')) {
  // user has preflight enabled
}
```

Returns boolean. Useful for plugins that want to behave differently
when core plugins are disabled. NOTE : `corePlugins` config is REMOVED
in v4 ; this helper exists for v3 only.

## e(str)

Escapes a string for safe use as a CSS class fragment :

```js
e('foo.bar')   // 'foo\\.bar'
e('1/2')       // '1\\/2'
```

Use when building class names that interpolate user-supplied values.

## plugin(fn, configObject?)

### Signature

```js
plugin(
  handler: (api: PluginAPI) => void,
  config?: { theme?: { extend?: object }, ... }
)
```

The second argument is a partial Tailwind config that merges into the
user's config. Use it to ship default theme values your plugin
depends on :

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

## plugin.withOptions(fn, configFn?)

### Signature

```js
plugin.withOptions<O>(
  handler: (options: O) => (api: PluginAPI) => void,
  configFn?: (options: O) => Partial<Config>
)
```

`handler` is a curried function : receives options, returns the
real plugin body. `configFn` similarly receives options and returns
the partial config.

```js
module.exports = plugin.withOptions(
  function (options = {}) {
    return function ({ addComponents }) {
      addComponents({
        [`.${options.className ?? 'box'}`]: {
          padding: options.padding ?? '1rem',
        },
      })
    }
  },
  function (options = {}) {
    return {
      theme: {
        extend: {
          spacing: { custom: options.customSpacing ?? '2rem' },
        },
      },
    }
  }
)
```

## v3-Only vs v4-Only vs Cross-Compatible

| Feature | v3 | v4 |
|---------|----|----|
| `plugin()` JS API | yes | yes (loaded via `@plugin`) |
| `plugin.withOptions()` | yes | yes |
| `addUtilities`, `addComponents`, `addBase` | yes | yes |
| `addVariant(name, selector)` | yes | yes |
| `matchUtilities`, `matchComponents`, `matchVariant` | yes | yes |
| `theme()`, `config()`, `e()` | yes | yes |
| `corePlugins()` helper | yes | helper exists but `corePlugins` config is removed |
| `@utility name { ... }` directive | NO | yes |
| `@utility name-* { property: --value(--ns-*) }` | NO | yes |
| `@custom-variant` directive | NO | yes |
| `@plugin "..."` in CSS | NO | yes |
| Shipping default theme via `plugin(fn, config)` second arg | yes | yes |

## Loader Resolution in v4 @plugin

`@plugin` accepts :

- Bare specifier : `@plugin "@tailwindcss/typography";` (resolved via
  Node module resolution from the CSS file's location)
- Relative path : `@plugin "./plugins/skew.js";` (resolved relative to
  the CSS file, NOT the project root)
- Absolute path : `@plugin "/abs/path/plugin.js";`

A typo in the path produces a clear build error, not a silent miss.
