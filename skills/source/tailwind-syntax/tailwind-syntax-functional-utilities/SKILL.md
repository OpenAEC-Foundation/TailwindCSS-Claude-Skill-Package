---
name: tailwind-syntax-functional-utilities
description: >
  Use when authoring custom Tailwind utilities in v4 with the `@utility`
  directive that need to accept parameters: a theme-namespace key (like
  `tab-github`, `tab-2`, `tab-4`), a bare integer/number/ratio/percentage,
  a literal keyword, an arbitrary bracketed value, or a slash-modifier
  (like `text-base/relaxed`).
  Prevents the common v3-to-v4 plugin-authoring mistakes: writing a
  `matchUtilities()` JS plugin where a native `@utility name-* { ... }`
  block would do, attempting to use `--value()` / `--modifier()` /
  `--alpha()` / `--spacing()` inside v3 (silently ignored as
  unrecognised CSS functions), and forgetting that `--value()` accepts
  multiple type forms simultaneously so users can pick theme key
  OR bare integer OR arbitrary bracket on the same utility.
  Covers: the four v4-only CSS functions `--value()`, `--modifier()`,
  `--alpha()`, `--spacing()`; type-hint syntax (`integer`, `[length]`,
  `--namespace-*`, literal strings, `[*]`); default-value syntax via
  `--default(<value>)`; combining theme keys with bare and arbitrary
  values in a single utility definition; the migration path from
  v3 `matchUtilities()` plugin to v4 `@utility` directive.
  v4 ONLY: these functions do not exist in v3 and silently fail there.
  Keywords: functional utilities, custom utility, @utility directive,
  --value, --modifier, --alpha, --spacing, theme namespace,
  bare values, arbitrary values, type hints, integer, length,
  ratio, percentage, color, custom plugin, matchUtilities,
  how do I make a custom utility, v4 plugin authoring,
  tab-size utility, --tab-size-*, --default, default value,
  modifier slash, text-base/6, line-height pairing,
  color-mix opacity, alpha compose, spacing calc.
license: MIT
compatibility: "Designed for Claude Code. Requires Tailwind CSS v4.0+."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# Tailwind CSS : Functional Utilities (v4 only)

Native CSS functions that turn `@utility name-* { ... }` blocks into parameterised utilities. Replaces v3 `matchUtilities()` JS plugin authoring with CSS-first directives. NEVER use these in v3 : the CSS functions are unrecognised tokens and the build silently emits broken CSS.

## Quick Reference

### Four functions

| Function | Purpose | Returns |
|----------|---------|---------|
| `--value(<type-or-namespace>)` | Resolve the suffix of a `name-*` utility against a type, literal list, theme namespace, or arbitrary bracket | The matched value |
| `--modifier(<type-or-namespace>)` | Resolve the part after `/` in `name-X/Y` against the same type system | The matched value, or `--default(...)` if none |
| `--alpha(var(--color-*) / <percentage>)` | Compose an alpha value into a colour via `color-mix()` | A colour with alpha applied |
| `--spacing(<number>)` | Multiply the spacing scale base by a number | `calc(var(--spacing) * <number>)` |

### Three invariants

1. ALWAYS define functional utilities with `@utility <name>-* { ... }` in v4. NEVER reach for `matchUtilities()` in a v4 project unless backward-compatibility with a v3 JS plugin is required.
2. ALWAYS list multiple `--value(...)` lines in the same `@utility` block to accept multiple value forms (theme key + bare + arbitrary). The CSS cascade picks the first match.
3. ALWAYS use `--alpha()` over `color-mix()` directly for clarity, and `--spacing()` over hand-written `calc(var(--spacing) * n)` for the same reason.

### Minimal canonical example

```css
@theme {
  --tab-size-github: 8;
}

@utility tab-* {
  tab-size: --value(--tab-size-*);   /* theme key form : tab-github */
  tab-size: --value(integer);        /* bare form      : tab-2, tab-4 */
  tab-size: --value([integer]);      /* arbitrary form : tab-[1], tab-[76] */
}
```

All three usages compile :
- `tab-github` resolves via the theme key `--tab-size-github = 8`.
- `tab-2`, `tab-4` resolve via the bare integer type.
- `tab-[1]`, `tab-[76]` resolve via the arbitrary integer bracket.

## Decision Tree 1 : When to use `@utility` + `--value()`

```
Q1. Does the project use Tailwind v4?
    no  → STOP. Use v3 `matchUtilities()` JS plugin instead. See
          [tailwind-impl-plugins-custom].
    yes → Q2

Q2. Does the utility need a parameter (suffix or modifier)?
    no  → ALWAYS use a static `@utility name { ... }` block (no `-*`
          suffix, no `--value()` call). For example `@utility no-scrollbar { ... }`.
    yes → Q3

Q3. Should the parameter resolve against a theme namespace
    (so consumers can extend by adding `--namespace-foo: ...` in `@theme`)?
    yes → ALWAYS include `--value(--namespace-*)` as a line.
    no  → Q4

Q4. Should the parameter accept a bare numeric or keyword value
    (without bracket syntax)?
    yes → ALWAYS include `--value(integer)` (or `number`, `ratio`,
          `percentage`, or `"literal1", "literal2"`).
    no  → Q5

Q5. Should the parameter accept arbitrary bracketed values?
    yes → ALWAYS include `--value([type])` (or `[*]` for unrestricted).
    no  → STOP. The utility has no value source ; consider whether it
          really needs `-*` suffix at all.
```

## Decision Tree 2 : Choosing the right type hint

```
Parameter is...

  → an integer (e.g., 2, 4, 76)                  ALWAYS `--value(integer)` + `[integer]`
  → a fractional number (e.g., 1.5, 2.25)        ALWAYS `--value(number)` + `[number]`
  → a ratio (e.g., 16/9)                          ALWAYS `--value(ratio)` + `[ratio]`
  → a percentage (e.g., 50%)                      ALWAYS `--value(percentage)` + `[percentage]`
  → a length (e.g., 12px, 1.5rem)                 ALWAYS `--value([length])` (length is bracket-only)
  → a colour                                      ALWAYS `--value([color])` (bracket) + `--value(--color-*)` (theme)
  → an angle (e.g., 45deg)                        ALWAYS `--value([angle])`
  → a URL (e.g., url('/x.png'))                   ALWAYS `--value([url])`
  → a literal keyword set                         ALWAYS `--value("a", "b", "c")`
  → anything (escape hatch)                       ALWAYS `--value([*])` (matches every arbitrary)
```

## Decision Tree 3 : Should I use `--modifier()`?

```
Q1. Does the utility need a slash-modifier (`name-X/Y`)?
    no  → STOP. Do not call `--modifier()`.
    yes → Q2

Q2. Should the modifier default to a value when omitted?
    yes → ALWAYS use `--modifier(<type>, --default(<value>))`.
    no  → ALWAYS use `--modifier(<type>)` and accept that the
          modifier-less form yields the empty value.
```

## Patterns

### Pattern 1 : Theme-namespace-driven utility

```css
@theme {
  --tab-size-github: 8;
  --tab-size-default: 4;
}

@utility tab-* {
  tab-size: --value(--tab-size-*);
}
```

Usage : `class="tab-github"`, `class="tab-default"`. NEW token values can be added to `@theme` without touching the `@utility` definition. ALWAYS pair theme namespace + matching utility namespace : `--tab-size-*` ↔ `tab-*`.

### Pattern 2 : Bare-value utility

```css
@utility tab-* {
  tab-size: --value(integer);
}
```

Usage : `class="tab-2"`, `class="tab-4"`, `class="tab-8"`. v4 dynamically accepts any integer literal in markup; no need to enumerate.

### Pattern 3 : Arbitrary-value utility

```css
@utility tab-* {
  tab-size: --value([integer]);
}
```

Usage : `class="tab-[1]"`, `class="tab-[76]"`. The bracket syntax in markup is mandatory; this form does NOT accept `tab-1` (use Pattern 2 for that).

### Pattern 4 : Combined (theme + bare + arbitrary)

ALWAYS combine all three forms in one block when the utility is exposed to users :

```css
@utility tab-* {
  tab-size: --value(--tab-size-*);   /* tab-github  */
  tab-size: --value(integer);        /* tab-2       */
  tab-size: --value([integer]);      /* tab-[76]    */
}
```

The CSS cascade resolves the first matching form. Users get full flexibility without separate utility names.

### Pattern 5 : Literal-keyword utility

```css
@utility tab-* {
  tab-size: --value("inherit", "initial", "unset", "revert");
}
```

Usage : `class="tab-inherit"`, `class="tab-initial"`. NEVER mix literal-keyword `--value` with bare-`integer` form on the same line; split into two lines :

```css
@utility tab-* {
  tab-size: --value("inherit", "initial", "unset");
  tab-size: --value(integer);
}
```

### Pattern 6 : Functional utility with modifier

```css
@theme {
  --text-base: 1rem;
  --text-lg: 1.125rem;
  --leading-relaxed: 1.625;
  --leading-tight: 1.25;
}

@utility text-* {
  font-size: --value(--text-*, [length]);
  line-height: --modifier(--leading-*, [length], [*]);
}
```

Usage :
- `text-base` : font-size 1rem, line-height unset (no modifier present).
- `text-base/relaxed` : font-size 1rem, line-height 1.625 (modifier matches `--leading-relaxed`).
- `text-base/6` : font-size 1rem, line-height matched against any-length bracket.
- `text-[18px]/[1.5]` : both font-size and line-height arbitrary.

The `--modifier(...)` reads everything after the `/` and resolves it against the listed type hints.

### Pattern 7 : Modifier with default value

```css
@utility text-* {
  font-size: --value(--text-*, [length]);
  line-height: --modifier(--leading-*, [length], --default(1));
}
```

When no modifier present (`text-base`), line-height resolves to `1`. The `--default(...)` value is emitted as a final fallback.

### Pattern 8 : `--alpha()` for colour-with-opacity composition

```css
@theme {
  --color-lime-300: oklch(0.87 0.18 142);
}

@utility highlight {
  background-color: --alpha(var(--color-lime-300) / 50%);
}
```

Compiles to :

```css
.highlight {
  background-color: color-mix(in oklab, var(--color-lime-300) 50%, transparent);
}
```

ALWAYS use `--alpha()` over hand-writing `color-mix()` for readability and forward-compat. NEVER apply this to a colour that is not registered in `@theme` or accessed via `var(...)` ; the function does not accept raw hex.

### Pattern 9 : `--spacing()` for spacing-scale calc

```css
.gutter {
  padding-block: --spacing(4);
}
```

Compiles to :

```css
.gutter {
  padding-block: calc(var(--spacing) * 4);
}
```

Useful in arbitrary-value brackets that mix spacing with other arithmetic :

```html
<div class="py-[calc(--spacing(4)-1px)]">...</div>
```

This produces `padding-block: calc(var(--spacing) * 4 - 1px);` : a hair shorter than the canonical `py-4` for pixel-fine layout adjustments.

NEVER use `--spacing()` outside `calc()` for plain values ; `py-4` is shorter and clearer.

### Pattern 10 : Migration from v3 `matchUtilities()`

v3 plugin authoring (JavaScript) :

```js
const plugin = require('tailwindcss/plugin')
module.exports = plugin(function({ matchUtilities, theme }) {
  matchUtilities(
    { tab: (value) => ({ tabSize: value }) },
    { values: theme('tabSize'), type: 'integer' }
  )
})
```

v4 equivalent (CSS, no JS plugin needed) :

```css
@theme {
  --tab-size-github: 8;
}

@utility tab-* {
  tab-size: --value(--tab-size-*);
  tab-size: --value(integer);
  tab-size: --value([integer]);
}
```

ALWAYS prefer the v4 CSS form. NEVER ship a JS plugin to v4 projects unless the plugin needs Node-only logic (e.g., reading external files at build time).

## Anti-patterns (summary; full list in references/anti-patterns.md)

NEVER do these :

1. NEVER use `--value()`, `--modifier()`, `--alpha()`, `--spacing()` in v3. The CSS functions are unrecognised; the build emits the raw token, browsers ignore the declaration, the utility silently does nothing.
2. NEVER write `@utility name-* { property: --value(--namespace-foo); }` with a specific suffix. ALWAYS use `--value(--namespace-*)` with the wildcard to match against the whole namespace.
3. NEVER omit the type-hint when calling `--value()`. The function REQUIRES a type or namespace; an empty `--value()` is a parse error.
4. NEVER combine `--value([length])` and bare numeric `--value(number)` expecting unitless `mt-1.5` to match `[length]`. Bracket-form ALWAYS requires the bracket; unitless numbers go to bare-numeric form.
5. NEVER use `--alpha()` to apply opacity to a non-themed colour. The function expects `var(--color-*)`. For arbitrary colours, use `color-mix()` directly.

## Reference Links

- [references/methods.md](references/methods.md) : complete type-hint catalogue, function signatures, `--default()` semantics, compilation rules
- [references/examples.md](references/examples.md) : multi-form utilities, slash-modifier patterns, real plugin migrations
- [references/anti-patterns.md](references/anti-patterns.md) : v3 backport attempts, missing type hints, bracket-vs-bare confusion

## Cross-references

- [tailwind-impl-config-v4](../../tailwind-impl/tailwind-impl-config-v4/SKILL.md) : the `@theme`, `@utility`, `@variant`, `@custom-variant`, `@source` family of v4 CSS directives
- [tailwind-impl-plugins-custom](../../tailwind-impl/tailwind-impl-plugins-custom/SKILL.md) : v3 `matchUtilities()` JS plugin authoring and the migration to v4 `@utility`
- [tailwind-core-architecture](../../tailwind-core/tailwind-core-architecture/SKILL.md) : utility-first doctrine that motivates custom utilities
- [tailwind-syntax-arbitrary-values](../tailwind-syntax-arbitrary-values/SKILL.md) : `[<value>]` bracket syntax in markup
- [tailwind-core-v3-vs-v4](../../tailwind-core/tailwind-core-v3-vs-v4/SKILL.md) : the JIT to Oxide engine shift that enables CSS-first functional utilities

## Sources

- https://tailwindcss.com/docs/adding-custom-styles (`@utility` directive, `--value()`, `--modifier()`)
- https://tailwindcss.com/docs/functions-and-directives (`--alpha()`, `--spacing()`, `theme()` deprecation)
- https://tailwindcss.com/blog/tailwindcss-v4 (v4 dynamic utility values rationale)
- https://tailwindcss.com/docs/theme (theme namespaces that `--value(--namespace-*)` reads from)

Verified 2026-05-19.
