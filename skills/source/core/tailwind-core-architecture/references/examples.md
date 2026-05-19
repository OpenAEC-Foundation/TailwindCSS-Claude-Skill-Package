# Examples : Tailwind Core Architecture

Concrete code examples illustrating utility-first composition, comparisons with alternative styling strategies, and component-extraction patterns.

## 1. Utility-first vs alternative strategies (same component, four ways)

### 1.1 Utility-first (Tailwind, recommended for design-system-driven projects)

```html
<div class="px-4 py-6 bg-white rounded-lg shadow-md hover:shadow-lg transition-shadow">
  <h2 class="text-xl font-bold text-slate-900">Card title</h2>
  <p class="mt-2 text-sm text-slate-600">Card body content.</p>
</div>
```

Properties :
- Zero stylesheet bytes for this card; all classes already exist in the generated bundle.
- Each utility is constrained to a theme token (`text-xl`, `bg-white`, `px-4`).
- Adding a state (`hover:`, `dark:`, `md:`) requires no extra CSS.

### 1.2 BEM

```html
<div class="card card--elevated">
  <h2 class="card__title">Card title</h2>
  <p class="card__body">Card body content.</p>
</div>
```

```css
.card {
  padding: 1rem 1.5rem;
  background: white;
  border-radius: 0.5rem;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
  transition: box-shadow 0.15s;
}
.card--elevated:hover {
  box-shadow: 0 4px 6px rgba(0,0,0,0.1);
}
.card__title {
  font-size: 1.25rem;
  font-weight: 700;
  color: #0f172a;
}
.card__body {
  margin-top: 0.5rem;
  font-size: 0.875rem;
  color: #475569;
}
```

Properties :
- Long-lived stylesheet that must stay in sync with markup.
- Naming overhead (block / element / modifier) on every component.
- Specificity is at risk once new modifiers stack.

### 1.3 CSS Modules

```tsx
import s from './Card.module.css'

export function Card({ title, body }) {
  return (
    <div className={s.card}>
      <h2 className={s.title}>{title}</h2>
      <p className={s.body}>{body}</p>
    </div>
  )
}
```

```css
/* Card.module.css */
.card { padding: 1rem 1.5rem; background: white; border-radius: 0.5rem; }
.title { font-size: 1.25rem; font-weight: 700; color: #0f172a; }
.body { margin-top: 0.5rem; font-size: 0.875rem; color: #475569; }
```

Properties :
- Locally scoped (no name collisions).
- Still requires per-component CSS files.
- No automatic design-system enforcement.

### 1.4 CSS-in-JS (runtime)

```tsx
import styled from 'styled-components'

const Card = styled.div`
  padding: 1rem 1.5rem;
  background: white;
  border-radius: 0.5rem;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
  transition: box-shadow 0.15s;
  &:hover { box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
`

export function CardComponent({ title, body }) {
  return <Card>...</Card>
}
```

Properties :
- Runtime cost (style serialisation on every render or hydration).
- Theming via React context, not CSS variables.
- SSR-heavier than zero-runtime alternatives.

### 1.5 Comparison verdict

For a design-system-driven project, ALWAYS prefer 1.1 (Tailwind utility-first). For a per-component bespoke graphic that intentionally lives outside the design system, 1.3 (CSS Modules) is acceptable. NEVER mix 1.1 and 1.4 in the same component tree.

## 2. Component extraction patterns

### 2.1 Inline loop (no extraction, same-file repetition)

```tsx
const items = [
  { id: 1, label: 'Inbox', count: 12 },
  { id: 2, label: 'Sent', count: 0 },
  { id: 3, label: 'Drafts', count: 3 },
]

export function Sidebar() {
  return (
    <ul>
      {items.map(item => (
        <li
          key={item.id}
          class="px-3 py-2 rounded-md hover:bg-slate-100 flex items-center justify-between"
        >
          <span>{item.label}</span>
          <span class="text-xs text-slate-500">{item.count}</span>
        </li>
      ))}
    </ul>
  )
}
```

ALWAYS use a loop when duplication is local to one file. NEVER extract to a separate component just to dedupe local lines.

### 2.2 React component extraction (cross-file repetition, framework available)

```tsx
type CardProps = { title: string; children: React.ReactNode }

export function Card({ title, children }: CardProps) {
  return (
    <div className="px-4 py-6 bg-white rounded-lg shadow-md hover:shadow-lg transition-shadow">
      <h2 className="text-xl font-bold text-slate-900">{title}</h2>
      <div className="mt-2 text-sm text-slate-600">{children}</div>
    </div>
  )
}
```

Caller :

```tsx
<Card title="Welcome">
  <p>This is a card.</p>
</Card>
```

ALWAYS prefer this pattern when a component framework is available. The card class list lives in exactly one place and per-instance variation flows through props.

### 2.3 `@apply` extraction (no component framework available, last resort)

v3 :

```css
@layer components {
  .card {
    @apply px-4 py-6 bg-white rounded-lg shadow-md transition-shadow;
  }
  .card:hover {
    @apply shadow-lg;
  }
}
```

v4 :

```css
@utility card {
  padding-inline: --spacing(4);
  padding-block: --spacing(6);
  background-color: var(--color-white);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-md);
  transition-property: box-shadow;
  transition-duration: 150ms;

  &:hover {
    box-shadow: var(--shadow-lg);
  }
}
```

Then in markup :

```html
<div class="card">
  <h2 class="text-xl font-bold">Card title</h2>
</div>
```

The v4 `@utility` form is **preferred** over `@layer components` because :
- Utilities applied on the same element properly override component properties without `!important`.
- It participates in variant generation (responsive, hover, dark) automatically.
- `--value()` and `--modifier()` enable functional utilities.

NEVER reach for `@apply` first when a component framework is in use; option 2.2 is always better.

## 3. The constraint-based design pattern

### 3.1 Token-driven utility composition (correct)

```html
<button class="px-4 py-2 bg-blue-600 text-white rounded-md text-sm font-medium hover:bg-blue-700">
  Save
</button>
```

Every value comes from the design system :
- `px-4`, `py-2` from the spacing scale.
- `bg-blue-600`, `bg-blue-700` from the colour palette.
- `rounded-md` from the radius scale.
- `text-sm`, `font-medium` from the type scale.

ALWAYS write tokens like this for system-bound decisions.

### 3.2 Escape-hatch arbitrary value (acceptable when scoped)

```html
<div class="absolute top-[37px] left-[219px]">
  Pixel-aligned annotation
</div>
```

Use case : one-off art positioning that does not belong in the theme. ALWAYS gate arbitrary values to truly one-off scenarios. NEVER scatter them across a codebase as a substitute for tokens.

### 3.3 Adding a token to the theme (system-wide value)

v3 :

```js
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: { brand: '#1da1f2' },
      spacing: { 18: '4.5rem' },
    }
  }
}
```

Use as : `bg-brand`, `p-18`.

v4 :

```css
@import "tailwindcss";

@theme {
  --color-brand: #1da1f2;
  --spacing-18: 4.5rem;
}
```

Use as : `bg-brand`, `p-18`.

ALWAYS add to the theme when a value will repeat in 3+ places.

## 4. Content scanning patterns

### 4.1 Static class names (correct)

```tsx
const buttonStyles = {
  primary: 'bg-blue-600 hover:bg-blue-700 text-white',
  secondary: 'bg-slate-200 hover:bg-slate-300 text-slate-900',
  danger: 'bg-red-600 hover:bg-red-700 text-white',
}

<button class={`px-4 py-2 rounded-md ${buttonStyles[variant]}`}>
  {label}
</button>
```

ALL utility tokens appear literally in source. The scanner finds them. The bundle includes them. Composition via runtime string concatenation is fine BECAUSE each class is a static literal in the source map; runtime only picks which literal to use.

### 4.2 Dynamic class names (broken)

```tsx
// NEVER do this
<button class={`bg-${color}-${shade} text-white px-4 py-2`}>
  {label}
</button>
```

The scanner sees the literal string `bg-${color}-${shade}`. Neither `bg-blue-600` nor any specific variant is ever a literal token. The class is **never generated**. The button has a class attribute that points to nothing.

### 4.3 The v4 escape hatch : `@source inline()`

When values genuinely come from data outside source, use brace expansion :

```css
@import "tailwindcss";

@source inline("{hover:,focus:,}bg-{red,blue,green,yellow}-{50,{100..900..100},950}");
```

This emits :
- `bg-red-50`, `bg-red-100`, ..., `bg-red-950`
- `hover:bg-red-50`, ..., `hover:bg-yellow-950`
- `focus:bg-red-50`, ..., `focus:bg-yellow-950`

ALWAYS prefer this over the static-map workaround ONLY when the value space is genuinely runtime and finite.

### 4.4 Scanning a node_modules package

```css
@import "tailwindcss";

@source "../node_modules/@acmecorp/ui-lib";
```

ALWAYS add an explicit `@source` for component libraries shipped via npm; the default v4 scanner excludes `node_modules`.

### 4.5 Excluding legacy directories

```css
@source not "../src/legacy";
```

ALWAYS prefer this over per-utility safelists when the entire path is dead.

## 5. Layer pattern : where custom CSS belongs

```css
@import "tailwindcss";

/* Global resets and typography defaults */
@layer base {
  html {
    scroll-behavior: smooth;
  }
  h1, h2, h3 {
    text-wrap: balance;
  }
}

/* Component shells extracted from markup */
@layer components {
  .prose-content blockquote {
    border-left: 4px solid var(--color-slate-200);
    padding-left: 1rem;
    font-style: italic;
  }
}

/* Custom single-purpose utilities (v3 style) */
@layer utilities {
  .scrollbar-hidden::-webkit-scrollbar {
    display: none;
  }
}

/* v4 preferred form for custom utilities */
@utility no-scrollbar {
  &::-webkit-scrollbar {
    display: none;
  }
}
```

NEVER place custom CSS outside any layer unless you explicitly want it to override everything else.

## 6. When utility-first WINS : a real example

A multi-team product platform with :
- A Figma design system (tokens for colour, spacing, radius, type).
- React for SPA flows, Astro for marketing pages, Phoenix LiveView for admin.
- 12 engineers, two designers.

Result with Tailwind :
- Token changes propagate from `@theme { }` to every page; designers can audit by reading CSS variables.
- New developers learn the utility vocabulary in ~2 days; no per-team class conventions to negotiate.
- Bundle stays under 30KB gzipped because only utilities used in markup are emitted.

## 7. When utility-first LOSES : a real example

A long-form journalism site where 90% of HTML is authored by editors in a WYSIWYG and stored in a CMS :
- Editors paste rich text containing `<p>`, `<h2>`, `<blockquote>`, `<img>` without classes.
- The dev team controls only the shell : nav, footer, article frame.

Result with Tailwind :
- The `.prose` plugin (`@tailwindcss/typography`) styles editor output via descendant selectors.
- Utility classes live on the shell, not on editor content.
- Acceptable, but the win over BEM + global typography rules is smaller than in design-system-driven projects.

ALWAYS scope Tailwind expectations to the part of the HTML developers control.
