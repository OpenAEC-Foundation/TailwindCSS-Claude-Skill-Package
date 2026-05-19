# State Modifiers : Anti-Patterns

Each entry pairs a real failure with the verified root cause and fix.
Sources : https://tailwindcss.com/docs/hover-focus-and-other-states,
https://v3.tailwindcss.com/docs/hover-focus-and-other-states.

## AP-1 : `before:` or `after:` Pseudo-Element Not Rendering

### Symptom
`<span class="before:ml-1 before:size-2 before:bg-red-500"></span>` produces
NO visible pseudo-element. DevTools shows the `::before` rule applied but
the element does not paint.

### Root cause
Both v3 and v4 rely on Preflight to inject the default `content: ''` for
`before:` and `after:` pseudo-elements. With Preflight disabled, neither
version adds the default, and the pseudo-element has no content, so it
collapses to a zero-size box that does not render.

### NEVER assume `before:` works automatically when Preflight is off.

### ALWAYS add explicit `content-['']`

```html
<span class="before:content-[''] before:ml-1 before:size-2 before:bg-red-500"></span>
```

Or re-enable Preflight :

- v3 : remove `corePlugins: { preflight: false }`.
- v4 : ensure `@import "tailwindcss"` (which includes preflight) rather
  than partial imports.

Source : https://v3.tailwindcss.com/docs/hover-focus-and-other-states
("If you've disabled preflight, the content property will not be set to
an empty string by default").

## AP-2 : `invalid:` Showing Red Border On Page Load

### Symptom
`<input type="email" required class="invalid:border-red-500" />` renders
with a red border the moment the page loads, before the user has typed
anything. Visually noisy on every required field.

### Root cause
`:invalid` matches from page load whenever the input does not satisfy
its constraints. An empty required field is `:invalid` immediately. The
correct selector for "invalid AFTER user interaction" is `:user-invalid`
(v4) or a workaround in v3.

### NEVER use `invalid:` for first-impression error styling.

### ALWAYS in v4 : use `user-invalid:`

```html
<input type="email" required
  class="user-invalid:border-red-500 user-invalid:text-red-700" />
```

### ALWAYS in v3 : use the placeholder trick

```html
<input type="email" required placeholder=" "
  class="invalid:not-placeholder-shown:border-red-500" />
```

The placeholder is a single space ; the input stops being
`:placeholder-shown` as soon as the user types. The variant chain fires
only after interaction.

## AP-3 : `first:` Matching the Wrong Element Type

### Symptom
```html
<div>
  <h2 class="first:text-xl">Heading</h2>
  <ul>
    <li class="first:pt-0">First item</li>
    <li>Second item</li>
  </ul>
</div>
```
The heading also gets `text-xl` even though intent was "first list item".

### Root cause
`first:` matches `:first-child`, counting ALL sibling types. The heading
is the first child of `<div>` regardless of being an `h2`.

### NEVER use `first:` on an element that shares a parent with different
tag types unless you mean "any first sibling".

### ALWAYS use `first-of-type:` when you mean "first of this tag"

```html
<div>
  <h2 class="first-of-type:text-xl">Heading</h2>
  <ul>
    <li class="first-of-type:pt-0">First li</li>
    <li>Second li</li>
  </ul>
</div>
```

Better : structure markup so each list lives in its own wrapper, removing
the ambiguity entirely.

## AP-4 : `placeholder:` On the Wrapper Instead of the Input

### Symptom
```html
<div class="placeholder:text-red-500">
  <input type="text" placeholder="Name" />
</div>
```
The placeholder text stays the default colour. The `placeholder:` variant
silently does nothing on the wrapper.

### Root cause
`placeholder:` compiles to `&::placeholder`. Pseudo-elements only exist on
the element that owns them. The `<div>` has no `::placeholder`.

### NEVER put `placeholder:` on a wrapper element.

### ALWAYS put `placeholder:` on the input itself

```html
<div>
  <input type="text" placeholder="Name"
    class="placeholder:text-red-500" />
</div>
```

The same rule applies to `file:`, `marker:`, `selection:`, `first-letter:`,
`first-line:`, and `backdrop:` : put the variant on the element that has
the pseudo-element, not an ancestor.

## AP-5 : `motion-safe:animate-spin` Without `motion-reduce:` Fallback

### Symptom
A loading spinner uses `animate-spin` gated only by `motion-safe:`.
Users with `prefers-reduced-motion: reduce` see... nothing. The whole
loading indicator disappears.

### Root cause
`motion-safe:animate-spin` adds the spin animation ONLY when the user
prefers full motion. With reduced motion, the class produces no animation
AND no replacement static styling. The spinner becomes invisible.

### NEVER ship animation gated only by `motion-safe:`.

### ALWAYS provide a `motion-reduce:` static fallback

```html
<svg class="
    motion-safe:animate-spin
    motion-reduce:opacity-50
    motion-reduce:hidden
  ">
  <!-- ... -->
</svg>
<span class="motion-reduce:inline hidden">Loading...</span>
```

This either hides the spinner and shows a text label, or shows a static
dimmed icon, depending on the design.

## AP-6 : `open:` Not Matching A `<dialog>` Opened Via `.show()`

### Symptom
v3 project uses `<dialog>` opened via `.show()` (modal-less). The
`open:` variant does NOT apply.

### Root cause
v3's `open:` compiles to `&[open]`. Calling `dialog.show()` or
`dialog.showModal()` DOES set the `[open]` attribute, so the selector
matches. If the variant still does not fire, the dialog is likely being
toggled via `style.display` instead of the native API.

### ALWAYS toggle `<dialog>` via the native API

```js
dialog.show()         // non-modal, [open] attribute set
dialog.showModal()    // modal with backdrop, [open] set
dialog.close()        // [open] removed
```

NEVER style-hide a dialog with `display: none` and expect `open:` to react.
The variant tracks the attribute, not visibility.

## AP-7 : Stacking Order Breakage After v3 to v4 Upgrade

### Symptom
After upgrading to v4, `first:*:pt-0` no longer matches the first child
of any direct child. Style is missing.

### Root cause
v4 reads variant chains left-to-right. v3 read them right-to-left. The
upgrade tool rewrites common cases, but bespoke chains may slip through.

### ALWAYS flip the order on chains of 2+ variants involving structural
selectors

```html
<!-- v3 -->
<ul class="first:*:pt-0">

<!-- v4 -->
<ul class="*:first:pt-0">
```

See `tailwind-syntax-variants` and `tailwind-core-v3-vs-v4` for the full
migration semantics.

## AP-8 : Whitespace Inside `content-[...]`

### Symptom
`content-['Hello World']` produces a CSS error or the literal
`content: 'Hello_World'` (with an underscore visible). The space is
mangled.

### Root cause
Tailwind treats whitespace inside arbitrary-value brackets as an
underscore separator. To include an actual space, use the underscore
literal (which Tailwind converts to a space at output) or escape it.

### NEVER
```html
<span class="before:content-['Hello World']"></span>
```

### ALWAYS
```html
<span class="before:content-['Hello_World']"></span>
<!-- OR with explicit double-quote and escape -->
<span class="before:content-['Hello\_World']"></span>
```

## AP-9 : `empty:` Matching A Whitespace-Only Element

### Symptom
`<ul class="empty:hidden"> </ul>` does NOT hide. The list still renders.

### Root cause
`:empty` matches when there is NO content : no children AND no text. A
single space counts as text content.

### NEVER rely on `empty:` for visually-empty containers that may contain
whitespace.

### ALWAYS render an empty parent with no whitespace

```jsx
{items.length === 0 ? <ul></ul> : <ul>{items.map(...)}</ul>}
```

Or use a JavaScript check on `items.length` to conditionally render the
parent.

## AP-10 : `target:` On An Element Without `id`

### Symptom
`<section class="target:bg-yellow-100">...</section>` never highlights.

### Root cause
`:target` matches an element whose `id` equals the URL fragment. Without
an `id`, the selector can never match.

### ALWAYS add an `id` matching the fragment

```html
<section id="contact" class="target:bg-yellow-100">...</section>
```

## AP-11 : Mixing `valid:` And `user-valid:` On The Same Field

### Symptom
A field uses both `invalid:border-red-500` AND `user-valid:border-green-500`.
On page load the field is red. After typing a valid value, it stays red
because both variants now apply with similar specificity.

### Root cause
`:invalid` and `:user-invalid` are independent. Combining `invalid:` styles
that fire on page load with `user-valid:` styles that fire after interaction
produces visual conflicts.

### NEVER mix the eager and lazy variants on the same field.

### ALWAYS pick one model

v4 (preferred) :
```html
<input required
  class="user-invalid:border-red-500 user-valid:border-green-500" />
```

v3 (placeholder trick) :
```html
<input required placeholder=" "
  class="
    invalid:not-placeholder-shown:border-red-500
    valid:not-placeholder-shown:border-green-500
  " />
```

## AP-12 : `read-only:` Not Matching An `aria-readonly` Element

### Symptom
`<div role="textbox" aria-readonly="true" class="read-only:bg-gray-100">`
does NOT pick up the gray background.

### Root cause
`read-only:` compiles to `&:read-only`, which only matches form elements
with the `readonly` attribute (`<input readonly>`, `<textarea readonly>`).
ARIA's `aria-readonly` is NOT recognised by the `:read-only` pseudo-class.

### NEVER use `read-only:` on non-form elements.

### ALWAYS use the aria-attribute variant

```html
<div role="textbox" aria-readonly="true"
  class="aria-readonly:bg-gray-100">...</div>
```

See `tailwind-syntax-variants` for the full aria-* and data-* variant
grammar.

## AP-13 : `default:` Not Matching A Programmatically-Selected Radio

### Symptom
JavaScript sets `radio.checked = true` on page load. The `default:` variant
does NOT fire.

### Root cause
`:default` matches the radio button or option that was the default at
PAGE PARSE TIME (the one with the `checked` attribute in HTML). Setting
`.checked` via JS does NOT update the `:default` pseudo-class.

### NEVER expect `default:` to follow live state.

### ALWAYS use `:checked` for current state

```html
<input type="radio" name="color" checked
  class="default:ring-2 default:ring-blue-500 checked:bg-blue-500" />
```

`default:` styles the initial selection. `checked:` styles the currently-checked one. They diverge as soon as the user clicks a different option.

## AP-14 : `backdrop:` On A Non-`<dialog>` Element

### Symptom
`<div class="backdrop:bg-black/30">...</div>` produces no visible backdrop.

### Root cause
`::backdrop` is a pseudo-element that ONLY exists on `<dialog>` and (in
v4) elements showing in the popover top-layer. A regular `<div>` has no
backdrop pseudo-element.

### NEVER apply `backdrop:` to ordinary block-level elements.

### ALWAYS use `backdrop:` on `<dialog>` or `[popover]`

```html
<dialog class="backdrop:bg-black/40">
  <p>Content</p>
</dialog>
```

For non-native modal overlays, use a sibling overlay div with absolute
positioning and `bg-black/40` instead of relying on `backdrop:`.

## AP-15 : Forgetting That `nth-[odd]:` And `odd:` Are Identical

### Symptom
Code uses `nth-[odd]:bg-gray-50` alongside `odd:bg-gray-50`. Duplicate
classes generated.

### Root cause
`odd:` is sugar for `:nth-child(odd)` ; so is `nth-[odd]:`. They produce
the same CSS.

### ALWAYS prefer the shorter sugar

```html
<li class="odd:bg-gray-50 even:bg-white">...</li>
```

Reserve `nth-[...]` for non-odd/even patterns like `nth-[3n+1]:`,
`nth-[5]:`, or `nth-[2n+1_of_li]:`.
