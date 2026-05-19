# Tailwind CSS Arbitrary Values: Complete Reference

Verified 2026-05-19 against tailwindcss.com/docs/styling-with-utility-classes and /docs/adding-custom-styles.

## Five Arbitrary Forms

### 1. Value-arbitrary (most common)

```
{utility-prefix}-[{value}]
```

Replaces the value portion of any utility.

| Utility | Arbitrary form | Compiles to |
|---------|----------------|-------------|
| `bg-{color}` | `bg-[#1da1f2]` | `background-color: #1da1f2` |
| `text-{size}` | `text-[14.5px]` | `font-size: 14.5px` |
| `w-{size}` | `w-[calc(100%-2rem)]` | `width: calc(100% - 2rem)` |
| `top-{spacing}` | `top-[-113px]` | `top: -113px` (negative auto-detected) |
| `grid-cols-{N}` | `grid-cols-[24rem_2.5rem_1fr]` | `grid-template-columns: 24rem 2.5rem 1fr` |
| `rotate-{deg}` | `rotate-[7.5deg]` | `rotate: 7.5deg` |
| `text-{size}/{leading}` | `text-[14.5px]/[1.3]` | `font-size: 14.5px; line-height: 1.3` |

### 2. Modifier-arbitrary (custom selector)

```
[{selector}]:{utility}
```

Custom selector applied to the utility. `&` represents the styled element. `_` represents a space inside the selector.

| Pattern | Selector | Compiles to |
|---------|----------|-------------|
| `[&_p]:text-red-500` | `& p` | descendant selector |
| `[&>li]:list-disc` | `& > li` | direct-child selector |
| `[&~p]:hidden` | `& ~ p` | general-sibling selector |
| `[&+p]:mt-4` | `& + p` | adjacent-sibling selector |
| `[&.is-active]:bg-blue-500` | `&.is-active` | compound class selector |
| `[&:nth-child(3n+1)]:underline` | `&:nth-child(3n+1)` | pseudo-class selector |
| `[&::-webkit-scrollbar]:hidden` | `&::-webkit-scrollbar` | vendor pseudo-element |

### 3. Variant-arbitrary (attribute or query)

```
{variant-prefix}-[{value}]:{utility}
[{query}]:{utility}
```

Attribute-value matches or full at-rule queries.

| Pattern | Compiles to |
|---------|-------------|
| `data-[state=open]:rotate-180` | `&[data-state="open"] { rotate: 180deg }` |
| `aria-[sort=ascending]:bg-blue-50` | `&[aria-sort="ascending"] { background-color: #eff6ff }` |
| `supports-[display:grid]:grid` | `@supports (display: grid) { & { display: grid } }` |
| `not-supports-[backdrop-filter]:bg-white/95` | `@supports not (backdrop-filter) { ... }` (v4) |
| `[@media(prefers-reduced-data:reduce)]:bg-none` | `@media (prefers-reduced-data: reduce) { ... }` |
| `[@container(width>=600px)]:grid` | `@container (width >= 600px) { ... }` |

### 4. Arbitrary property (no utility namespace)

```
[{css-property}:{value}]
```

Direct CSS declaration. No utility prefix at all.

| Pattern | Compiles to |
|---------|-------------|
| `[mask-type:luminance]` | `mask-type: luminance` |
| `[scroll-snap-stop:always]` | `scroll-snap-stop: always` |
| `[contain:layout]` | `contain: layout` |
| `[content-visibility:auto]` | `content-visibility: auto` |
| `[view-transition-name:hero]` | `view-transition-name: hero` |
| `[appearance:textfield]` | `appearance: textfield` |
| `hover:[mask-type:alpha]` | hover-scoped arbitrary property |

### 5. Type-hinted arbitrary (disambiguation)

```
{utility-prefix}-[{type}:{value}]
```

Type hint goes BEFORE the value, separated by colon.

| Hint | Used for | Example |
|------|----------|---------|
| `length:` | sizes (font-size, width, height) when otherwise ambiguous | `text-[length:var(--size)]` |
| `color:` | colors when otherwise ambiguous | `text-[color:var(--accent)]` |
| `image:` | background images | `bg-[image:url('/hero.png')]`, `bg-[image:linear-gradient(...)]` |
| `family-name:` | font-family from CSS var | `font-[family-name:var(--my-font)]` |
| `position:` | background-position | `bg-[position:50%_50%]` |
| `size:` | background-size | `bg-[size:cover]`, `bg-[size:200px_100px]` |
| `number:` | unitless numbers | `leading-[number:1.5]` |
| `percentage:` | percentages | `tracking-[percentage:5%]` |
| `angle:` | rotations | `rotate-[angle:15deg]` |

## CSS Variable Shorthand (v4 only)

```
{utility-prefix}-({--var-name})
{utility-prefix}-({type}:--{var-name})
```

The parens form is v4-only. v3 must use brackets with `var()`.

| v3 syntax | v4 syntax (shorthand) |
|-----------|------------------------|
| `bg-[var(--brand)]` | `bg-(--brand)` |
| `text-[var(--fg)]` | `text-(--fg)` |
| `w-[var(--col-1)]` | `w-(--col-1)` |
| `text-[color:var(--accent)]` | `text-(color:--accent)` |
| `text-[length:var(--size)]` | `text-(length:--size)` |
| `font-[family-name:var(--font)]` | `font-(family-name:--font)` |

## Space and Comma Encoding

CSS class names cannot contain literal spaces. Tailwind uses `_` as the in-arbitrary space substitute.

| Pattern | Compiles to |
|---------|-------------|
| `grid-cols-[1fr_2fr_1fr]` | `grid-template-columns: 1fr 2fr 1fr` |
| `bg-[200px_100px]` | `background-position: 200px 100px` (or size, hint with `length:`) |
| `[transition:transform_0.3s_ease]` | `transition: transform 0.3s ease` |

For commas inside arbitrary values, v3 and v4 differ in CONVENTION (not parsing) :

| Version | Convention | Example |
|---------|------------|---------|
| v3.4 | commas allowed | `grid-cols-[max-content,1fr,max-content]` |
| v4.0+ | prefer underscores | `grid-cols-[max-content_1fr_max-content]` |

The v4 upgrade tool rewrites commas to underscores in grid-template patterns.

## Class Detection Rules (CRITICAL)

ALWAYS write class names as complete literal tokens in source files. The Tailwind compiler scans source files (HTML, JSX, TSX, Vue, Svelte, etc.) as plain text and identifies classes by literal match.

### What works (literal tokens)

```jsx
<div className="bg-[#1da1f2]" />
<div className={isPrimary ? "bg-[#1da1f2]" : "bg-[#888]"} />
const STYLES = { primary: "bg-[#1da1f2]", secondary: "bg-[#888]" }
<div className={STYLES[variant]} />
```

### What fails (concatenation, interpolation, runtime composition)

```jsx
<div className={`bg-[${color}]`} />              // NEVER generates
<div className={`bg-${theme}-500`} />            // NEVER generates
<div className={"bg-" + colorName} />            // NEVER generates
<div className={cn(`bg-[#${hex}]`)} />           // NEVER generates
```

### v4 safelist escape hatch

```css
@import "tailwindcss";
@source inline("bg-[#{1da1f2,888,B91C1C}]");
@source inline("{hover:,}bg-{red,blue,green}-{50,{100..900..100},950}");
```

`@source inline()` accepts brace expansion. The v3 equivalent is `safelist: [...]` in `tailwind.config.js` (removed in v4).

## v3 vs v4 Summary

| Aspect | v3.4 | v4.0+ |
|--------|------|-------|
| Square-bracket arbitrary | yes | yes |
| Modifier-arbitrary `[&...]` | yes | yes |
| Variant-arbitrary `data-[...]` | yes | yes |
| Arbitrary property `[prop:value]` | yes | yes |
| Type hints | yes | yes (+ parens form `type:--var`) |
| CSS var brackets `[var(--c)]` | yes | yes (still works) |
| CSS var parens shorthand `(--c)` | no | yes |
| Comma in grid-template arbitrary | preferred | accepted ; tool rewrites to `_` |
| Performance | baseline JIT | ~5x full / ~100x incremental |
| Safelist | `safelist: [...]` in config | `@source inline("...")` in CSS |
