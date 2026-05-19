# Examples : Full Plugin Source

Ten complete plugin files with consumer-side usage. Each example is
runnable. JS examples work in both v3 and v4 unless noted.

## Example 1 : Skew Utilities

`plugins/skew.js` :

```js
const plugin = require('tailwindcss/plugin')

module.exports = plugin(function ({ addUtilities }) {
  addUtilities({
    '.skew-5deg': { transform: 'skewY(-5deg)' },
    '.skew-10deg': { transform: 'skewY(-10deg)' },
    '.skew-15deg': { transform: 'skewY(-15deg)' },
    '.skew-20deg': { transform: 'skewY(-20deg)' },
  })
})
```

v3 consumer (`tailwind.config.js`) :

```js
module.exports = {
  content: ['./src/**/*.{html,js,ts,jsx,tsx}'],
  plugins: [require('./plugins/skew')],
}
```

v4 consumer (`app.css`) :

```css
@import "tailwindcss";
@plugin "./plugins/skew.js";
```

Usage :

```html
<div class="skew-10deg hover:skew-15deg">Tilted</div>
```

## Example 2 : Button Components With Theme Tokens

`plugins/buttons.js` :

```js
const plugin = require('tailwindcss/plugin')

module.exports = plugin(function ({ addComponents, theme }) {
  addComponents({
    '.btn': {
      display: 'inline-flex',
      alignItems: 'center',
      gap: theme('spacing.2'),
      padding: `${theme('spacing.2')} ${theme('spacing.4')}`,
      borderRadius: theme('borderRadius.md'),
      fontWeight: theme('fontWeight.medium'),
      transition: 'background-color 150ms',
    },
    '.btn-primary': {
      backgroundColor: theme('colors.indigo.600'),
      color: theme('colors.white'),
      '&:hover': { backgroundColor: theme('colors.indigo.700') },
    },
    '.btn-secondary': {
      backgroundColor: theme('colors.zinc.200'),
      color: theme('colors.zinc.900'),
      '&:hover': { backgroundColor: theme('colors.zinc.300') },
    },
  })
})
```

Usage :

```html
<button class="btn btn-primary">Save</button>
<button class="btn btn-secondary">Cancel</button>
```

## Example 3 : Typography Base Resets

`plugins/typography-base.js` :

```js
const plugin = require('tailwindcss/plugin')

module.exports = plugin(function ({ addBase, theme }) {
  addBase({
    h1: {
      fontSize: theme('fontSize.4xl'),
      fontWeight: theme('fontWeight.bold'),
      lineHeight: theme('lineHeight.tight'),
    },
    h2: {
      fontSize: theme('fontSize.3xl'),
      fontWeight: theme('fontWeight.semibold'),
      lineHeight: theme('lineHeight.snug'),
    },
    h3: {
      fontSize: theme('fontSize.2xl'),
      fontWeight: theme('fontWeight.semibold'),
    },
    'a:where(:not([class]))': {
      color: theme('colors.indigo.600'),
      textDecoration: 'underline',
    },
    code: {
      fontFamily: theme('fontFamily.mono').join(', '),
      fontSize: '0.875em',
    },
  })
})
```

## Example 4 : hocus Variant

`plugins/variants.js` :

```js
const plugin = require('tailwindcss/plugin')

module.exports = plugin(function ({ addVariant }) {
  addVariant('hocus', ['&:hover', '&:focus'])
  addVariant('group-hocus', [
    ':merge(.group):hover &',
    ':merge(.group):focus &',
  ])
  addVariant('peer-hocus', [
    ':merge(.peer):hover ~ &',
    ':merge(.peer):focus ~ &',
  ])
})
```

Usage :

```html
<button class="text-zinc-600 hocus:text-zinc-900">Pointer or keyboard</button>
<a class="group">
  <span class="opacity-0 group-hocus:opacity-100">visible on hover OR focus</span>
</a>
```

## Example 5 : Tab Size Functional Utility (matchUtilities)

`plugins/tab-size.js` :

```js
const plugin = require('tailwindcss/plugin')

module.exports = plugin(
  function ({ matchUtilities, theme }) {
    matchUtilities(
      {
        tab: (value) => ({ tabSize: value }),
      },
      {
        values: theme('tabSize'),
        type: ['number', 'length'],
      }
    )
  },
  {
    theme: {
      tabSize: {
        1: '1',
        2: '2',
        4: '4',
        8: '8',
      },
    },
  }
)
```

Usage :

```html
<pre class="tab-4">function f() { ... }</pre>
<pre class="tab-[7]">arbitrary tab size</pre>
```

## Example 6 : Aspect-Ratio With Negative Support and Modifiers

`plugins/aspect.js` :

```js
const plugin = require('tailwindcss/plugin')

module.exports = plugin(function ({ matchUtilities, theme }) {
  matchUtilities(
    {
      ratio: (value, { modifier }) => ({
        aspectRatio: modifier ? `${value} / ${modifier}` : value,
      }),
    },
    {
      values: { square: '1', portrait: '3 / 4', landscape: '4 / 3' },
      modifiers: 'any',
    }
  )
})
```

Usage :

```html
<div class="ratio-square">1:1</div>
<div class="ratio-[16]/9">16:9 via modifier</div>
```

## Example 7 : Parameterised nth Variant (matchVariant)

`plugins/nth.js` :

```js
const plugin = require('tailwindcss/plugin')

module.exports = plugin(function ({ matchVariant }) {
  matchVariant(
    'nth',
    (value) => `&:nth-child(${value})`,
    {
      values: {
        1: '1',
        2: '2',
        3: '3',
        first: '1',
        last: 'last',
        odd: 'odd',
        even: 'even',
      },
    }
  )
})
```

Usage :

```html
<ul>
  <li class="nth-odd:bg-zinc-100">Striped row 1</li>
  <li class="nth-odd:bg-zinc-100">Striped row 2</li>
  <li class="nth-odd:bg-zinc-100">Striped row 3</li>
</ul>
```

## Example 8 : Plugin With User Options (withOptions)

`plugins/markdown.js` :

```js
const plugin = require('tailwindcss/plugin')

module.exports = plugin.withOptions(
  function (options = {}) {
    const className = options.className ?? 'markdown'
    const accent = options.accentColor ?? 'indigo'
    return function ({ addComponents, theme }) {
      addComponents({
        [`.${className}`]: {
          maxWidth: theme('maxWidth.prose'),
          color: theme('colors.zinc.700'),
          [`& a`]: {
            color: theme(`colors.${accent}.600`),
            textDecoration: 'underline',
          },
          [`& h1, & h2, & h3`]: {
            fontWeight: theme('fontWeight.bold'),
            color: theme('colors.zinc.900'),
          },
          [`& pre`]: {
            backgroundColor: theme('colors.zinc.900'),
            color: theme('colors.zinc.50'),
            padding: theme('spacing.4'),
            borderRadius: theme('borderRadius.md'),
          },
        },
      })
    }
  },
  function () {
    return {
      theme: {
        extend: {
          maxWidth: { prose: '65ch' },
        },
      },
    }
  }
)
```

v3 consumer :

```js
module.exports = {
  plugins: [
    require('./plugins/markdown')({ className: 'docs', accentColor: 'emerald' }),
  ],
}
```

v4 consumer : pre-instantiate in a separate JS file because `@plugin`
itself does not accept option arguments :

```js
// plugins/docs.js
module.exports = require('./markdown')({ className: 'docs' })
```

```css
@plugin "./plugins/docs.js";
```

## Example 9 : v4 CSS-Native Equivalent

Same Skew utilities as Example 1, but written as CSS :

`app.css` :

```css
@import "tailwindcss";

@utility skew-5deg {
  transform: skewY(-5deg);
}

@utility skew-10deg {
  transform: skewY(-10deg);
}

@utility skew-15deg {
  transform: skewY(-15deg);
}
```

Functional form :

```css
@theme {
  --skew-5: -5deg;
  --skew-10: -10deg;
  --skew-15: -15deg;
}

@utility skew-* {
  transform: skewY(--value(--skew-*));
}
```

## Example 10 : npm Package Layout

Project tree :

```
my-tailwind-skew/
  package.json
  src/
    index.js
  README.md
  LICENSE
```

`package.json` :

```json
{
  "name": "@example/tailwind-skew",
  "version": "1.0.0",
  "description": "Skew utilities for Tailwind CSS",
  "license": "MIT",
  "main": "./src/index.js",
  "exports": {
    ".": "./src/index.js"
  },
  "files": ["src"],
  "peerDependencies": {
    "tailwindcss": ">=3.4 <5"
  },
  "keywords": ["tailwindcss", "tailwindcss-plugin", "skew"]
}
```

`src/index.js` :

```js
const plugin = require('tailwindcss/plugin')

module.exports = plugin(
  function ({ matchUtilities, theme }) {
    matchUtilities(
      {
        'skew-y': (value) => ({ transform: `skewY(${value})` }),
        'skew-x': (value) => ({ transform: `skewX(${value})` }),
      },
      {
        values: theme('skew'),
        supportsNegativeValues: true,
      }
    )
  },
  {
    theme: {
      skew: {
        0: '0deg',
        5: '5deg',
        10: '10deg',
        15: '15deg',
        20: '20deg',
      },
    },
  }
)
```

Publish :

```bash
npm publish --access public
```

Consumer-side install :

```bash
npm install @example/tailwind-skew
```

v3 `tailwind.config.js` :

```js
module.exports = {
  plugins: [require('@example/tailwind-skew')],
}
```

v4 CSS :

```css
@import "tailwindcss";
@plugin "@example/tailwind-skew";
```
