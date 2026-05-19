# Tailwind CSS Arbitrary Values: Worked Examples

Verified against tailwindcss.com/docs/styling-with-utility-classes (2026-05-19).

## Example 1: Brand color outside the palette

```html
<!-- Twitter blue is #1da1f2, not in the default palette -->
<button class="bg-[#1da1f2] text-white px-4 py-2 rounded-md hover:bg-[#0f8cd5]">
  Sign in with Twitter
</button>
```

When this color appears 2+ times, ALWAYS extend the theme instead :

```css
/* v4 */
@theme {
  --color-twitter-500: oklch(0.69 0.171 244);
  --color-twitter-600: oklch(0.55 0.171 244);
}
```

```html
<button class="bg-twitter-500 hover:bg-twitter-600">Sign in</button>
```

## Example 2: Custom grid template

Tailwind ships `grid-cols-1` through `grid-cols-12`. Anything else needs arbitrary syntax.

```html
<!-- Two flexible columns sandwiching a fixed-width center -->
<div class="grid grid-cols-[1fr_300px_1fr] gap-4">
  <aside>Left</aside>
  <main>Center</main>
  <aside>Right</aside>
</div>

<!-- Auto-fit responsive cards -->
<div class="grid grid-cols-[repeat(auto-fit,minmax(20rem,1fr))] gap-4">
  <div>Card 1</div>
  <div>Card 2</div>
  <div>Card 3</div>
</div>

<!-- Mixed track sizes -->
<div class="grid grid-cols-[24rem_2.5rem_minmax(0,1fr)]">
  <aside class="bg-slate-100">Sidebar</aside>
  <div></div>
  <main>Content</main>
</div>
```

## Example 3: calc() and viewport-relative sizes

```html
<!-- Full viewport height minus a 4rem header -->
<main class="h-[calc(100vh-4rem)] overflow-y-auto">...</main>

<!-- Width that respects a parent padding -->
<div class="w-[calc(100%-2rem)] mx-auto">Card with negative-margin breakout</div>

<!-- Mixing arbitrary calc with theme tokens -->
<div class="max-h-[calc(100dvh-(--spacing(6)))]">
  Modal content area
</div>
```

## Example 4: Modifier-arbitrary (custom selector)

```html
<!-- Style every direct list-item child -->
<ul class="flex gap-2 [&>li]:rounded-full [&>li]:border [&>li]:px-3 [&>li]:py-1">
  <li>Pill</li>
  <li>Pill</li>
</ul>

<!-- Style every paragraph descendant in an article -->
<article class="[&_p]:text-slate-600 [&_p]:leading-relaxed [&_p]:mt-3">
  <p>First paragraph.</p>
  <p>Second paragraph.</p>
</article>

<!-- Style based on compound classes -->
<li class="cursor-grab [&.dragging]:cursor-grabbing">Drag-aware</li>

<!-- Style sibling chain via complex selector -->
<form>
  <input class="peer" />
  <p class="hidden peer-invalid:block text-red-600">Error</p>
  <div class="[&>[data-active]+span]:text-blue-600">
    <button data-active>...</button>
    <span>Becomes blue when sibling is data-active</span>
  </div>
</form>

<!-- Hide native scrollbar (vendor pseudo-element) -->
<div class="overflow-auto [&::-webkit-scrollbar]:hidden [scrollbar-width:none]">
  Scrollable content
</div>
```

## Example 5: Variant-arbitrary for Radix UI patterns

```html
<!-- Radix Popover content with side-aware animations -->
<div
  data-side="bottom"
  data-state="open"
  class="
    rounded-md border bg-white p-4 shadow-md
    data-[state=open]:animate-in
    data-[state=closed]:animate-out
    data-[state=closed]:fade-out-0
    data-[state=open]:fade-in-0
    data-[side=bottom]:slide-in-from-top-2
    data-[side=top]:slide-in-from-bottom-2
    data-[side=left]:slide-in-from-right-2
    data-[side=right]:slide-in-from-left-2
  "
>
  Popover content
</div>

<!-- Table column with sort indicator -->
<th aria-sort="ascending" class="aria-[sort=ascending]:bg-blue-50 aria-[sort=descending]:bg-red-50">
  Name
</th>

<!-- Browser feature detection with fallback -->
<div class="
  flex flex-col gap-4
  supports-[display:grid]:grid
  supports-[display:grid]:grid-cols-3
  supports-[display:grid]:gap-6
">
  Flex-fallback for browsers without grid
</div>
```

## Example 6: Arbitrary property (no namespace)

```html
<!-- CSS properties without a Tailwind utility -->
<div class="[mask-type:luminance] hover:[mask-type:alpha]">SVG mask</div>
<section class="[scroll-snap-stop:always] [scroll-snap-align:center]">Snap target</section>
<div class="[contain:layout] [content-visibility:auto]">Performance hint</div>
<div class="[view-transition-name:hero]">View Transitions API target</div>

<!-- Strip the number-input arrows -->
<input
  type="number"
  class="[appearance:textfield] [&::-webkit-inner-spin-button]:appearance-none [&::-webkit-outer-spin-button]:appearance-none"
/>

<!-- Custom CSS property declaration -->
<div class="[--my-custom-prop:42]">Sets --my-custom-prop: 42 on the element</div>
```

## Example 7: Type hints

```html
<!-- bg-[200px_100px] is ambiguous : size or position? -->
<div class="bg-[length:200px_100px] bg-[url('/pattern.png')] bg-repeat">
  Tiled at 200x100
</div>
<div class="bg-[position:200px_100px] bg-[url('/hero.jpg')]">
  Positioned at 200x100
</div>

<!-- text-[var(--c)] is ambiguous : color or font-size? -->
<div class="text-[color:var(--brand)]">Color utility</div>
<div class="text-[length:var(--display-size)]">Size utility</div>

<!-- v4 parens shorthand with type hints -->
<div class="text-(color:--brand)">v4 color from CSS var</div>
<div class="text-(length:--display-size)">v4 size from CSS var</div>

<!-- font-family from CSS var (always needs hint) -->
<div class="font-[family-name:var(--brand-font)]">v3 / v4 brackets</div>
<div class="font-(family-name:--brand-font)">v4 parens</div>

<!-- Background image from arbitrary url() -->
<div class="bg-[image:url('/hero.png')] bg-cover">v3 / v4 brackets</div>
```

## Example 8: CSS variables (v3 vs v4)

```html
<!-- v3 : ALWAYS brackets + var() -->
<button class="bg-[var(--brand-bg)] text-[var(--brand-fg)] hover:bg-[var(--brand-bg-hover)]">
  v3 brand button
</button>

<!-- v4 : parens shorthand is preferred (shorter, scans the same) -->
<button class="bg-(--brand-bg) text-(--brand-fg) hover:bg-(--brand-bg-hover)">
  v4 brand button
</button>

<!-- v4 : type-hinted parens for ambiguous CSS vars -->
<div class="text-(color:--accent) text-(length:--display-size)">
  v4 disambiguated
</div>

<!-- v4 : you can still use the v3 brackets form ; it works -->
<button class="bg-[var(--brand-bg)]">v4 still parses brackets</button>
```

## Example 9: Negative arbitrary values

Tailwind auto-detects negative values inside square brackets.

```html
<div class="top-[-113px]">Negative top</div>
<div class="mt-[-1rem]">Negative margin-top</div>
<div class="rotate-[-15deg]">Negative rotation</div>

<!-- Equivalent shorthand form -->
<div class="-top-[113px]">Same effect, prefix bang</div>
```

## Example 10: Reaching for theme tokens instead

Most "I need an arbitrary value" reflexes are wrong : the theme already has the value.

```html
<!-- AVOID : repeated arbitrary value -->
<div class="p-[16px] gap-[16px] mb-[16px]">Triple repetition</div>

<!-- PREFER : theme token -->
<div class="p-4 gap-4 mb-4">Identical CSS, scans cleanly, uses design system</div>

<!-- AVOID (v4) : arbitrary for integer-multiple spacing -->
<div class="mt-[68px] gap-[172px]">v4 anti-pattern</div>

<!-- PREFER (v4) : dynamic integer spacing -->
<div class="mt-17 gap-43">v4 generates calc(--spacing * N) natively</div>

<!-- AVOID : arbitrary color used 5+ times -->
<button class="bg-[#1da1f2]">A</button>
<button class="bg-[#1da1f2]">B</button>
<button class="bg-[#1da1f2]">C</button>

<!-- PREFER : extend the theme once -->
<!-- @theme { --color-brand: oklch(0.69 0.171 244); } -->
<button class="bg-brand">A</button>
<button class="bg-brand">B</button>
<button class="bg-brand">C</button>
```

## Example 11: Combining everything

```html
<article
  data-state="loading"
  class="
    [view-transition-name:article]
    [contain:layout]
    grid grid-cols-[24rem_minmax(0,1fr)] gap-(--spacing-6)
    [&_h2]:text-[1.75rem]/[1.2]
    [&_p]:text-(color:--muted-fg)
    data-[state=loading]:opacity-[0.6]
    data-[state=loading]:[pointer-events:none]
    supports-[backdrop-filter]:backdrop-blur-md
    not-supports-[backdrop-filter]:bg-white/95
  "
>
  <h2>Heading</h2>
  <p>Body text</p>
</article>
```

Every flavour (value-arbitrary, type-hinted, modifier-arbitrary, variant-arbitrary, arbitrary property, supports-feature query) composes cleanly.
