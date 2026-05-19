# State Modifiers : Complete API Reference

Verified against https://tailwindcss.com/docs/hover-focus-and-other-states
(v4) and https://v3.tailwindcss.com/docs/hover-focus-and-other-states (v3),
2026-05-19.

## 1. Structural Pseudo-Class Variants

| Variant | CSS selector | v3 | v4 | Notes |
|---------|--------------|----|----|-------|
| `first` | `&:first-child` | Yes | Yes | First sibling of ANY tag |
| `last` | `&:last-child` | Yes | Yes | Last sibling of ANY tag |
| `only` | `&:only-child` | Yes | Yes | Only child of parent |
| `odd` | `&:nth-child(odd)` | Yes | Yes | 1st, 3rd, 5th, ... |
| `even` | `&:nth-child(even)` | Yes | Yes | 2nd, 4th, 6th, ... |
| `first-of-type` | `&:first-of-type` | Yes | Yes | First sibling of THIS tag |
| `last-of-type` | `&:last-of-type` | Yes | Yes | Last sibling of THIS tag |
| `only-of-type` | `&:only-of-type` | Yes | Yes | Only sibling of THIS tag |
| `empty` | `&:empty` | Yes | Yes | No children AND no text |

### Arbitrary nth- variants (v4)

| Variant | Selector |
|---------|----------|
| `nth-[3]` | `&:nth-child(3)` |
| `nth-[3n+1]` | `&:nth-child(3n+1)` |
| `nth-last-[2]` | `&:nth-last-child(2)` |
| `nth-of-type-[2]` | `&:nth-of-type(2)` |
| `nth-of-type-[odd]` | `&:nth-of-type(odd)` |
| `nth-last-of-type-[1]` | `&:nth-last-of-type(1)` |
| `nth-[2n+1_of_li]` | `&:nth-child(2n+1 of li)` (CSS Selectors 4) |

v3 supports bracket-arbitrary variants via `[&:nth-child(3)]:`. v4
parameterised forms (`nth-[3]:`) are syntactic sugar that resolves to the
same selector.

## 2. Pseudo-Element Variants

| Variant | Selector | Browser support note |
|---------|----------|----------------------|
| `before` | `&::before` | All browsers |
| `after` | `&::after` | All browsers |
| `placeholder` | `&::placeholder` | All browsers (modern syntax since Chrome 57+) |
| `file` | `&::file-selector-button` | Safari 13.1+, Chrome 89+, Firefox 82+ |
| `marker` | `&::marker, & *::marker` | Safari 11.1+, Chrome 86+, Firefox 68+ |
| `selection` | `&::selection` | All browsers |
| `first-letter` | `&::first-letter` | All browsers |
| `first-line` | `&::first-line` | All browsers |
| `backdrop` | `&::backdrop` | Safari 15.4+, Chrome 37+, Firefox 47+ (for `<dialog>`) |

### Content Utility (v3 and v4)

```html
<span class="before:content-['*'] before:text-red-500"></span>
<span class="after:content-['→']"></span>
<span data-x="New" class="after:content-[attr(data-x)]"></span>
<span class="before:content-['']"></span>
```

Both v3 and v4 auto-insert `content: ''` for `before:` and `after:` when
Preflight (or `@import "tailwindcss"` in v4 which includes preflight) is
active. With Preflight disabled, both versions require explicit `content-['']`.

Underscores inside the brackets convert to spaces : `content-['Hello_World']`
renders as "Hello World".

## 3. Motion-Preference Variants

| Variant | Media query | v3 | v4 |
|---------|-------------|----|----|
| `motion-safe` | `@media (prefers-reduced-motion: no-preference)` | Yes | Yes |
| `motion-reduce` | `@media (prefers-reduced-motion: reduce)` | Yes | Yes |

## 4. Form Input State Variants

| Variant | Selector | v3 | v4 | Notes |
|---------|----------|----|----|-------|
| `required` | `&:required` | Yes | Yes | `<input required>` |
| `optional` | `&:optional` | Yes | Yes | `<input>` (no `required`) |
| `valid` | `&:valid` | Yes | Yes | HTML5 valid (fires before user input) |
| `invalid` | `&:invalid` | Yes | Yes | HTML5 invalid (fires before user input) |
| `user-valid` | `&:user-valid` | No | Yes | Only after user interaction |
| `user-invalid` | `&:user-invalid` | No | Yes | Only after user interaction |
| `disabled` | `&:disabled` | Yes | Yes | `<input disabled>` |
| `enabled` | `&:enabled` | Yes | Yes | `<input>` not disabled |
| `checked` | `&:checked` | Yes | Yes | Checkbox / radio checked |
| `indeterminate` | `&:indeterminate` | Yes | Yes | Checkbox `indeterminate=true` |
| `default` | `&:default` | Yes | Yes | Browser-pre-selected radio / form submit |
| `in-range` | `&:in-range` | Yes | Yes | `<input type="number">` within min/max |
| `out-of-range` | `&:out-of-range` | Yes | Yes | Number/range outside min/max |
| `placeholder-shown` | `&:placeholder-shown` | Yes | Yes | Empty input with placeholder visible |
| `autofill` | `&:autofill` | Yes | Yes | Browser-autofilled (Webkit `:-webkit-autofill`, modern `:autofill`) |
| `read-only` | `&:read-only` | Yes | Yes | `<input readonly>` |

### v4-Only : `user-valid` / `user-invalid`

Source : https://developer.mozilla.org/en-US/docs/Web/CSS/:user-invalid.
The `:user-valid` and `:user-invalid` pseudo-classes match only after the
user has interacted with the field (typed, blurred, or submitted). `:valid`
and `:invalid` match from page load.

ALWAYS prefer `user-invalid:` in v4 for error styling. In v3, use the
placeholder trick :

```html
<input required placeholder=" "
  class="invalid:not-placeholder-shown:border-red-500" />
```

This requires the input to have a non-empty placeholder ; the variant
chain (`invalid` AND `not-placeholder-shown`) triggers only once the user
types or the placeholder is empty.

## 5. Target Variant

| Variant | Selector | v3 | v4 |
|---------|----------|----|----|
| `target` | `&:target` | Yes | Yes |

Matches when the element's id equals the document's URL fragment.

```html
<section id="contact" class="target:bg-yellow-100">Contact</section>
```

Browsing to `/about#contact` highlights the section.

## 6. Open Variant

| Version | Selector | Matches |
|---------|----------|---------|
| v3 | `&[open]` | `<details open>`, `<dialog open>` |
| v4 | `&:is([open], :popover-open, :open)` | All of the above plus `[popover]` shown |

```html
<details class="open:bg-gray-50">...</details>
<dialog class="open:p-6">...</dialog>
<div popover class="open:shadow-lg">...</div>   <!-- v4 only -->
```

The popover API requires browser support (Safari 17+, Chrome 114+,
Firefox 125+). v3's `open:` does not match it.

## 7. Empty Variant

```html
<ul class="empty:hidden">
  <!-- whole ul hidden when there are no <li> children AND no text content -->
</ul>
```

Whitespace counts as text, so a `<ul> </ul>` is NOT empty for the purpose
of `:empty`.

## 8. Stacking Order (v3 vs v4)

v3 reads variant chains right-to-left ; v4 reads left-to-right. Most state
variants are LEAF variants (no nested intent), so the order rarely matters.
The exception is structural + child-selector combinations :

```html
<!-- v3 : "first child of any direct child" -->
<ul class="first:*:pt-0"></ul>

<!-- v4 : "any direct child, first one" (same intent) -->
<ul class="*:first:pt-0"></ul>
```

See `tailwind-syntax-variants` for the full migration rules. The upgrade
codemod handles common cases automatically.

## 9. Customisation

State variants are NOT individually configurable in either v3 or v4. They
ship as core variants. To add a custom variant :

### v3 : plugin with `addVariant()`

```js
const plugin = require('tailwindcss/plugin')
module.exports = plugin(function ({ addVariant }) {
  addVariant('user-invalid', '&:user-invalid')
})
```

### v4 : `@custom-variant`

```css
@import "tailwindcss";
@custom-variant my-valid (&:user-valid);
```

## 10. Browser-Support Caveats

| Variant | Min Safari | Min Chrome | Min Firefox |
|---------|-----------|-----------|------------|
| `user-valid` / `user-invalid` | 16.5 | 119 | 88 |
| `[popover]` matching in `open` | 17 | 114 | 125 |
| `file` (file-selector-button) | 13.1 | 89 | 82 |
| `marker` | 11.1 | 86 | 68 |
| `backdrop` (for `<dialog>`) | 15.4 | 37 | 47 |
| `:autofill` standard syntax | 14.5 | 88 | 79 |

v4 baseline (Safari 16.4+, Chrome 111+, Firefox 128+) covers all of these
except popover and user-valid which require even newer browsers in some
cases. Verify support if targeting Safari 16.4 strictly.
