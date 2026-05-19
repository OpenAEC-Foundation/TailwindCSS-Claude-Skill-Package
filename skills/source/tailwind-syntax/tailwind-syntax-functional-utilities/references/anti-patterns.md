# Anti-Patterns : Tailwind Functional Utilities

Six v4-specific failures, plus the canonical v3-backport mistake.

## 1. Trying to use `--value()`/`--modifier()`/`--alpha()`/`--spacing()` in v3

### Symptom

A v3 project's CSS includes :

```css
@utility tab-* {
  tab-size: --value(integer);
}
```

The build succeeds (PostCSS doesn't reject unknown CSS functions). The browser parses the rule, sees `tab-size: --value(integer)`, fails CSS variable resolution (no such variable), drops the declaration. The utility does NOTHING.

### Root cause

v3 (JIT engine) does not understand the `@utility` directive OR the `--value()` family of CSS functions. The `@utility` line is treated as raw CSS and emitted as-is. The `--value(integer)` is interpreted by the browser as a custom-property reference (because of the `--` prefix), but no such property is registered, so the declaration is invalid and discarded.

### Fix

ALWAYS check the Tailwind version BEFORE writing functional-utility CSS. In v3, use the JS plugin API :

```js
const plugin = require('tailwindcss/plugin')
module.exports = plugin(function({ matchUtilities, theme }) {
  matchUtilities(
    { tab: (v) => ({ tabSize: v }) },
    { values: theme('tabSize'), type: 'integer' }
  )
})
```

In v4, use `@utility name-* { ... }` with `--value(...)`. See [tailwind-impl-plugins-custom] for v3 patterns and [tailwind-impl-config-v4] for v4 directives.

## 2. Calling `--value()` with no argument

### Symptom

```css
@utility tab-* {
  tab-size: --value();
}
```

Build emits a parse warning OR the utility produces `tab-size: undefined` which the browser drops.

### Root cause

`--value()` requires at least one argument that tells the compiler what to look for. An empty call has no type and no namespace to match against.

### Fix

ALWAYS provide a type hint, namespace, or literal list :

```css
@utility tab-* {
  tab-size: --value(integer);          /* type hint */
  tab-size: --value(--tab-size-*);     /* namespace */
  tab-size: --value("inherit");        /* literal */
  tab-size: --value([*]);              /* wildcard arbitrary */
}
```

## 3. Mixing literal strings and bare types in one `--value()` call

### Symptom

```css
@utility tab-* {
  tab-size: --value(integer, "inherit");
}
```

The compiler accepts the line but resolves `tab-inherit` ambiguously (sometimes literal, sometimes treated as broken integer). Behaviour is unstable across compiler revisions.

### Root cause

A single `--value()` call expects EITHER a single type / namespace / wildcard form OR a list of literal strings. Mixing types and literals in one call is unsupported.

### Fix

ALWAYS split into separate property lines :

```css
@utility tab-* {
  tab-size: --value("inherit", "initial", "unset", "revert");
  tab-size: --value(integer);
  tab-size: --value([integer]);
}
```

The cascade picks the matching form. Order from most-specific to least-specific.

## 4. Expecting `--value([length])` to match bare unitless numbers

### Symptom

```css
@utility margin-y-* {
  margin-block: --value([length]);
}
```

Markup `class="margin-y-4"` does NOTHING. Markup `class="margin-y-[4]"` also does nothing.

### Root cause

- `[length]` requires bracket syntax in markup : `margin-y-[4px]`, `margin-y-[2rem]`. Bare `margin-y-4` is unitless and matches type `integer`, not `[length]`.
- `[4]` is unitless inside brackets ; `[length]` requires a unit (px, rem, em, etc.).

### Fix

ALWAYS pair bare-numeric and bracket forms when both are wanted :

```css
@utility margin-y-* {
  margin-block: --spacing(--value(integer));   /* margin-y-4 multiplies spacing scale */
  margin-block: --value([length]);             /* margin-y-[12px] takes raw length */
}
```

Or use `--value(number)` to accept fractional unitless values.

## 5. Using `--alpha()` with a raw colour literal

### Symptom

```css
@utility brand-tint {
  background: --alpha(#1da1f2 / 50%);
}
```

Build emits a parse error OR silently produces an invalid declaration.

### Root cause

`--alpha()` expects the colour argument to be a CSS-variable reference (`var(--color-*)`). It uses `color-mix(in oklab, ...)` under the hood which works well with variable-resolved colours; raw hex breaks the interpolation contract.

### Fix

ALWAYS register the colour in `@theme` first :

```css
@theme {
  --color-twitter: #1da1f2;
}

@utility brand-tint {
  background: --alpha(var(--color-twitter) / 50%);
}
```

For one-off arbitrary colours that should not be themed, use `color-mix()` directly :

```css
@utility brand-tint {
  background: color-mix(in oklab, #1da1f2 50%, transparent);
}
```

## 6. Using `--spacing()` outside `calc()` for plain values

### Symptom

```css
@utility margin-y-* {
  margin-block: --spacing(--value(integer));
}
```

Works correctly. But the equivalent markup `class="my-4"` is shorter and clearer.

### Root cause

`--spacing()` shines INSIDE `calc()` for compound arithmetic (`calc(--spacing(4) - 1px)`). For plain "n * spacing-base" values, the built-in `my-4` utility already does this.

### Fix

NEVER reinvent `my-*`, `mx-*`, `pt-*`, etc. in custom `@utility` blocks unless they need behaviour the built-ins lack. Reserve `--spacing()` for :
- Arbitrary-value calc combinations : `class="py-[calc(--spacing(4)-1px)]"`.
- Custom utilities that mix spacing with other tokens : `--spacing(4)` in a shadow offset.

## 7. Defining a static utility with `-*` suffix and no parameter use

### Symptom

```css
@utility no-scrollbar-* {
  scrollbar-width: none;
}
```

Markup `class="no-scrollbar-foo"`, `class="no-scrollbar-bar"` all produce the same CSS. The `-*` suffix is meaningless.

### Root cause

The `-*` is a placeholder for a parameter. A `@utility` block that does not call `--value()` or `--modifier()` has no use for the parameter; the suffix becomes noise.

### Fix

ALWAYS drop the `-*` for static utilities :

```css
@utility no-scrollbar {
  scrollbar-width: none;
}
@utility no-scrollbar::-webkit-scrollbar {
  display: none;
}
```

Markup : `class="no-scrollbar"`.

## 8. Re-implementing `matchUtilities` in v4

### Symptom

A v3-to-v4 migration leaves a `plugins/tab.js` file untouched and the v4 project loads it via `@config "./tailwind.config.js"`. The plugin still works (backward compat), but the project now has both :
- A JS plugin defining `tab-*` utilities.
- A v4 `@utility tab-*` block in CSS.

The two compete; debugging which one applies becomes hard.

### Root cause

v4 supports v3 plugins for migration ease, but the new CSS-first approach should replace them. Leaving both creates duplicate definitions.

### Fix

ALWAYS migrate v3 `matchUtilities()` to v4 `@utility` once the project is on v4 :

```js
// DELETE this v3 plugin file once migrated
matchUtilities({ tab: (v) => ({ tabSize: v }) }, { values: theme('tabSize') })
```

Replace with :

```css
@utility tab-* {
  tab-size: --value(--tab-size-*);
  tab-size: --value(integer);
  tab-size: --value([integer]);
}
```

And remove the plugin from `tailwind.config.js` (or delete the JS config entirely if no other plugins remain).

## 9. Expecting `@utility` blocks to override built-in utilities

### Symptom

```css
@utility text-base {
  font-size: 18px;
  line-height: 1.5;
}
```

Built-in `text-base` in v4 sets `font-size: 1rem`. The custom block was meant to override. Sometimes the built-in wins, sometimes the custom wins, depending on source order.

### Root cause

`@utility` participates in the utilities layer. If multiple definitions exist for the same class name, the LATER definition in CSS source order wins. Without controlling source order explicitly, the override is fragile.

### Fix

ALWAYS use a different name for custom utilities :

```css
@utility text-base-custom {
  font-size: 18px;
  line-height: 1.5;
}
```

OR, when truly overriding, ensure the `@utility` comes AFTER `@import "tailwindcss"` in source order, and accept that future Tailwind updates could re-introduce conflicts.

NEVER name a custom utility identically to a built-in unless the intent is to fully replace and the override is locked in.

## 10. Forgetting that `--value(--namespace-*)` requires the wildcard

### Symptom

```css
@theme {
  --tab-size-github: 8;
}

@utility tab-* {
  tab-size: --value(--tab-size-github);
}
```

Markup `class="tab-github"` does nothing. Other markup `class="tab-prose"` also does nothing.

### Root cause

`--value(--tab-size-github)` matches ONLY the literal name `--tab-size-github`. It does not iterate the namespace. The utility resolves only when the user writes `class="tab-github"` AND the compiler's parameter resolution matches exactly.

Wait : even then, the resolution does not work because `--value()` expects either a wildcard (`--namespace-*`) or a literal-list / type form. Passing a specific variable name without `*` is malformed.

### Fix

ALWAYS use the wildcard form to read the parameter against a namespace :

```css
@utility tab-* {
  tab-size: --value(--tab-size-*);
}
```

The `*` in BOTH the `@utility` name AND the `--value()` namespace tells the compiler : "the suffix of the utility is the suffix of the variable name."

## Sources

- https://tailwindcss.com/docs/adding-custom-styles (`@utility`, `--value()`, `--modifier()`)
- https://tailwindcss.com/docs/functions-and-directives (`--alpha()`, `--spacing()`)
- https://tailwindcss.com/docs/upgrade-guide (v3 to v4 plugin migration)
- LESSONS.md L-001 to L-006

Verified 2026-05-19.
