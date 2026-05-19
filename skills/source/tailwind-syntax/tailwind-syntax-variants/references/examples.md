# Tailwind CSS Variants: Working Examples

Every example is verified against the v4 docs (tailwindcss.com/docs/hover-focus-and-other-states) and v3 docs (v3.tailwindcss.com/docs/hover-focus-and-other-states), 2026-05-19.

## Example 1: Stacking pseudo-class + breakpoint + dark mode

Works identically in v3 and v4 (stacking order applies to STRUCTURAL variants, not these).

```html
<button class="
  bg-blue-600 text-white px-4 py-2 rounded-md
  hover:bg-blue-700
  focus-visible:outline-2 focus-visible:outline-blue-400
  dark:bg-blue-500
  dark:hover:bg-blue-400
  md:px-6 md:py-3
  motion-reduce:transition-none
">
  Sign in
</button>
```

## Example 2: Named groups for nested hover hierarchies

```html
<article class="group/card rounded-lg border p-4 hover:shadow-lg">
  <h3 class="text-slate-900 group-hover/card:text-blue-600">Card title</h3>
  <p class="text-slate-600">Hovering anywhere on the card recolors the title.</p>

  <button class="group/edit invisible mt-2 group-hover/card:visible">
    <svg class="size-4 group-hover/edit:rotate-12 transition-transform">...</svg>
    <span class="text-sm group-hover/edit:underline">Edit</span>
  </button>
</article>
```

Hovering the card recolors the title and reveals the edit button. Hovering the button (a separate group) rotates the icon and underlines the label.

## Example 3: Peer state for sibling-driven styling

```html
<div class="flex flex-col">
  <input
    type="email"
    required
    placeholder="you@example.com"
    class="peer rounded border px-3 py-2 invalid:border-red-500"
  />
  <p class="hidden peer-invalid:block text-sm text-red-600 mt-1">
    Enter a valid email address.
  </p>
</div>
```

The `<p>` appears only when the previous sibling (`peer`) is invalid.

## Example 4: ARIA + data attribute variants (Radix UI pattern)

```html
<!-- Built-in aria-* shortcuts -->
<button
  aria-pressed="true"
  class="rounded-md border px-3 py-1.5
         aria-pressed:bg-slate-900
         aria-pressed:text-white
         aria-disabled:opacity-50"
>
  Bold
</button>

<!-- Arbitrary aria-[...] -->
<th aria-sort="ascending" class="aria-[sort=ascending]:bg-blue-50">
  Name
</th>

<!-- data-state pattern (every Radix UI controlled component) -->
<button
  data-state="open"
  class="transition-transform
         data-[state=open]:rotate-180
         data-[state=closed]:rotate-0"
>
  <svg class="size-4">...</svg>
</button>

<!-- data-side (Radix Popover, Tooltip, DropdownMenu) -->
<div
  data-side="bottom"
  class="data-[side=top]:slide-in-from-bottom-2
         data-[side=bottom]:slide-in-from-top-2
         data-[side=left]:slide-in-from-right-2
         data-[side=right]:slide-in-from-left-2"
>
  Tooltip
</div>
```

## Example 5: Has, Not, In selectors (v4)

```html
<!-- has-* : style the LABEL based on the child input state -->
<label class="
  flex items-center gap-2 rounded-md border p-3
  has-checked:bg-indigo-50
  has-checked:ring-2 has-checked:ring-indigo-500
">
  <input type="radio" name="plan" />
  Standard plan
</label>

<!-- not-* : negate any variant -->
<button class="
  bg-blue-600 text-white
  opacity-50
  not-hover:opacity-100
">
  Default 100%, hover 50% (inverted from typical)
</button>

<div class="
  grid-cols-3 grid
  not-supports-[display:grid]:flex
  not-supports-[display:grid]:flex-col
">
  Falls back to flex column when CSS grid is unsupported.
</div>

<!-- in-* : style a child based on an ancestor's attribute without `group` class -->
<div role="tabpanel">
  <button class="
    text-slate-500
    in-[role=tabpanel]:text-slate-900
    in-[role=tabpanel]:font-semibold
  ">
    Visible only inside a tabpanel
  </button>
</div>
```

## Example 6: Position-in-parent variants

```html
<!-- Direct children -->
<ul class="flex gap-2 *:rounded-full *:border *:px-3 *:py-1 *:text-sm">
  <li>Apple</li>
  <li>Banana</li>
  <li>Cherry</li>
</ul>

<!-- Deep descendants (v4 only) -->
<section class="**:data-avatar:size-12 **:data-avatar:rounded-full">
  <ul>
    <li><img data-avatar src="/a.png" /></li>
    <li><img data-avatar src="/b.png" /></li>
  </ul>
</section>

<!-- Structural pseudo-classes -->
<ul class="divide-y">
  <li class="first:pt-0 last:pb-0 py-2 odd:bg-white even:bg-slate-50">First</li>
  <li class="first:pt-0 last:pb-0 py-2 odd:bg-white even:bg-slate-50">Second</li>
  <li class="first:pt-0 last:pb-0 py-2 odd:bg-white even:bg-slate-50">Third</li>
</ul>

<!-- nth-* shortcuts (v4) -->
<table>
  <tr class="nth-3:bg-yellow-100"><td>Row</td></tr>
  <tr class="nth-[3n+1]:bg-blue-50"><td>Row</td></tr>
  <tr class="nth-last-2:font-bold"><td>Row</td></tr>
</table>
```

## Example 7: Stacking-order flip (v3 vs v4)

The single most common silent breakage during migration.

```html
<!-- v3 syntax (right-to-left) -->
<ul class="py-4 first:*:pt-0 last:*:pb-0">
  <li>One</li>
  <li>Two</li>
  <li>Three</li>
</ul>

<!-- v4 syntax (left-to-right) -->
<ul class="py-4 *:first:pt-0 *:last:pb-0">
  <li>One</li>
  <li>Two</li>
  <li>Three</li>
</ul>
```

Both compile to `ul > :first-child { padding-top: 0; }` but the WRITTEN order is reversed. The same rule applies to any stack involving `*:` or `**:` :

| v3 | v4 |
|----|----|
| `first:*:pt-0` | `*:first:pt-0` |
| `last:*:pb-0` | `*:last:pb-0` |
| `odd:*:bg-white` | `*:odd:bg-white` |
| `hover:[&_p]:underline` | `[&_p]:hover:underline` |

## Example 8: Important modifier (BREAKING CHANGE)

```html
<!-- v3 leading bang -->
<div class="!flex !bg-red-500 hover:!bg-red-600">v3</div>

<!-- v4 trailing bang -->
<div class="flex! bg-red-500! hover:bg-red-600!">v4</div>
```

Both produce `display: flex !important; background-color: ... !important; ...`. Migrate every `!` from leading to trailing.

## Example 9: Custom variants

### v3 (JavaScript plugin in `tailwind.config.js`)

```js
// tailwind.config.js
const plugin = require('tailwindcss/plugin')

module.exports = {
  content: ['./src/**/*.{html,jsx,tsx}'],
  plugins: [
    plugin(({ addVariant }) => {
      addVariant('third', '&:nth-child(3)')
      addVariant('hocus', ['&:hover', '&:focus'])
      addVariant('supports-grid', '@supports (display: grid)')
    }),
  ],
}
```

### v4 (CSS directive in main stylesheet)

```css
/* src/app.css */
@import "tailwindcss";

@custom-variant third (&:nth-child(3));
@custom-variant hocus (&:hover, &:focus);

@custom-variant supports-grid {
  @supports (display: grid) {
    @slot;
  }
}
```

### Usage (identical in both versions)

```html
<li class="third:underline">Third item underlined</li>
<button class="hocus:bg-blue-100">Highlight on hover or focus</button>
<div class="supports-grid:grid">Grid only when supported</div>
```

## Example 10: Restoring v3 hover behaviour in v4

If the v4 default of gating `hover:` on `@media (hover: hover)` breaks an existing design, restore the v3 behaviour with a `@custom-variant` :

```css
@import "tailwindcss";

@custom-variant hover (&:hover);
```

After this, `hover:bg-blue-700` applies on touch devices too (matching v3).

## Example 11: Arbitrary variants for one-off selectors

```html
<!-- Custom class selector -->
<div class="grid grid-cols-3 gap-4 [&.dragging]:cursor-grabbing">Items</div>

<!-- Descendant selector via _ -->
<article class="[&_p]:text-slate-600 [&_h2]:text-2xl [&_a]:text-blue-600 [&_a]:underline">
  <h2>Title</h2>
  <p>Body</p>
  <a href="#">Link</a>
</article>

<!-- Stacked arbitrary + built-in -->
<li class="[&.dragging]:active:cursor-grabbing">Drag-aware</li>

<!-- Custom at-rule -->
<div class="
  [@media(prefers-reduced-data:reduce)]:bg-none
  [@media(prefers-reduced-data:no-preference)]:bg-[url('/hero.jpg')]
">
  Hero
</div>
```
