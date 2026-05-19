# State Modifiers : Examples

Verified against https://tailwindcss.com/docs/hover-focus-and-other-states.
All examples work in both v3 and v4 unless flagged.

## 1. Striped Table Rows

```html
<table>
  <tbody>
    <tr class="odd:bg-white even:bg-gray-50">
      <td>Row 1</td>
    </tr>
    <tr class="odd:bg-white even:bg-gray-50">
      <td>Row 2</td>
    </tr>
  </tbody>
</table>
```

Or with `*:` to apply once to all children :

```html
<tbody class="*:odd:bg-white *:even:bg-gray-50">
  <tr><td>Row 1</td></tr>
  <tr><td>Row 2</td></tr>
</tbody>
```

`*:` here means "direct children". Combined with `odd:` and `even:`, the
parent declares the alternating pattern once.

## 2. Borderless First and Last List Items

```html
<ul class="divide-y">
  <li class="py-3 first:pt-0 last:pb-0">A</li>
  <li class="py-3 first:pt-0 last:pb-0">B</li>
  <li class="py-3 first:pt-0 last:pb-0">C</li>
</ul>
```

Removes vertical padding at the edges so the list visually hugs its
container.

## 3. Arbitrary nth Selector

```html
<ul>
  <li class="nth-[3n+1]:font-bold">A (3n+1 : 1, 4, 7, ...)</li>
  <li>B</li>
  <li>C</li>
  <li class="nth-[3n+1]:font-bold">D</li>
</ul>
```

Pre-v4 equivalent : `[&:nth-child(3n+1)]:font-bold`. Both forms work in v4.

## 4. Drop-Cap First Letter

```html
<p class="first-letter:text-5xl first-letter:font-bold first-letter:text-red-500 first-letter:mr-1 first-letter:float-left">
  Lorem ipsum dolor sit amet, consectetur adipiscing elit.
</p>
```

## 5. Styled Bullets

```html
<ul class="list-disc pl-6 marker:text-red-500 marker:text-xl">
  <li>One</li>
  <li>Two</li>
</ul>
```

For ordered lists :

```html
<ol class="list-decimal pl-6 marker:font-mono marker:text-blue-600">
  <li>First</li>
  <li>Second</li>
</ol>
```

## 6. Placeholder and Empty-Input Styling

```html
<input type="text" placeholder="your name"
  class="
    border px-3 py-2 rounded
    placeholder:text-gray-400 placeholder:italic
    placeholder-shown:bg-gray-50
  " />
```

`placeholder:` targets the placeholder text. `placeholder-shown:` targets
the INPUT when the placeholder is visible (i.e. empty input).

## 7. File Input Button

```html
<input type="file"
  class="
    file:mr-4 file:py-2 file:px-4 file:rounded
    file:border-0 file:bg-blue-50 file:text-blue-700
    file:cursor-pointer
    hover:file:bg-blue-100
  " />
```

Note the `hover:file:` stacking : hover on the input, target the
file-selector-button pseudo-element.

## 8. Selection Highlight

```html
<article class="selection:bg-yellow-200 selection:text-black">
  Select this text to see custom highlight colours.
</article>
```

## 9. Modal Backdrop and Open State

```html
<dialog id="confirm"
  class="
    rounded-lg p-6 shadow-lg
    backdrop:bg-black/40 backdrop:backdrop-blur-sm
    open:animate-in
  ">
  <p>Are you sure?</p>
  <form method="dialog">
    <button>Cancel</button>
    <button value="confirm">Confirm</button>
  </form>
</dialog>

<script>
  document.getElementById('confirm').showModal()
</script>
```

`backdrop:` styles the `::backdrop` pseudo-element. `open:` matches when
the dialog has the `[open]` attribute (set by `showModal()` or `show()`).

## 10. Details / Summary Disclosure

```html
<details class="border rounded p-3 open:bg-gray-50">
  <summary class="cursor-pointer font-medium">Read more</summary>
  <div class="mt-2 text-sm text-gray-700">
    Hidden content. Visible when the user clicks the summary.
  </div>
</details>
```

## 11. Anchor Target Highlight

```html
<a href="#contact">Jump to contact</a>

<section id="contact"
  class="
    transition-shadow
    target:shadow-lg target:ring-2 target:ring-blue-500
  ">
  <h2>Contact</h2>
  <p>Highlights when URL fragment is #contact.</p>
</section>
```

## 12. Validated Form (v4 with user-invalid)

```html
<form class="space-y-4">
  <label class="block">
    <span class="text-sm font-medium">Email</span>
    <input type="email" required
      class="
        mt-1 block w-full rounded border px-3 py-2
        focus:outline-none focus:ring-2 focus:ring-blue-500
        user-invalid:border-red-500 user-invalid:ring-red-300
      "
    />
    <span class="hidden text-sm text-red-600 user-invalid:[&]:block">
      Please enter a valid email address.
    </span>
  </label>

  <button type="submit"
    class="rounded bg-blue-500 px-4 py-2 text-white disabled:opacity-50">
    Submit
  </button>
</form>
```

`user-invalid:` fires only after the user has interacted with the field.
With plain `invalid:` the error border would show before the user types
anything.

## 13. Validated Form (v3 with placeholder trick)

```html
<input type="email" required placeholder=" "
  class="
    border px-3 py-2 rounded
    invalid:not-placeholder-shown:border-red-500
  " />
```

The placeholder is a single space ; the input is NOT placeholder-shown
as soon as the user types or pastes. Combined with `invalid:`, the red
border fires only after interaction.

## 14. Checkbox and Indeterminate

```html
<label class="flex items-center gap-2">
  <input type="checkbox"
    class="
      h-5 w-5 rounded border-gray-300
      checked:bg-blue-500 checked:border-blue-500
      indeterminate:bg-gray-400 indeterminate:border-gray-400
    "
  />
  <span>Toggle me</span>
</label>

<script>
  document.querySelector('input[type=checkbox]').indeterminate = true
</script>
```

## 15. Accessible Spinner

```html
<button type="button" disabled
  class="
    inline-flex items-center gap-2 rounded bg-blue-500 px-4 py-2 text-white
    disabled:opacity-50 disabled:cursor-not-allowed
  ">
  <svg viewBox="0 0 24 24"
    class="
      h-4 w-4
      motion-safe:animate-spin
      motion-reduce:hidden
    ">
    <!-- spinner geometry -->
  </svg>
  <span>Processing</span>
</button>
```

ALWAYS pair `motion-safe:animate-*` with a `motion-reduce:` fallback.
Spinners can cause harm to users with vestibular conditions ; hide them
or replace with static text when `prefers-reduced-motion: reduce`.

## 16. Empty Placeholder

```html
<ul id="todo-list" class="empty:hidden">
  <!-- JS appends <li> ; while empty the whole list is display:none -->
</ul>

<p class="hidden empty-list:block">No items yet.</p>
```

`empty:` matches when there are no children AND no text. Use it to
collapse list placeholders that may receive items later.

## 17. Range Input Validity

```html
<input type="number" min="0" max="100"
  class="
    border px-2 py-1 rounded
    in-range:border-green-500 in-range:bg-green-50
    out-of-range:border-red-500 out-of-range:bg-red-50
  " />
```

## 18. Read-Only Visual Cue

```html
<input type="text" readonly value="Cannot edit"
  class="
    border px-3 py-2 rounded
    read-only:bg-gray-100 read-only:text-gray-600
    read-only:cursor-not-allowed
  " />
```

## 19. Custom Content via Data Attribute

```html
<div data-counter="42" class="
    relative pr-8
    after:absolute after:right-0 after:top-0
    after:content-[attr(data-counter)]
    after:rounded-full after:bg-red-500 after:px-2 after:text-xs after:text-white
  ">
  Inbox
</div>
```

Combined with a data-* attribute that updates from JS, the badge content
stays in sync without re-rendering.

## 20. Preflight-Off Workaround

```html
<!-- when preflight is disabled, content-[''] MUST be explicit -->
<span class="
    before:content-['']
    before:ml-1 before:inline-block before:size-2 before:bg-red-500
  ">
  Status
</span>
```

Without `content-['']`, the `::before` pseudo-element has no content and
does not render in either v3 or v4 if preflight is off.
