# Examples : Tailwind Utility Classes

Real-world layout patterns composed from only the base utility families documented in `SKILL.md`. Each example is verified to render against Tailwind v4 defaults; v3 differences flagged where relevant.

## 1. Centered card with shadow and hover

```html
<div class="mx-auto mt-12 max-w-md rounded-lg bg-white p-6 shadow-md hover:shadow-lg transition-shadow">
  <h2 class="text-xl font-semibold text-slate-900">Card title</h2>
  <p class="mt-2 text-sm text-slate-600">
    Card body content lives here. The card is centred horizontally with a maximum width.
  </p>
  <div class="mt-4 flex gap-2">
    <button class="rounded-md bg-blue-600 px-4 py-2 text-sm font-medium text-white hover:bg-blue-700">Save</button>
    <button class="rounded-md border border-slate-300 px-4 py-2 text-sm font-medium text-slate-700 hover:bg-slate-50">Cancel</button>
  </div>
</div>
```

v3 note : `shadow-md` rendered slightly different shadow values in v3 because v4 reorganised the scale; consider adjusting to `shadow-lg` if porting from v3 and the visual feels too subtle.

## 2. Stat grid with three columns

```html
<dl class="grid grid-cols-1 gap-4 sm:grid-cols-3">
  <div class="rounded-lg bg-slate-50 p-6">
    <dt class="text-sm font-medium text-slate-500">Revenue</dt>
    <dd class="mt-2 text-3xl font-semibold text-slate-900">$45,231</dd>
  </div>
  <div class="rounded-lg bg-slate-50 p-6">
    <dt class="text-sm font-medium text-slate-500">Active users</dt>
    <dd class="mt-2 text-3xl font-semibold text-slate-900">2,338</dd>
  </div>
  <div class="rounded-lg bg-slate-50 p-6">
    <dt class="text-sm font-medium text-slate-500">Conversion</dt>
    <dd class="mt-2 text-3xl font-semibold text-slate-900">12.5%</dd>
  </div>
</dl>
```

ALWAYS read `grid-cols-1` as the mobile default and `sm:grid-cols-3` as the tablet-and-up override. NEVER expect `sm:` to mean small screens only; see [tailwind-syntax-responsive].

## 3. Sidebar layout (flex)

```html
<div class="flex min-h-screen">
  <aside class="w-64 shrink-0 border-r border-slate-200 bg-slate-50 p-4">
    <nav class="flex flex-col gap-1">
      <a class="rounded-md px-3 py-2 text-sm font-medium text-slate-700 hover:bg-slate-100">Dashboard</a>
      <a class="rounded-md px-3 py-2 text-sm font-medium text-slate-700 hover:bg-slate-100">Reports</a>
      <a class="rounded-md px-3 py-2 text-sm font-medium text-slate-700 hover:bg-slate-100">Settings</a>
    </nav>
  </aside>
  <main class="flex-1 p-8">
    <h1 class="text-2xl font-bold text-slate-900">Dashboard</h1>
    <p class="mt-2 text-slate-600">Main content area expands to fill available space.</p>
  </main>
</div>
```

Key utilities :
- `flex` on parent + `flex-1` on main = sidebar fixed width, content fills the rest.
- `shrink-0` on the sidebar prevents it from compressing when content overflows.
- `min-h-screen` ensures the layout fills the viewport even with sparse content.

v3 note : `shrink-0` is v4 syntax. v3 uses `flex-shrink-0`.

## 4. Centered hero with two CTAs

```html
<section class="bg-gradient-to-br from-slate-50 to-white py-24">
  <div class="mx-auto max-w-3xl px-4 text-center">
    <h1 class="text-4xl font-bold tracking-tight text-slate-900 sm:text-5xl md:text-6xl">
      Build faster with utility classes
    </h1>
    <p class="mt-6 text-lg leading-relaxed text-slate-600">
      A complete utility-first vocabulary that scales from prototype to production.
    </p>
    <div class="mt-10 flex justify-center gap-4">
      <button class="rounded-md bg-slate-900 px-6 py-3 text-base font-medium text-white hover:bg-slate-800">
        Get started
      </button>
      <button class="rounded-md border border-slate-300 px-6 py-3 text-base font-medium text-slate-900 hover:bg-slate-50">
        Read docs
      </button>
    </div>
  </div>
</section>
```

v4 note : `bg-gradient-to-br` becomes `bg-linear-to-br` in v4 markup. The v3 form continues to work via the upgrade tool's auto-rename.

## 5. Form field with label and helper

```html
<div class="space-y-2 max-w-sm">
  <label for="email" class="block text-sm font-medium text-slate-700">
    Email address
  </label>
  <input
    id="email"
    type="email"
    class="block w-full rounded-md border border-slate-300 px-3 py-2 text-sm shadow-xs focus:border-blue-500 focus:outline-none focus:ring-2 focus:ring-blue-500/30"
  />
  <p class="text-xs text-slate-500">We will never share your email.</p>
</div>
```

Key utilities :
- `shadow-xs` is v4-only; in v3 use `shadow-sm`.
- `focus:ring-blue-500/30` uses opacity-slash syntax (replaces v3 `ring-blue-500 ring-opacity-30`).
- `focus:outline-none` removes the default outline; ALWAYS pair with `focus:ring-*` for accessibility.

## 6. Responsive nav with hidden mobile menu

```html
<header class="border-b border-slate-200 bg-white">
  <div class="mx-auto flex max-w-7xl items-center justify-between px-4 py-4">
    <a class="text-lg font-bold text-slate-900">Brand</a>
    <nav class="hidden gap-6 md:flex">
      <a class="text-sm text-slate-700 hover:text-slate-900">Products</a>
      <a class="text-sm text-slate-700 hover:text-slate-900">Pricing</a>
      <a class="text-sm text-slate-700 hover:text-slate-900">Docs</a>
    </nav>
    <button class="md:hidden rounded-md p-2 text-slate-700 hover:bg-slate-100" aria-label="Menu">
      <svg class="size-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <path stroke-linecap="round" stroke-linejoin="round" d="M4 6h16M4 12h16M4 18h16" />
      </svg>
    </button>
  </div>
</header>
```

Pattern : `hidden ... md:flex` shows the desktop nav from `md:` breakpoint and up. `md:hidden` hides the burger button from the same breakpoint. ALWAYS pair the two so exactly one is visible per breakpoint.

## 7. Stack of cards with alternating padding (grid)

```html
<div class="grid grid-cols-1 gap-6 md:grid-cols-2 lg:grid-cols-4">
  <article class="rounded-lg border border-slate-200 bg-white p-4">
    <h3 class="text-base font-semibold text-slate-900">Item 1</h3>
    <p class="mt-2 text-sm text-slate-600">Description.</p>
  </article>
  <article class="rounded-lg border border-slate-200 bg-white p-4">
    <h3 class="text-base font-semibold text-slate-900">Item 2</h3>
    <p class="mt-2 text-sm text-slate-600">Description.</p>
  </article>
  <article class="rounded-lg border border-slate-200 bg-white p-4">
    <h3 class="text-base font-semibold text-slate-900">Item 3</h3>
    <p class="mt-2 text-sm text-slate-600">Description.</p>
  </article>
  <article class="rounded-lg border border-slate-200 bg-white p-4">
    <h3 class="text-base font-semibold text-slate-900">Item 4</h3>
    <p class="mt-2 text-sm text-slate-600">Description.</p>
  </article>
</div>
```

Grid responsive pattern : `1` mobile, `2` tablet, `4` desktop. ALWAYS use `gap-*` over `space-y-*` / `space-x-*` for grid and flex layouts; the spacing primitive is correct for both.

## 8. Notification badge (absolute positioning)

```html
<button class="relative rounded-md bg-slate-100 p-2 text-slate-700 hover:bg-slate-200">
  <svg class="size-5" viewBox="0 0 24 24"><!-- bell --></svg>
  <span class="absolute -top-1 -right-1 flex size-5 items-center justify-center rounded-full bg-red-500 text-xs font-bold text-white">
    3
  </span>
</button>
```

Pattern : `relative` parent + `absolute` child with negative offsets. The `size-5` shorthand sets both width and height to 1.25rem.

## 9. Color modifier palette (opacity-slash showcase)

```html
<div class="space-y-2">
  <div class="bg-red-500 p-4 text-white">bg-red-500 (solid)</div>
  <div class="bg-red-500/75 p-4 text-white">bg-red-500/75</div>
  <div class="bg-red-500/50 p-4 text-white">bg-red-500/50</div>
  <div class="bg-red-500/25 p-4 text-white">bg-red-500/25</div>
  <div class="bg-red-500/10 p-4 text-red-900">bg-red-500/10 (subtle background)</div>
</div>
```

The slash modifier works on any color utility : `bg-`, `text-`, `border-`, `ring-`, `divide-`, `accent-`, `caret-`, `outline-`, `fill-`, `stroke-`, `placeholder-`, `from-`, `via-`, `to-`, `shadow-`, `decoration-`.

## 10. Typography sample

```html
<article class="mx-auto max-w-prose px-4 py-8">
  <h1 class="text-4xl font-bold tracking-tight text-slate-900">Article title</h1>
  <p class="mt-2 text-sm text-slate-500">Published 2026-05-19 · 6 min read</p>
  <p class="mt-6 text-base leading-relaxed text-slate-700">
    First paragraph. The leading-relaxed utility loosens line-height to 1.625 for body copy.
  </p>
  <h2 class="mt-8 text-2xl font-semibold tracking-tight text-slate-900">Section heading</h2>
  <p class="mt-4 text-base leading-relaxed text-slate-700">
    Subsequent paragraph. Note how <code class="rounded bg-slate-100 px-1.5 py-0.5 text-sm font-mono text-slate-800">code spans</code>
    inherit text-base and add their own padding.
  </p>
  <blockquote class="mt-6 border-l-4 border-slate-300 pl-4 italic text-slate-600">
    A pull quote in italic, with a left border indicating attribution.
  </blockquote>
</article>
```

`max-w-prose` (65ch) is the canonical comfortable line length for reading.

## 11. Sizing via fractions

```html
<div class="flex">
  <div class="w-1/3 bg-slate-100 p-4">One third</div>
  <div class="w-2/3 bg-slate-200 p-4">Two thirds</div>
</div>

<div class="flex mt-4">
  <div class="w-1/4 bg-blue-100 p-4">25%</div>
  <div class="w-1/2 bg-blue-200 p-4">50%</div>
  <div class="w-1/4 bg-blue-300 p-4">25%</div>
</div>

<div class="flex mt-4">
  <div class="w-1/5 bg-green-100 p-4">20%</div>
  <div class="w-2/5 bg-green-200 p-4">40%</div>
  <div class="w-2/5 bg-green-300 p-4">40%</div>
</div>
```

Fraction utilities support up to twelfths : `w-1/12` through `w-11/12`. For arbitrary fractions, use `w-[37%]`.

## 12. v3 to v4 migration snippet (side-by-side same component)

v3 :

```html
<div class="bg-white shadow-sm rounded-lg p-6 border border-gray-200">
  <h2 class="text-xl font-bold text-gray-900">Card</h2>
  <button class="!bg-blue-500 hover:bg-blue-600 bg-opacity-90 text-white px-4 py-2 rounded">Save</button>
</div>
```

v4 (equivalent rendering) :

```html
<div class="bg-white shadow-xs rounded-lg p-6 border border-slate-200">
  <h2 class="text-xl font-bold text-slate-900">Card</h2>
  <button class="bg-blue-500! hover:bg-blue-600 bg-blue-500/90 text-white px-4 py-2 rounded">Save</button>
</div>
```

Differences :
- `shadow-sm` → `shadow-xs` because v3 `shadow-sm` corresponds to v4 `shadow-xs` after the scale shift.
- `gray-*` → `slate-*` recommended for new v4 code (gray remains valid; slate aligns with the new neutral palette).
- `!bg-blue-500` → `bg-blue-500!` (important-modifier moved to suffix).
- `bg-opacity-90` → `bg-blue-500/90` (opacity utilities removed in v4).
- The `border` utility alone is fine in both because we set an explicit `border-slate-200`; in v4 omitting the color would yield `currentColor` instead of `gray-200`.

ALWAYS run `npx @tailwindcss/upgrade` instead of hand-migrating; the tool handles 95% of these renames. See [tailwind-impl-migration-v3-v4].
