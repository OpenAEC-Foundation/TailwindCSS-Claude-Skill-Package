# Methods : Tailwind Functional Utilities (v4 only)

Complete grammar for `--value()`, `--modifier()`, `--alpha()`, `--spacing()` inside `@utility` directives.

## 1. `--value(<arg>)` signatures

| Form | Example | Matches |
|------|---------|---------|
| Theme namespace | `--value(--color-*)` | Any `name-foo` where `--color-foo` exists in `@theme` |
| Bare data type | `--value(integer)` | `name-2`, `name-76`, any unitless integer |
| Bare type alt | `--value(number)`, `--value(ratio)`, `--value(percentage)` | Matching bare numeric forms |
| Literal list | `--value("inherit", "initial", "unset", "revert")` | `name-inherit`, `name-initial`, etc. |
| Arbitrary type | `--value([integer])`, `--value([length])`, `--value([color])` | `name-[<value>]` with bracket syntax in markup |
| Wildcard arbitrary | `--value([*])` | Any bracketed value, no type check |

### Supported bare data types

`number`, `integer`, `ratio`, `percentage`.

### Supported arbitrary data types (used inside `[ ]`)

`absolute-size`, `angle`, `bg-size`, `color`, `family-name`, `generic-name`, `image`, `integer`, `length`, `line-width`, `number`, `percentage`, `position`, `ratio`, `relative-size`, `url`, `vector`, `*`.

### Stacking multiple `--value()` lines

ALWAYS list every accepted form on its own property line within `@utility` :

```css
@utility tab-* {
  tab-size: --value(--tab-size-*);
  tab-size: --value(integer);
  tab-size: --value([integer]);
  tab-size: --value([*]);
}
```

The compiler picks the first form whose argument shape matches the utility used in markup. The order in the `@utility` block is the priority order.

## 2. `--modifier(<arg>)` signatures

Same argument shapes as `--value()`. Used in the same `@utility` block to handle the slash-suffix.

| Markup | `--modifier()` reads |
|--------|----------------------|
| `text-base/relaxed` | the string `relaxed` (matches `--leading-relaxed`) |
| `text-base/6` | the string `6` (matches bare number 6 or `--leading-6` if registered) |
| `text-base/[1.5]` | the bracketed value `1.5` (matches `[number]` or `[length]`) |
| `text-base` (no slash) | undefined; matches `--default(...)` if provided, else empty |

### `--default(<value>)` for fallback

```css
@utility text-* {
  font-size: --value(--text-*, [length]);
  line-height: --modifier(--leading-*, [length], --default(1));
}
```

When no modifier present, `line-height` becomes `1`.

## 3. `--alpha(var(--color-*) / <percentage>)`

### Signature

```
--alpha(var(--color-NAME) / N%)
```

### Compilation

```css
.x { color: --alpha(var(--color-lime-300) / 50%); }
```

compiles to :

```css
.x { color: color-mix(in oklab, var(--color-lime-300) 50%, transparent); }
```

ALWAYS pass a `var(--color-*)` reference, NOT a raw hex or rgb. The function requires a CSS-variable colour because the underlying `color-mix(in oklab, ...)` expects an interpolation-stable form.

## 4. `--spacing(<number>)`

### Signature

```
--spacing(N)
```

where `N` is a positive number (integer or fractional).

### Compilation

```css
.x { margin: --spacing(4); }
```

compiles to :

```css
.x { margin: calc(var(--spacing) * 4); }
```

ALWAYS use inside `calc()` for non-trivial arithmetic :

```css
.x { padding-block: calc(--spacing(4) - 1px); }
```

NEVER use `--spacing()` outside `calc()` for plain values; `class="py-4"` in markup is shorter and clearer.

## 5. Composing multi-property utilities

A single `@utility` block can declare multiple properties. ALL `--value()` and `--modifier()` calls within the block share the same parameter source (the part of the utility class after the prefix and before the slash).

```css
@utility text-* {
  font-size: --value(--text-*, [length]);            /* uses pre-slash */
  line-height: --modifier(--leading-*, [length]);     /* uses post-slash */
}

@utility shadow-glow-* {
  box-shadow: 0 0 --value([length], --default(8px)) --alpha(var(--color-*) / 50%);
  /* Note : a single `*` here is impossible because the utility has only ONE parameter slot.
     For multi-parameter utilities, layer multiple utilities or use arbitrary values. */
}
```

NEVER expect a single utility to expose two independent parameter slots; the `*` placeholder is single-valued.

## 6. `@utility` directive grammar

```
@utility <name> {
  <css-declarations>
}

@utility <name>-* {
  <css-declarations-using-value-or-modifier>
}
```

| Form | Meaning |
|------|---------|
| `@utility no-scrollbar { ... }` | Static utility, no parameter, used as `class="no-scrollbar"` |
| `@utility tab-* { tab-size: --value(integer); }` | Functional utility, takes parameter via `--value()` |
| `@utility text-* { font-size: --value(...); line-height: --modifier(...); }` | Functional with modifier |

### Variant pipeline integration

Utilities defined with `@utility` participate automatically in :
- Responsive variants : `sm:tab-2`, `md:tab-[4]`.
- State variants : `hover:tab-8`, `focus:tab-github`.
- Dark mode : `dark:tab-4`.
- Arbitrary variants : `[&>p]:tab-2`.

NEVER need to register the utility separately for variant generation; the `@utility` directive does it.

### `@layer utilities { .name { ... } }` comparison (legacy)

The old form :

```css
@layer utilities {
  .tab-2 { tab-size: 2; }
  .tab-4 { tab-size: 4; }
  .tab-8 { tab-size: 8; }
}
```

ALWAYS prefer `@utility` over `@layer utilities`. The legacy form :
- Requires enumerating every value (no dynamic resolution).
- Does NOT support the functional CSS functions (`--value`, `--modifier`).
- Does NOT integrate with variant generation as cleanly.
- Cannot be overridden by user-applied utilities without `!important`.

## 7. Removed/Deprecated `theme()` function

`theme(colors.red.500)` with dot-notation is DEPRECATED in v4. Two paths forward :

ALWAYS use direct `var()` :

```css
.x { color: var(--color-red-500); }
```

OR, if `theme()` syntax is preferred, use the CSS-variable form :

```css
.x { color: theme(--color-red-500); }
```

NEVER write `theme(colors.red.500)` in new v4 code. The build will warn or emit invalid CSS.

## 8. v3 vs v4 plugin authoring comparison

| Capability | v3 | v4 |
|------------|----|----|
| Static utility | `addUtilities({ '.no-scrollbar': { ... } })` | `@utility no-scrollbar { ... }` |
| Functional utility | `matchUtilities({ tab: (v) => ({ tabSize: v }) }, { values: theme('tabSize') })` | `@utility tab-* { tab-size: --value(--tab-size-*); }` |
| Component class | `addComponents({ '.btn': { ... } })` | `@utility btn { ... }` (preferred) or `@layer components` |
| Custom variant | `addVariant('hocus', ['&:hover', '&:focus'])` | `@custom-variant hocus (&:hover, &:focus);` |
| Parameterised variant | `matchVariant('nth', (v) => `&:nth-child(${v})`)` | `@custom-variant nth-* (&:nth-child(--value(integer)));` |
| Theme access | `theme('colors.red.500')` | `var(--color-red-500)` |
| `theme()` (compat) | yes | yes but DEPRECATED with dot-notation |
| Modifier (slash) handling | not directly | `--modifier(...)` |
| Default modifier | not directly | `--default(...)` |
| Alpha-from-variable | manual `rgba()` / `color-mix()` | `--alpha(var(--color-*) / N%)` |
| Spacing-scale calc | `theme('spacing.4')` | `--spacing(4)` |

ALWAYS choose v4 CSS-first when both forms are available. Reserve v3 JS plugins for projects pinned to v3.

## Sources

- https://tailwindcss.com/docs/adding-custom-styles (`@utility`, `--value()`, `--modifier()`)
- https://tailwindcss.com/docs/functions-and-directives (`--alpha()`, `--spacing()`, `theme()` deprecation)
- https://tailwindcss.com/docs/theme (theme namespaces for `--value(--namespace-*)`)
- https://tailwindcss.com/blog/tailwindcss-v4 (v4 dynamic utility values rationale)

Verified 2026-05-19.
