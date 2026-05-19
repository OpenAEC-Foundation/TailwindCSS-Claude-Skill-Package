---
name: tailwind-syntax-modern-utilities
description: >
  Use when reaching for v4-only modern CSS utilities that were unavailable
  or required heavy custom code in v3: CSS-only enter and exit animations
  on dialogs and popovers via `@starting-style` plus the `starting:`
  variant and `transition-discrete`, auto-resizing textareas via
  `field-sizing-content`, native form-control colour scheme via
  `scheme-*`, variable-font width axes via `font-stretch-*`, and
  layered inset shadows and inset rings via `inset-shadow-*` and
  `inset-ring-*`.
  Prevents the canonical v3-mental-model traps: trying to animate
  `display: none` to `display: block` without `transition-discrete`
  (the property never transitions in v3), using a JS resize hook
  instead of `field-sizing-content` (verbose and laggy compared to
  the native CSS property), setting `color-scheme` via inline style
  when the `scheme-*` utility exists, and stacking multiple
  `box-shadow` declarations in custom CSS when `inset-shadow-*`
  plus outer `shadow-*` plus `ring-*` plus `inset-ring-*` compose
  four layers natively.
  Covers: `transition-discrete` and the `starting:` variant for
  `@starting-style` enter animations, `field-sizing-content` and
  `field-sizing-fixed` for textareas and inputs, the six `scheme-*`
  utilities (`normal`, `dark`, `light`, `light-dark`, `only-dark`,
  `only-light`), the nine named `font-stretch-*` values plus
  percentage and arbitrary forms, the full `inset-shadow-*` family
  (2xs/xs/sm/none plus color variants), and the `inset-ring-*` family
  with width and color, including layering rules across the four
  shadow layers (outer-shadow, inset-shadow, ring, inset-ring).
  v4 ONLY. These utilities do not exist in v3.
  Keywords: starting variant, @starting-style, transition-discrete,
  transition-behavior allow-discrete, popover enter animation,
  dialog enter animation, CSS only popover, field-sizing,
  field-sizing-content, field-sizing-fixed, auto-resizing textarea,
  color-scheme, scheme-light, scheme-dark, scheme-light-dark,
  native date picker dark, font-stretch, font-stretch-condensed,
  font-stretch-expanded, variable font width, inset-shadow,
  inset-ring, layered shadows, 4 shadow layers, multi-shadow,
  ring vs shadow, why does my textarea not resize,
  why does popover not animate, how to animate display none,
  date picker dark mode, native form control theming.
license: MIT
compatibility: "Designed for Claude Code. Requires Tailwind CSS v4.0+."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# Tailwind CSS : Modern Utilities (v4 only)

Seven v4-only utility families that surface modern CSS features. Each maps to a CSS specification that v3 either never wrapped or relied on plugins for. ALWAYS check the v4-version gate before adopting these; NEVER backport to v3 (silent failure).

## Quick Reference

### Seven utility families covered

| Family | Purpose | Underlying CSS spec |
|--------|---------|----------------------|
| `transition-discrete` | Allow discrete properties (display, visibility) to transition | `transition-behavior: allow-discrete` |
| `starting:` variant | Set initial values for enter animations | `@starting-style` |
| `field-sizing-content` | Auto-resize form controls based on content | `field-sizing: content` |
| `scheme-*` | Declare colour scheme for native form controls | `color-scheme` |
| `font-stretch-*` | Variable-font width axis | `font-stretch` (a.k.a. `font-width`) |
| `inset-shadow-*` | Inset box-shadows | `box-shadow: inset ...` |
| `inset-ring-*` | Inset ring (separate from outer ring) | `box-shadow: inset 0 0 0 N` |

### Three invariants

1. ALWAYS pair `starting:` with `transition-discrete` when animating an element whose visibility changes (popover, dialog, `details`); without `transition-discrete` the `display` change is instant and the start-style never animates.
2. ALWAYS use `field-sizing-content` instead of JS-based auto-resize hooks for `<textarea>` and `<input type="text">`; the native property is zero-runtime-cost and resilient to React re-renders.
3. ALWAYS use `scheme-*` (the v4 class name) when setting `color-scheme`. The masterplan-aliased `color-scheme-*` form does NOT exist; the canonical class names are `scheme-light`, `scheme-dark`, `scheme-light-dark`, etc.

## Decision Tree 1 : Choosing a transition strategy

```
Q1. Is the property being transitioned continuous (color, opacity, transform, size)?
    yes → ALWAYS use the standard `transition-*` family. NEVER need
          `transition-discrete` or `starting:`.
    no  → Q2

Q2. Is the property discrete (display, visibility, content-visibility)?
    yes → ALWAYS add `transition-discrete` to allow the discrete
          property change to participate in the transition. Pair with
          `starting:` to define the enter state.
    no  → check whether the property is animatable at all
```

## Decision Tree 2 : Enter-animation pattern for popover/dialog

```
Q1. Does the element use `[popover]` attribute or `<dialog>`?
    yes → ALWAYS use the pattern :
          1. transition the visual properties (opacity, transform)
          2. add `transition-discrete` so display can transition
          3. use `starting:` to define the off-screen initial state
    no  → consider whether the element really needs CSS-only animation,
          or whether a framework-driven enter (Framer Motion, React Transition Group)
          fits better
```

## Decision Tree 3 : Field-sizing usage

```
Q1. Is the field a <textarea>?
    yes → ALWAYS apply `field-sizing-content` to auto-grow with content,
          OR `field-sizing-fixed` to keep the natural sized box. Pair with
          `min-h-*` and `max-h-*` for bounds.
    no  → Q2

Q2. Is the field a single-line <input>?
    yes → `field-sizing-content` still works; the input grows horizontally
          to fit content (up to its container).
    no  → not applicable
```

## Decision Tree 4 : Picking a `scheme-*` value

```
Q1. Should native form controls follow OS preference?
    yes → `scheme-normal` (default; no class needed unless overriding)
    no  → Q2

Q2. Should native form controls always render in light mode?
    yes → `scheme-light` (allows fallback to dark) or `scheme-only-light` (strict)
    no  → Q3

Q3. Always dark?
    yes → `scheme-dark` (allows fallback) or `scheme-only-dark` (strict)
    no  → `scheme-light-dark` to declare BOTH supported, letting the UA pick
```

## Patterns

### Pattern 1 : `transition-discrete` + `starting:` for popover enter animation

```html
<button popovertarget="my-popover">Open</button>

<div
  id="my-popover"
  popover
  class="
    bg-white shadow-lg rounded-lg p-4
    opacity-100 translate-y-0
    starting:open:opacity-0 starting:open:-translate-y-4
    transition-all transition-discrete duration-200
  "
>
  Popover content
</div>
```

The element starts off-screen (`-translate-y-4`) and transparent (`opacity-0`) ONLY at the moment the popover opens (`starting:open:` triggers `@starting-style { :where([popover]:popover-open) { ... } }`). The `transition-discrete` utility allows the `display: none` to `display: block` transition. The browser interpolates from the start-style to the open-style over 200ms.

### Pattern 2 : `transition-discrete` for `display` animation

```html
<details class="group">
  <summary class="cursor-pointer">Expand</summary>
  <div class="
    overflow-hidden
    max-h-0 group-open:max-h-96
    transition-all transition-discrete duration-300
  ">
    Hidden content slides in.
  </div>
</details>
```

The `max-h-0` to `max-h-96` change is continuous; the additional `display` handling (controlled by `<details>` natively) becomes animatable thanks to `transition-discrete`.

### Pattern 3 : Auto-resizing textarea

```html
<textarea
  class="
    field-sizing-content
    min-h-[3rem] max-h-[20rem]
    w-full
    rounded-md border border-slate-300 p-2
  "
  rows="2"
>
Type here and the textarea grows.
</textarea>
```

ALWAYS pair `field-sizing-content` with `min-h-*` and `max-h-*` to prevent it from collapsing to one row OR ballooning past viewport when the user pastes a novel.

### Pattern 4 : Native form control colour scheme

```html
<html class="dark">
  <body class="bg-slate-900 text-white">
    <input type="date" class="scheme-dark border rounded p-2" />
    <input type="color" class="scheme-dark" />
    <select class="scheme-dark border rounded p-2">
      <option>One</option>
      <option>Two</option>
    </select>
  </body>
</html>
```

The native date picker, colour picker, and dropdown render in dark mode (matching the surrounding dark theme). Without `scheme-dark` these controls would render in OS-default colours, looking out of place inside a dark app shell.

Combined with the dark-mode variant :

```html
<html>
  <body>
    <input type="date" class="scheme-light dark:scheme-dark" />
  </body>
</html>
```

ALWAYS pair `scheme-*` with the matching `dark:scheme-*` when supporting both themes.

### Pattern 5 : Variable-font width

```html
<p class="font-stretch-condensed">Compact text for narrow layouts.</p>
<p class="font-stretch-normal">Default width.</p>
<p class="font-stretch-expanded">Wider letters for headers.</p>

<!-- Percentage form -->
<h1 class="font-stretch-75%">Three-quarters width.</h1>
<h1 class="font-stretch-115%">Slightly expanded.</h1>

<!-- Arbitrary -->
<h1 class="font-stretch-[66.66%]">Exact precision.</h1>
```

Only renders the difference when the font is a variable font with a width axis (e.g., Inter Variable, Recursive, Roboto Flex). NEVER expect `font-stretch-*` to do anything on a single-width font.

### Pattern 6 : Inset shadow stacked with outer shadow

```html
<button class="
  bg-slate-100 px-4 py-2 rounded-md
  shadow-md
  inset-shadow-xs inset-shadow-white/50
">
  Beveled button
</button>
```

The button has an outer drop shadow AND a subtle inner highlight. The two compose at the `box-shadow` level; v4 keeps them in separate CSS variables (`--tw-shadow`, `--tw-inset-shadow`) and composites in the final rule.

### Pattern 7 : Multi-layer shadow + ring composition

```html
<div class="
  bg-white rounded-lg p-6
  shadow-lg
  inset-shadow-sm
  ring-1 ring-slate-200
  inset-ring-1 inset-ring-white/80
">
  Four-layer composition : outer shadow, inset shadow, outer ring, inset ring.
</div>
```

ALL four `box-shadow` layers compose simultaneously. v3 required hand-written multi-layer shadows; v4 ships them as composable utilities.

### Pattern 8 : `scheme-*` plus theme-token swap

```css
@import "tailwindcss";
@custom-variant dark (&:where(.dark, .dark *));

@theme {
  --color-background: oklch(1 0 0);
}

.dark {
  --color-background: oklch(0.15 0.02 250);
}
```

```html
<html class="dark scheme-dark">
  <body class="bg-background">
    <input type="date" />
  </body>
</html>
```

The `scheme-dark` on `<html>` propagates to all native controls inside, no per-control class needed.

### Pattern 9 : Inset ring as focus indicator

```html
<button class="
  bg-blue-600 text-white rounded-md px-4 py-2
  focus-visible:inset-ring-2 focus-visible:inset-ring-blue-300
  focus:outline-none
">
  Click me
</button>
```

Inset ring renders INSIDE the element bounds. Useful when an outer ring would clip against neighbouring elements or affect layout. ALWAYS prefer outer `ring-*` for general focus indicators (clearer affordance); use `inset-ring-*` when geometry constraints demand.

### Pattern 10 : Combining `starting:` + variant stacking

```html
<dialog
  id="modal"
  class="
    bg-white rounded-lg shadow-xl p-6
    opacity-100 scale-100
    starting:open:opacity-0 starting:open:scale-95
    transition-all transition-discrete duration-300 ease-out
    backdrop:bg-black/50
    starting:open:backdrop:bg-black/0
  "
>
  <p>Dialog content with enter animation on open.</p>
  <form method="dialog">
    <button>Close</button>
  </form>
</dialog>
```

The backdrop also animates from transparent to 50% black on open. The variant stacks read : `starting:open:opacity-0` means "AT the starting style, when the element is `:open`, set opacity to 0."

## Anti-patterns (summary; full list in references/anti-patterns.md)

NEVER do these :

1. NEVER omit `transition-discrete` when animating a `display: none` to `display: block` change. Without it, `display` snaps instantly to the final value and the `starting:` start-style never has time to render.
2. NEVER use `transition-discrete` on continuous properties (color, opacity, transform). It has no effect there; the existing `transition-*` already handles those.
3. NEVER set `color-scheme` via inline style or custom CSS when the v4 `scheme-*` utility exists. The utility participates in variants (`dark:scheme-dark`) and the typed scale; inline styles do not.
4. NEVER expect `font-stretch-*` to do anything on a non-variable-font. The font-family must support a width axis.
5. NEVER write `color-scheme-light` or `color-scheme-dark` ; these are NOT the v4 class names. The canonical form is `scheme-light`, `scheme-dark`, `scheme-light-dark`.
6. NEVER stack four `box-shadow` declarations manually in custom CSS. The v4 layered composition (`shadow-*` + `inset-shadow-*` + `ring-*` + `inset-ring-*`) handles this; manual stacking will override the v4 cascade.

## Reference Links

- [references/methods.md](references/methods.md) : full class list per family, CSS-spec mapping, browser-support baseline
- [references/examples.md](references/examples.md) : working popover, modal, accordion, auto-resize textarea, theme-aware native controls
- [references/anti-patterns.md](references/anti-patterns.md) : transition-discrete misuse, scheme-class-name confusion, font-stretch silent failure

## Cross-references

- [tailwind-core-architecture](../../tailwind-core/tailwind-core-architecture/SKILL.md) : utility-first principles that govern when to add custom CSS vs use utilities
- [tailwind-core-v3-vs-v4](../../tailwind-core/tailwind-core-v3-vs-v4/SKILL.md) : the engine and browser-baseline shift that enables these modern utilities
- [tailwind-syntax-utility-classes](../tailwind-syntax-utility-classes/SKILL.md) : base utility families including `shadow-*`, `ring-*` that compose with these inset variants
- [tailwind-syntax-functional-utilities](../tailwind-syntax-functional-utilities/SKILL.md) : `@utility` + `--value()` for authoring custom variants of these patterns
- [tailwind-syntax-variants](../tailwind-syntax-variants/SKILL.md) : the `starting:`, `open:`, `dark:` variant prefixes used in this skill

## Sources

- https://tailwindcss.com/docs/transition-behavior (`transition-discrete`, `transition-normal`)
- https://tailwindcss.com/docs/field-sizing (`field-sizing-content`, `field-sizing-fixed`)
- https://tailwindcss.com/docs/color-scheme (`scheme-*` family)
- https://tailwindcss.com/docs/font-stretch (`font-stretch-*` family)
- https://tailwindcss.com/docs/box-shadow (`inset-shadow-*`, `inset-ring-*` layering)
- https://tailwindcss.com/blog/tailwindcss-v4 (modern utility motivation)

Verified 2026-05-19.
