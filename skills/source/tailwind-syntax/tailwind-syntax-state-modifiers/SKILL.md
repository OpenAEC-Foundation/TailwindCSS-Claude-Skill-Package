---
name: tailwind-syntax-state-modifiers
description: >
  Use when styling element states (form input validity, checked, disabled,
  required, placeholder visibility), structural positions (first, last,
  even, odd, nth-child), pseudo-elements (before, after, placeholder, file,
  marker, selection, first-letter), or motion-preference (motion-safe,
  motion-reduce), or when wiring a Tailwind class to a CSS pseudo-element
  with the content utility. Prevents the empty-before-pseudo trap (the
  ::before disappears when Preflight is disabled), the structural-variant
  scope mistake (first: applied to grandchildren when it should be on
  direct children), the placeholder-styling mistake (placeholder: on the
  wrapper instead of the input), the form-state target mistake (invalid:
  triggers before user input via :user-invalid in v4), and the dialog/details
  open-state confusion (open: works for both via the :is() compound).
  Covers structural pseudo-class variants, all pseudo-element variants, the
  content-[...] utility for ::before and ::after, motion-preference media
  query variants, the full form-state variant set, and target/open variants.
  Keywords: tailwind state, first-child, last-child, nth-child, odd, even,
  before, after, placeholder, content-[], content utility, motion-safe,
  motion-reduce, prefers-reduced-motion, required, valid, invalid,
  user-valid, user-invalid, disabled, checked, indeterminate, autofill,
  read-only, placeholder-shown, in-range, out-of-range, target, open,
  details open, dialog open, empty, first-letter, first-line, marker,
  selection, backdrop, file-selector-button, my ::before is not showing,
  content empty string, preflight disabled, why is first: not working, how
  do I style placeholder text, how do I style disabled input, how to style
  checked checkbox, pseudo element variant, structural variant.
license: MIT
compatibility: "Designed for Claude Code. Requires Tailwind CSS v3.4 or v4.0+."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# Tailwind CSS State Modifiers

This skill covers variants that target CSS states and pseudo-elements other
than interaction states (`hover`, `focus`, `active`). Use it together with :

- `tailwind-syntax-variants` : full variant grammar including hover/focus,
  group/peer, attribute, and arbitrary variants
- `tailwind-syntax-responsive` : breakpoint and container-query variants
- `tailwind-syntax-dark-mode` : the `dark:` variant configuration

## Quick Reference : Variant Categories

| Category | Examples |
|----------|----------|
| Structural position | `first:`, `last:`, `only:`, `odd:`, `even:`, `first-of-type:`, `last-of-type:`, `only-of-type:`, `nth-[...]:`, `empty:` |
| Pseudo-elements | `before:`, `after:`, `placeholder:`, `file:`, `marker:`, `selection:`, `first-letter:`, `first-line:`, `backdrop:` |
| Motion preference | `motion-safe:`, `motion-reduce:` |
| Form input state | `required:`, `optional:`, `valid:`, `invalid:`, `user-valid:`, `user-invalid:`, `disabled:`, `enabled:`, `checked:`, `indeterminate:`, `default:`, `in-range:`, `out-of-range:`, `placeholder-shown:`, `autofill:`, `read-only:` |
| Target / Open | `target:`, `open:` |

All variants in this skill are SHARED between v3 and v4 unless flagged
otherwise. The one divergence : `user-valid:` and `user-invalid:` are
v4-only (CSS-native, gated on browser support in v3).

## Structural Variants : Position in Parent

```html
<ul>
  <li class="first:pt-0">A</li>     <!-- &:first-child -->
  <li>B</li>
  <li>C</li>
  <li class="last:pb-0">D</li>      <!-- &:last-child -->
</ul>

<ul>
  <li class="odd:bg-gray-50 even:bg-white">A</li>   <!-- &:nth-child(odd/even) -->
  <li class="odd:bg-gray-50 even:bg-white">B</li>
</ul>

<ul>
  <li class="only:font-bold">Alone</li>             <!-- &:only-child -->
</ul>

<table>
  <tr class="first-of-type:font-bold last-of-type:border-b-0">
    <td>Row data</td>
  </tr>
</table>

<div class="empty:hidden"></div>                     <!-- &:empty (no children, no text) -->
```

### Arbitrary `nth-` Selectors

| Variant | Selector |
|---------|----------|
| `nth-[3n+1]:` | `&:nth-child(3n+1)` |
| `nth-last-[3]:` | `&:nth-last-child(3)` |
| `nth-of-type-[2n]:` | `&:nth-of-type(2n)` |
| `nth-last-of-type-[odd]:` | `&:nth-last-of-type(odd)` |
| `nth-[2n+1_of_li]:` | `&:nth-child(2n+1 of li)` |

NEVER apply structural variants to elements that share a parent with
DIFFERENT element types unless you want the variant to count ALL siblings.
`first-of-type:` counts only same-tag siblings ; `first:` counts every kind.

```html
<!-- "first:" picks the heading even though it's not a li -->
<div>
  <h2 class="first:text-xl">Heading</h2>
  <ul>
    <li class="first:pt-0">First li (correct)</li>
    <li>Second</li>
  </ul>
</div>
```

## Pseudo-Element Variants

| Variant | Selector | Use case |
|---------|----------|----------|
| `before:` | `&::before` | Inject content before element |
| `after:` | `&::after` | Inject content after element |
| `placeholder:` | `&::placeholder` | Style `<input>` placeholder text |
| `file:` | `&::file-selector-button` | Style `<input type="file">` button |
| `marker:` | `&::marker, & *::marker` | Style `<li>` bullets and ordered numbers |
| `selection:` | `&::selection` | Style user-selected text |
| `first-letter:` | `&::first-letter` | Drop-cap |
| `first-line:` | `&::first-line` | Style first line of text |
| `backdrop:` | `&::backdrop` | Style native `<dialog>` overlay |

## The Content Utility (before / after)

Both v3 and v4 automatically insert `content: ''` for `before:` and `after:`
WHEN PREFLIGHT IS ACTIVE. If Preflight is disabled, both versions fail
silently : the pseudo-element does not render. This is the single most
common before/after bug.

```html
<!-- Works with preflight on (both v3 and v4) -->
<span class="before:ml-1 before:inline-block before:size-2 before:bg-red-500"></span>

<!-- Override the default with a string -->
<span class="before:content-['→'] before:mr-1">Next</span>

<!-- Read from data attribute -->
<span data-label="New" class="after:content-[attr(data-label)]"></span>

<!-- Empty content (also explicit) -->
<span class="before:content-['']"></span>
```

Source : https://tailwindcss.com/docs/hover-focus-and-other-states
("Tailwind will automatically add content: '' by default so you don't have
to specify it unless you want a different value").

NEVER use a literal space between brackets in `content-[...]`. Whitespace
is parsed as an underscore. Use `content-['Hello_World']` (Tailwind
converts `_` to space) or `content-["Hello World"]` with double quotes.

### Preflight-Disabled Workaround

When `preflight: false` (v3) or `@import "tailwindcss/utilities";` only
(v4 without the preflight import), you MUST add `content-['']` explicitly
to every `before:` / `after:` usage :

```html
<!-- Preflight off : explicit content required -->
<span class="before:content-[''] before:ml-1 before:inline-block before:size-2 before:bg-red-500"></span>
```

## Form State Variants

```html
<input class="required:border-red-500" required />
<input class="optional:border-gray-300" />

<!-- HTML5 validation : :valid / :invalid match BEFORE first interaction -->
<input type="email" required
  class="invalid:border-red-500 invalid:text-red-700" />

<!-- v4 only : user-* variants match only AFTER user has interacted -->
<input type="email" required
  class="user-invalid:border-red-500 user-invalid:text-red-700" />

<input type="checkbox" class="checked:bg-blue-500" />
<input type="checkbox" class="indeterminate:bg-gray-400" />

<input type="radio" class="default:ring-2 default:ring-blue-500" />

<input type="text" disabled class="disabled:opacity-50 disabled:cursor-not-allowed" />
<input type="text" readonly class="read-only:bg-gray-50" />

<input class="placeholder-shown:border-gray-300" placeholder="Type here" />

<input type="text" class="autofill:bg-yellow-50" />

<input type="number" min="0" max="100"
  class="in-range:border-green-500 out-of-range:border-red-500" />
```

### Variant Selectors

| Variant | Selector |
|---------|----------|
| `required` | `&:required` |
| `optional` | `&:optional` |
| `valid` | `&:valid` (matches before first interaction) |
| `invalid` | `&:invalid` (matches before first interaction) |
| `user-valid` | `&:user-valid` (v4 only ; matches after user input) |
| `user-invalid` | `&:user-invalid` (v4 only) |
| `disabled` | `&:disabled` |
| `enabled` | `&:enabled` |
| `checked` | `&:checked` |
| `indeterminate` | `&:indeterminate` |
| `default` | `&:default` (form input pre-selected by browser) |
| `in-range` | `&:in-range` (number/range inputs) |
| `out-of-range` | `&:out-of-range` |
| `placeholder-shown` | `&:placeholder-shown` |
| `autofill` | `&:autofill` |
| `read-only` | `&:read-only` |

ALWAYS prefer `user-invalid:` over `invalid:` in v4 when styling field
errors : `invalid:` fires on every required field as soon as the page
loads, before the user has typed anything. `user-invalid:` waits for
interaction. v3 has no `user-*` variants : work around with the
`:placeholder-shown` trick or wire validation via JS.

```html
<!-- v3 trick : invalid only matters after user typed something -->
<input type="email" required
  class="invalid:not-placeholder-shown:border-red-500"
  placeholder=" " />
```

## Motion Preference Variants

```html
<!-- Animation only when user has NOT set reduce-motion preference -->
<div class="motion-safe:animate-spin">spinner</div>

<!-- Static fallback when user prefers reduced motion -->
<div class="motion-safe:animate-bounce motion-reduce:opacity-50"></div>
```

| Variant | Media query |
|---------|-------------|
| `motion-safe` | `@media (prefers-reduced-motion: no-preference)` |
| `motion-reduce` | `@media (prefers-reduced-motion: reduce)` |

ALWAYS gate animations with `motion-safe:` OR provide a `motion-reduce:`
override. NEVER ship `animate-spin` unconditionally on critical UI : users
with vestibular conditions can be physically hurt by it.

## Target Variant : URL Fragment

```html
<a href="#section-2">Jump</a>

<section id="section-2" class="target:bg-yellow-100">
  Section 2 (highlights when URL fragment is #section-2).
</section>
```

`target:` matches when the element's id equals the current URL fragment.
Useful for in-page anchor highlighting without JS.

## Open Variant : details / dialog / popover

```html
<details class="open:bg-gray-50">
  <summary>Click to expand</summary>
  <p>Hidden until summary is clicked.</p>
</details>

<dialog id="d" class="backdrop:bg-black/30 open:p-4">
  <p>Modal content</p>
</dialog>

<div popover class="open:shadow-lg">popover content</div>
```

The v4 `open:` variant compiles to `&:is([open], :popover-open, :open)` so
it matches `<details>`, `<dialog>`, and `[popover]` elements simultaneously.
The v3 `open:` compiles to `&[open]` only and does NOT match popovers.

NEVER assume `open:` matches modals you open via `dialog.showModal()` :
that adds the `[open]` attribute, which IS matched. Programmatically
showing a `<dialog>` via `.show()` also sets `[open]`.

## Empty Variant

```html
<div class="empty:hidden">
  <!-- if this div has zero children AND zero text, it is display:none -->
</div>
```

`empty:` matches both no-child and no-text states. Whitespace counts as
text, so a `<div> </div>` is NOT empty. Use `empty:hidden` to collapse
list placeholders that may receive no items.

## Decision Tree : Which Variant Group?

```
Styling based on what?
├── Position among siblings → structural (first/last/nth-)
├── A pseudo-element (::before, ::placeholder, ::marker, etc.) → pseudo-element variants
├── A form input attribute state (required, disabled, checked) → form state
├── HTML5 validity (valid, invalid) → form-state, prefer user-valid/user-invalid in v4
├── User accessibility preference (reduced motion) → motion-safe/motion-reduce
├── URL fragment match → target:
├── details/dialog/popover open → open:
├── No children → empty:
└── User pointer interaction (hover, focus) → see tailwind-syntax-variants
```

## Stacking With Other Variants

```html
<!-- Position + hover -->
<li class="first:hover:bg-gray-100"></li>

<!-- Position + responsive -->
<li class="first:pt-0 md:first:pt-4"></li>

<!-- Form state + dark -->
<input class="invalid:border-red-500 dark:invalid:border-red-300" />

<!-- Pseudo-element + group -->
<div class="group">
  <span class="before:opacity-0 group-hover:before:opacity-100">→</span>
</div>
```

v4 stacks chains left-to-right (per upgrade guide). The functional intent
is the same as v3 ; only the syntax order differs. See `tailwind-syntax-variants`
for the stacking-order migration.

## Reference Files

- `references/methods.md` : every variant with full CSS selector and
  browser-support note
- `references/examples.md` : full real-world patterns (validated form,
  drop-cap, custom bullet, modal backdrop, accessible spinner)
- `references/anti-patterns.md` : the empty-before-with-preflight-off
  trap, `invalid:` firing before user input, `placeholder:` on wrapper
  instead of input, `first:` counting wrong sibling type

## Verified Sources

- https://tailwindcss.com/docs/hover-focus-and-other-states (v4)
- https://v3.tailwindcss.com/docs/hover-focus-and-other-states (v3)
- vooronderzoek-tailwind.md §6 Variant System
