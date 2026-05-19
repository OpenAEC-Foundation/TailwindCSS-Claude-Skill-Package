# tailwind-syntax-modern-utilities : Methods Reference

Complete v4-only utility signatures for the seven modern utility families.

## `transition-discrete`

Single utility. Compiles to `transition-behavior: allow-discrete`.

| Class | CSS |
|-------|-----|
| `transition-discrete` | `transition-behavior: allow-discrete` |
| `transition-normal` | `transition-behavior: normal` (default) |

Apply alongside a `transition-*` and a `duration-*` to make discrete-property transitions (`display`, `content-visibility`, `overlay`) animate instead of snap.

## `starting:` Variant

Compiles to `@starting-style { & { ... } }`.

| Pattern | Effect |
|---------|--------|
| `starting:opacity-0` | Element starts at `opacity: 0` when inserted into DOM |
| `starting:scale-95` | Element starts at `scale: 0.95` when entering |
| `starting:translate-y-2` | Element starts shifted before settling |

Used with `transition-*` + `transition-discrete` + final-state utilities to build CSS-only popover / dialog enter animations.

## `field-sizing-*`

| Class | CSS |
|-------|-----|
| `field-sizing-content` | `field-sizing: content` (resize to content) |
| `field-sizing-fixed` | `field-sizing: fixed` (default, ignore content) |

Applies to `<textarea>`, `<input>`, `<select>`. The element grows or shrinks to fit its content automatically.

## `scheme-*` (Color Scheme)

| Class | CSS |
|-------|-----|
| `scheme-normal` | `color-scheme: normal` |
| `scheme-dark` | `color-scheme: dark` |
| `scheme-light` | `color-scheme: light` |
| `scheme-light-dark` | `color-scheme: light dark` |
| `scheme-only-dark` | `color-scheme: only dark` |
| `scheme-only-light` | `color-scheme: only light` |

Tells the browser which colour palette to use for native form controls (date picker, scrollbar, checkbox).

## `font-stretch-*`

Named keyword utilities :

| Class | CSS |
|-------|-----|
| `font-stretch-ultra-condensed` | `font-stretch: ultra-condensed` (50%) |
| `font-stretch-extra-condensed` | `font-stretch: extra-condensed` (62.5%) |
| `font-stretch-condensed` | `font-stretch: condensed` (75%) |
| `font-stretch-semi-condensed` | `font-stretch: semi-condensed` (87.5%) |
| `font-stretch-normal` | `font-stretch: normal` (100%) |
| `font-stretch-semi-expanded` | `font-stretch: semi-expanded` (112.5%) |
| `font-stretch-expanded` | `font-stretch: expanded` (125%) |
| `font-stretch-extra-expanded` | `font-stretch: extra-expanded` (150%) |
| `font-stretch-ultra-expanded` | `font-stretch: ultra-expanded` (200%) |

Percentage and arbitrary forms :

| Class | CSS |
|-------|-----|
| `font-stretch-50%` | `font-stretch: 50%` |
| `font-stretch-125%` | `font-stretch: 125%` |
| `font-stretch-[83%]` | `font-stretch: 83%` (arbitrary) |
| `font-stretch-[var(--my-stretch)]` | CSS variable |

Requires a variable font with `wdth` axis (e.g. Inter Variable, Roboto Flex).

## `inset-shadow-*`

Inset variant of `shadow-*`. Composable with outer shadow + ring + inset-ring.

| Class | CSS box-shadow |
|-------|----------------|
| `inset-shadow-2xs` | `inset 0 1px var(--inset-shadow-2xs)` |
| `inset-shadow-xs` | `inset 0 1px 1px var(--inset-shadow-xs)` |
| `inset-shadow-sm` | `inset 0 2px 4px var(--inset-shadow-sm)` |
| `inset-shadow-md` | `inset 0 4px 6px var(--inset-shadow-md)` |
| `inset-shadow-lg` | `inset 0 8px 12px var(--inset-shadow-lg)` |
| `inset-shadow-xl` | `inset 0 12px 24px var(--inset-shadow-xl)` |
| `inset-shadow-2xl` | `inset 0 24px 48px var(--inset-shadow-2xl)` |
| `inset-shadow-none` | `inset-shadow: 0 0 #0000` |
| `inset-shadow-[...]` | arbitrary value |
| `inset-shadow-black/25` | colour modifier |

## `inset-ring-*`

Inset version of `ring-*`. Composable layer.

| Class | CSS |
|-------|-----|
| `inset-ring` | `inset-ring: 1px var(--color-blue-500)` (default) |
| `inset-ring-{0,1,2,4,8}` | `inset-ring-width: <n>px` |
| `inset-ring-[3px]` | arbitrary width |
| `inset-ring-blue-500` | `inset-ring-color: var(--color-blue-500)` |
| `inset-ring-blue-500/50` | colour with opacity |

## Shadow Layer Composition

Four independent shadow layers (each can be styled with width, colour, opacity) :

1. `shadow-*` : outer box-shadow
2. `inset-shadow-*` : inner box-shadow
3. `ring-*` : outer ring (set via `box-shadow`)
4. `inset-ring-*` : inner ring (set via `box-shadow`)

```html
<div class="shadow-lg inset-shadow-sm ring-2 ring-blue-500 inset-ring-1 inset-ring-white">
  Four-layer shadow composition
</div>
```

CSS output : single `box-shadow` declaration with four comma-separated values.

## `@starting-style` Specification Reference

```css
.popover {
  opacity: 1;
  transform: scale(1);
  transition: opacity 200ms, transform 200ms allow-discrete;
  transition-behavior: allow-discrete;
}

@starting-style {
  .popover {
    opacity: 0;
    transform: scale(0.95);
  }
}
```

The `starting:` variant in Tailwind generates the `@starting-style` block. `transition-discrete` adds `transition-behavior: allow-discrete`.

## Browser Baseline (2026)

| Feature | Chrome | Safari | Firefox |
|---------|--------|--------|---------|
| `@starting-style` | 117+ | 17.5+ | 129+ |
| `transition-behavior: allow-discrete` | 117+ | 17.4+ | 129+ |
| `field-sizing` | 123+ | 17.4+ | not yet (use polyfill) |
| `color-scheme` | 81+ | 13+ | 96+ |
| `font-stretch` | 60+ | 11+ | 9+ |
| Multi-shadow stacking | universal | universal | universal |

Source : MDN per property + `https://tailwindcss.com/blog/tailwindcss-v4`.

## Verified Sources

- https://tailwindcss.com/docs/transition-behavior
- https://tailwindcss.com/docs/field-sizing
- https://tailwindcss.com/docs/color-scheme
- https://tailwindcss.com/docs/font-stretch
- https://tailwindcss.com/docs/box-shadow (inset-shadow, inset-ring sections)
- https://developer.mozilla.org/en-US/docs/Web/CSS/@starting-style
- https://tailwindcss.com/blog/tailwindcss-v4

Last verified : 2026-05-19.
