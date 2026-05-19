# Examples : Tailwind Functional Utilities

Working CSS plus the markup that triggers each form. v4 only.

## 1. Hello-world functional utility

```css
@import "tailwindcss";

@utility tab-* {
  tab-size: --value(integer);
}
```

```html
<pre class="tab-2">
function example() {
  return 'two-space tab stop';
}
</pre>
<pre class="tab-8">
function example() {
  return 'eight-space tab stop';
}
</pre>
```

Adding more values requires no CSS change. `class="tab-12"`, `class="tab-19"` all work via dynamic integer matching.

## 2. Theme-namespace utility (named values)

```css
@import "tailwindcss";

@theme {
  --tab-size-default: 4;
  --tab-size-prose: 2;
  --tab-size-github: 8;
}

@utility tab-* {
  tab-size: --value(--tab-size-*);
}
```

```html
<pre class="tab-default">default 4-space</pre>
<pre class="tab-prose">comfortable 2-space for prose</pre>
<pre class="tab-github">github-style 8-space</pre>
```

New names register in `@theme`; the `@utility` block needs no change.

## 3. Combined utility (theme + bare + arbitrary)

```css
@import "tailwindcss";

@theme {
  --tab-size-github: 8;
}

@utility tab-* {
  tab-size: --value(--tab-size-*);   /* theme key */
  tab-size: --value(integer);        /* bare integer */
  tab-size: --value([integer]);      /* arbitrary integer */
  tab-size: --value([*]);            /* arbitrary anything */
}
```

```html
<pre class="tab-github">named</pre>
<pre class="tab-2">bare</pre>
<pre class="tab-[5]">bracketed integer</pre>
<pre class="tab-[1.5em]">bracketed anything (last-resort)</pre>
```

ALWAYS order from most-specific to least-specific. The compiler picks the first match.

## 4. Literal-keyword utility

```css
@utility tab-* {
  tab-size: --value("inherit", "initial", "unset", "revert");
}
```

```html
<pre class="tab-inherit">inherits from parent</pre>
<pre class="tab-initial">CSS initial value</pre>
```

Stacks alongside other forms :

```css
@utility tab-* {
  tab-size: --value("inherit", "initial", "unset", "revert");
  tab-size: --value(integer);
  tab-size: --value([integer]);
}
```

## 5. Slash-modifier utility (text + line-height)

```css
@import "tailwindcss";

@theme {
  --text-base: 1rem;
  --text-lg: 1.125rem;
  --text-xl: 1.25rem;
  --leading-tight: 1.25;
  --leading-normal: 1.5;
  --leading-relaxed: 1.625;
}

@utility text-* {
  font-size: --value(--text-*, [length]);
  line-height: --modifier(--leading-*, [length], [*]);
}
```

```html
<p class="text-base">base size, no line-height set</p>
<p class="text-base/relaxed">base size, leading 1.625</p>
<p class="text-base/[1.7]">base size, arbitrary 1.7 line-height</p>
<p class="text-lg/6">large, line-height 1.5rem (matches --leading-6 if registered)</p>
<p class="text-[20px]/[1.4]">arbitrary font-size and line-height</p>
```

## 6. Modifier with default value

```css
@utility text-* {
  font-size: --value(--text-*, [length]);
  line-height: --modifier(--leading-*, [length], --default(1.5));
}
```

```html
<p class="text-base">font 1rem, line-height defaults to 1.5</p>
<p class="text-base/2">font 1rem, line-height matched to --leading-2 or 2 if bare</p>
```

ALWAYS use `--default()` when the modifier-less form should still produce a sensible value.

## 7. `--alpha()` for opacity composition in custom utilities

```css
@import "tailwindcss";

@theme {
  --color-brand: oklch(0.62 0.22 250);
}

@utility brand-tint-* {
  background-color: --alpha(var(--color-brand) / --value(percentage));
}
```

```html
<div class="brand-tint-25">25% brand background</div>
<div class="brand-tint-50">50% brand background</div>
<div class="brand-tint-75">75% brand background</div>
```

Each variant emits a `color-mix(in oklab, var(--color-brand) N%, transparent)` declaration.

## 8. `--spacing()` for spacing-scale calc inside arbitrary

```html
<div class="py-[calc(--spacing(4)+8px)]">
  padding-block: calc(var(--spacing) * 4 + 8px)
  resolves to 1rem + 8px = 24px (assuming default --spacing 0.25rem)
</div>

<div class="mt-[calc(--spacing(8)-1px)]">
  margin-top: 2rem - 1px (for pixel-perfect alignment)
</div>
```

## 9. Multi-property functional utility

```css
@theme {
  --color-glow-cyan: oklch(0.78 0.13 195);
  --color-glow-magenta: oklch(0.78 0.27 0);
}

@utility glow-* {
  color: --value(--color-glow-*);
  text-shadow: 0 0 8px --alpha(--value(--color-glow-*) / 60%);
}
```

```html
<span class="glow-cyan">cyan glow</span>
<span class="glow-magenta">magenta glow</span>
```

Both `color` and `text-shadow` resolve from the SAME parameter ; the `--value(--color-glow-*)` call is repeated because each declaration is independent.

## 10. Migration : v3 `matchUtilities()` to v4 `@utility`

### v3 plugin (JavaScript)

```js
// plugin/tab.js
const plugin = require('tailwindcss/plugin')

module.exports = plugin(function({ matchUtilities, theme }) {
  matchUtilities(
    {
      tab: (value) => ({ tabSize: value })
    },
    {
      values: theme('tabSize'),
      type: 'integer',
      supportsNegativeValues: false,
    }
  )
}, {
  theme: {
    tabSize: {
      DEFAULT: 4,
      github: 8,
      prose: 2,
    }
  }
})
```

```js
// tailwind.config.js
module.exports = {
  plugins: [require('./plugin/tab.js')]
}
```

### v4 equivalent (CSS only)

```css
/* app.css */
@import "tailwindcss";

@theme {
  --tab-size-default: 4;
  --tab-size-github: 8;
  --tab-size-prose: 2;
}

@utility tab-* {
  tab-size: --value(--tab-size-*);
  tab-size: --value(integer);
  tab-size: --value([integer]);
}
```

No JS plugin needed. No `tailwind.config.js` needed.

## 11. Conditional / responsive use

Functional utilities participate in variants automatically :

```css
@utility tab-* {
  tab-size: --value(integer);
}
```

```html
<pre class="tab-2 sm:tab-4 dark:tab-8 hover:tab-[12]">
  Reads as : 2 on mobile, 4 from sm-up, 8 in dark mode, 12 on hover.
</pre>
```

Variant pipeline integration is automatic; no extra registration needed.

## 12. Sharing the same theme key across utilities

```css
@theme {
  --color-brand: oklch(0.62 0.22 250);
}

@utility brand-text {
  color: var(--color-brand);
}

@utility brand-tint-* {
  background-color: --alpha(var(--color-brand) / --value(percentage));
}

@utility brand-shadow-* {
  box-shadow: 0 0 --value([length]) --alpha(var(--color-brand) / 50%);
}
```

```html
<h2 class="brand-text">heading</h2>
<div class="brand-tint-30">light brand background</div>
<button class="brand-shadow-[12px]">glowing brand button</button>
```

ALWAYS centralise colours in `@theme` and reference via `var(--color-*)` in `@utility` blocks. Changing the brand colour in `@theme` updates all three utilities simultaneously.

## 13. Type-hint disambiguation example

```css
@utility size-* {
  height: --value([length]);
  width:  --value([length]);
  height: --value(--size-*);
  width:  --value(--size-*);
}
```

The `[length]` lines accept bracket arbitrary values : `size-[100px]`, `size-[5rem]`. The `--size-*` lines accept theme-registered named sizes : `size-card`, `size-modal` (if `--size-card`, `--size-modal` exist).

ALWAYS list all accepted forms; the cascade picks the matching one.

## 14. Anti-pattern fixed (literal vs bare numeric conflict)

WRONG :

```css
@utility tab-* {
  tab-size: --value(integer, "inherit");
}
```

`--value()` does NOT accept mixed type and literal arguments in one call.

CORRECT :

```css
@utility tab-* {
  tab-size: --value("inherit", "initial");
  tab-size: --value(integer);
  tab-size: --value([integer]);
}
```

ALWAYS split forms into separate lines.

## 15. Real-world example : `text-shadow-*` (v3 had no shadow util for text)

```css
@theme {
  --text-shadow-sm: 0 1px 2px rgb(0 0 0 / 0.1);
  --text-shadow-md: 0 2px 4px rgb(0 0 0 / 0.15);
  --text-shadow-lg: 0 4px 8px rgb(0 0 0 / 0.2);
}

@utility text-shadow-* {
  text-shadow: --value(--text-shadow-*);
  text-shadow: --value([*]);
}
```

```html
<h1 class="text-shadow-sm">subtle</h1>
<h1 class="text-shadow-lg">prominent</h1>
<h1 class="text-shadow-[2px_2px_8px_rgba(0,0,0,0.6)]">custom</h1>
```

v3 required a plugin for this. v4 handles it in 10 lines of CSS.
