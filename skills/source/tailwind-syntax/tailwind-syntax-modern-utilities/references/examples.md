# tailwind-syntax-modern-utilities : Examples

## CSS-Only Popover Enter Animation

```html
<button popovertarget="my-popover">Open</button>

<div
  popover
  id="my-popover"
  class="
    opacity-100 scale-100
    transition-all duration-200 transition-discrete
    starting:opacity-0 starting:scale-95
  "
>
  Hi from a CSS-only animated popover.
</div>
```

No JavaScript. The `starting:` variant sets the initial state, `transition-discrete` allows the discrete `display: none` to `display: block` jump to participate in the transition.

## `<dialog>` Enter Animation

```html
<dialog
  id="confirm-dialog"
  class="
    backdrop:bg-black/50 backdrop:opacity-100
    open:opacity-100 open:scale-100
    transition-all duration-300 transition-discrete
    starting:opacity-0 starting:scale-90
    starting:backdrop:opacity-0
  "
>
  <h2>Confirm</h2>
  <button>Yes</button>
  <button>No</button>
</dialog>

<script>
  document.getElementById("confirm-dialog").showModal();
</script>
```

## Auto-Resizing Textarea (CSS-only)

```html
<textarea
  class="
    field-sizing-content
    min-h-20 max-h-64
    w-full p-3
    border rounded
    resize-none
  "
  placeholder="Type, watch me grow"
></textarea>
```

Replaces the JS resize-on-input hack. Browser handles the size adjustment per keystroke.

## Native Date Picker That Respects Dark Mode

```html
<input type="date" class="scheme-light-dark dark:scheme-dark border rounded p-2" />
```

The native picker icon and dropdown adopt the colour scheme. Without `scheme-*` the picker stays light even in dark mode.

## Variable-Font Tracking with Width Axis

```html
<style>
  @font-face {
    font-family: "Inter Variable";
    src: url("/fonts/Inter-Variable.woff2") format("woff2");
    font-weight: 100 900;
    font-stretch: 50% 200%;
  }
</style>

<h1 class="font-[Inter Variable] font-stretch-expanded">Expanded headline</h1>
<p class="font-stretch-condensed">Condensed body text</p>
<p class="font-stretch-[83%]">Custom 83% width</p>
```

## Four-Layer Shadow Composition

```html
<button
  class="
    px-6 py-3 rounded-lg
    bg-gradient-to-b from-blue-500 to-blue-600
    text-white
    shadow-lg
    inset-shadow-sm
    inset-shadow-white/20
    ring-2 ring-blue-700
    inset-ring-1 inset-ring-white/40
  "
>
  Layered Button
</button>
```

Renders : outer shadow + inner highlight (inset-shadow-white/20) + outer ring + thin inner glow (inset-ring-white/40). All in one `box-shadow` declaration.

## Tab Switcher with Starting-Style Slide-In

```html
<div role="tablist" class="flex gap-2 border-b">
  <button role="tab" aria-selected="true">Active</button>
  <button role="tab">Inactive</button>
</div>

<div role="tabpanel" class="
  transition-all duration-200 transition-discrete
  starting:opacity-0 starting:translate-y-2
  opacity-100 translate-y-0
">
  Tab content slides in on first render.
</div>
```

## Form with Native Picker + Dark Mode + Field-Sizing

```html
<form class="space-y-4 p-6 bg-white dark:bg-slate-900 dark:scheme-dark scheme-light-dark">
  <label class="block">
    <span class="block text-sm">Description</span>
    <textarea
      class="field-sizing-content min-h-16 w-full border rounded p-2"
    ></textarea>
  </label>

  <label class="block">
    <span class="block text-sm">Date</span>
    <input type="date" class="border rounded p-2 scheme-light-dark dark:scheme-dark" />
  </label>

  <label class="block">
    <span class="block text-sm">Notes</span>
    <textarea
      class="field-sizing-content max-h-40 w-full border rounded p-2"
    ></textarea>
  </label>
</form>
```

## Hover Card with Composed Effects

```html
<div class="
  group relative p-6 rounded-lg
  bg-white dark:bg-slate-800
  shadow-md
  inset-shadow-sm inset-shadow-black/5
  ring-1 ring-black/10
  hover:shadow-xl
  hover:inset-shadow-sm
  hover:ring-2 hover:ring-blue-500
  transition-all duration-200
">
  <h3 class="font-bold">Card Title</h3>
  <p>Content</p>
</div>
```

## Verified Sources

- https://tailwindcss.com/docs/transition-behavior
- https://tailwindcss.com/docs/field-sizing
- https://tailwindcss.com/docs/color-scheme
- https://tailwindcss.com/docs/font-stretch
- https://tailwindcss.com/docs/box-shadow
- https://developer.mozilla.org/en-US/docs/Web/CSS/@starting-style

Last verified : 2026-05-19.
