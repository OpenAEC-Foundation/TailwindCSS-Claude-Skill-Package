# Anti-Patterns : Tailwind Utility Classes

Per-family anti-patterns. Most fail because of the content-scan model documented in [tailwind-core-architecture] or because v4 renamed / removed a v3 utility.

## 1. Using removed v4 opacity utilities

### Symptom
```html
<div class="bg-red-500 bg-opacity-50">background</div>
<p class="text-white text-opacity-75">text</p>
```
In v4 the opacity utilities have no effect. The background renders fully opaque; the text renders fully opaque.

### Root cause
v4 removed the per-property opacity utility families : `bg-opacity-*`, `text-opacity-*`, `border-opacity-*`, `divide-opacity-*`, `ring-opacity-*`, `placeholder-opacity-*`. The classes are silently ignored.

### Fix
ALWAYS use the slash modifier on the color utility itself :
```html
<div class="bg-red-500/50">background</div>
<p class="text-white/75">text</p>
```

## 2. Assuming `shadow-sm` is the same in v3 and v4

### Symptom
After migrating from v3 to v4, all `shadow-sm` elements look too prominent. Designer feedback : "the shadows are louder than before".

### Root cause
v4 inserted `shadow-2xs` and `shadow-xs` below the previously-smallest `shadow-sm`. The CSS value of v4 `shadow-sm` is what v3 called `shadow-md`. The whole scale shifted up.

| v3 class | v4 equivalent class | Why |
|----------|---------------------|-----|
| `shadow-sm` | `shadow-xs` | v3 small ≈ v4 extra-small after insertion |
| `shadow` (no suffix) | `shadow-sm` | v3 default ≈ v4 small after insertion |
| `shadow-md` | `shadow-md` | name unchanged but value slightly tweaked |
| `shadow-lg` | `shadow-lg` | unchanged |

### Fix
- ALWAYS run `npx @tailwindcss/upgrade` which handles this automatically.
- For manual fixes : map v3 names down one step in v4. `shadow-sm` → `shadow-xs`, `shadow` → `shadow-sm`.
- The same shift applies to `blur-*`, `rounded-*`, `drop-shadow-*`, `backdrop-blur-*`. Treat them identically.

## 3. `flex-shrink-0` / `flex-grow-1` in v4 code

### Symptom
```html
<aside class="flex-shrink-0 w-64">Sidebar</aside>
```
The sidebar still shrinks when content grows. The class is generated but does not apply.

### Root cause
v4 renamed the shorthand utilities :
- `flex-shrink-*` → `shrink-*`
- `flex-grow-*` → `grow-*`

In v4 there IS no `flex-shrink-0` utility. The class name is a dead string.

### Fix
```html
<aside class="shrink-0 w-64">Sidebar</aside>
```

## 4. `bg-gradient-to-r` in v4 code

### Symptom
```html
<div class="bg-gradient-to-r from-blue-500 to-purple-500">gradient</div>
```
No gradient renders in v4.

### Root cause
v4 renamed the linear-gradient family to align with `bg-conic-*` and `bg-radial-*` naming :
- `bg-gradient-*` → `bg-linear-*`

### Fix
```html
<div class="bg-linear-to-r from-blue-500 to-purple-500">gradient</div>
```

See [tailwind-syntax-gradients] for the full v4 gradient family including conic, radial, angle-specified linear (`bg-linear-45`), and interpolation modifiers (`/oklch`, `/srgb`).

## 5. `overflow-ellipsis` in v4 code

### Symptom
A long string should clip with an ellipsis. The HTML/CSS combination :
```html
<div class="w-32 overflow-hidden whitespace-nowrap overflow-ellipsis">A very long sentence...</div>
```
Renders without the ellipsis; the text just clips.

### Root cause
v4 renamed the text-overflow utilities :
- `overflow-ellipsis` → `text-ellipsis`
- `overflow-clip` → `text-clip` (when applied to text-overflow)

The new `overflow-clip` in v4 is a separate `overflow` value, not a `text-overflow` value.

### Fix
```html
<div class="w-32 overflow-hidden whitespace-nowrap text-ellipsis">A very long sentence...</div>
```

Or use the all-in-one shorthand :
```html
<div class="w-32 truncate">A very long sentence...</div>
```

## 6. `!font-bold` leading-important in v4

### Symptom
```html
<p class="!font-bold">important text</p>
```
The `!important` flag does not apply in v4.

### Root cause
v4 moved the important-modifier from prefix to suffix :
- v3 : `!font-bold` (leading bang)
- v4 : `font-bold!` (trailing bang)

### Fix
```html
<p class="font-bold!">important text</p>
```

NEVER use important-modifier as a routine override mechanism. Identify the source of competing specificity (a less-specific `@layer`, plugin ordering, an unlayered CSS rule) and fix it there. See [tailwind-errors-specificity].

## 7. Expecting `border` alone to produce a visible border in v4

### Symptom
```html
<div class="border p-4">card</div>
```
In v3, this renders a 1px gray-200 border. In v4, the border appears in the text color (`currentColor`), which is often near-black on white text, so the visual is very different. Designers report "the borders look too dark now".

### Root cause
v4 changed the default border color from `gray-200` to `currentColor`. The `border` utility alone sets only width and style; color now defaults to whatever the element's text color is.

### Fix
ALWAYS pair `border` with an explicit color :
```html
<div class="border border-slate-200 p-4">card</div>
```

Or, for a v3 visual restore globally :
```css
@layer base {
  *, ::after, ::before, ::backdrop, ::file-selector-button {
    border-color: var(--color-gray-200, currentcolor);
  }
}
```

Same fix logic for `ring` : v4 default ring color is `currentColor` and default width is 1px (v3 was 3px). Use `ring-3 ring-blue-500` or a base-layer shim.

## 8. Dynamic class assembly that the scanner cannot detect

### Symptom
```tsx
<div className={`bg-${color}-${shade} text-white p-4`}>...</div>
```
Works in dev; broken in production. The DOM has the class but the stylesheet has no rule.

### Root cause
The content scanner tokenises source files as plain text. The literal string in the source is `bg-${color}-${shade}`, never `bg-red-500`. The class is never generated. Detailed in [tailwind-core-architecture] and [tailwind-errors-dynamic-classes].

### Fix
Static map :
```tsx
const bg = { red: 'bg-red-500', blue: 'bg-blue-500' }[color] || 'bg-slate-500'
<div className={`${bg} text-white p-4`}>...</div>
```

Or v4 `@source inline()` with brace expansion :
```css
@source inline("{hover:,}bg-{red,blue,green}-{50,{100..900..100},950}");
```

## 9. `theme()` function with dot-notation in v4 CSS

### Symptom
```css
.card {
  background: theme(colors.slate.50);
  border-color: theme(colors.slate.200);
}
```
Build warns or fails in v4 : "theme function with dot notation is deprecated".

### Root cause
v4 deprecated dot-notation `theme()`. The new path uses the CSS-variable form.

### Fix
Use CSS variables directly (preferred) :
```css
.card {
  background: var(--color-slate-50);
  border-color: var(--color-slate-200);
}
```

Or, if `theme()` must remain :
```css
.card {
  background: theme(--color-slate-50);
  border-color: theme(--color-slate-200);
}
```

## 10. `space-y-4` on elements with margin overrides

### Symptom
```html
<div class="space-y-4">
  <p>One</p>
  <p class="mb-8">Two (with extra margin)</p>
  <p>Three</p>
</div>
```
Spacing between the second and third paragraph is unpredictable. v3 and v4 produce different gaps.

### Root cause
- v3 `space-y-*` selector : `> :not([hidden]) ~ :not([hidden])` adds top-margin.
- v4 `space-y-*` selector : `> :not(:last-child)` adds bottom-margin.

Either way, custom `mb-*` on a child fights the auto-applied margin from `space-y-*`. The interaction is fragile.

### Fix
ALWAYS prefer `flex flex-col gap-4` over `space-y-4` :
```html
<div class="flex flex-col gap-4">
  <p>One</p>
  <p class="mb-8">Two (mb-8 still works, but combined with gap-4 you have 8 + 4 stacking)</p>
  <p>Three</p>
</div>
```

The `gap-*` primitive is honoured by flex and grid containers without inter-sibling-selector trickery.

## 11. `outline-none` expecting the outline to be invisible-but-accessible

### Symptom
v3 :
```html
<button class="outline-none focus:ring-2 focus:ring-blue-500">Click</button>
```
On Windows High Contrast Mode (forced-colors), the focus indicator disappears.

### Root cause
- v3 `outline-none` set `outline: 2px solid transparent; outline-offset: 2px;` (the "visually hidden but reachable" pattern).
- v4 `outline-none` literally zeros the outline (`outline: none`), losing the High-Contrast-Mode hook.
- v4 introduced `outline-hidden` for the v3 visually-hidden-but-accessible behaviour.

### Fix
For accessibility, ALWAYS use `outline-hidden` in v4 :
```html
<button class="outline-hidden focus:ring-2 focus:ring-blue-500">Click</button>
```

## 12. Mixing v3 and v4 idioms in the same component

### Symptom
A component file has both `!font-bold` and `font-bold!`, `bg-gradient-to-r` and `bg-linear-to-r`, `shadow-sm` chosen as if v3 next to `shadow-xs`.

### Root cause
Partial migration : the file was edited in two versions without running the official upgrade tool to completion.

### Fix
ALWAYS run `npx @tailwindcss/upgrade` on a clean branch. ALWAYS audit the diff. NEVER hand-migrate piecewise; the renames are too numerous to track manually without tooling.

## 13. Believing `sm:` means "small screens only"

### Symptom
```html
<p class="sm:text-center">should be centered on mobile</p>
```
On desktop, the text is also centred. Designer expected the centring to apply only to small screens.

### Root cause
Tailwind breakpoints are mobile-first. `sm:` means "AT the `sm` breakpoint and ABOVE", i.e. roughly tablet and up.

### Fix
For desktop-first (apply only below a breakpoint), use the `max-*` prefix :
```html
<p class="max-sm:text-center">centered up to sm, default left from sm and up</p>
```

Or invert the logic : center by default, override at the breakpoint :
```html
<p class="text-center sm:text-left">centered on mobile, left from sm</p>
```

See [tailwind-syntax-responsive].

## Sources

- https://tailwindcss.com/docs/upgrade-guide (rename and removal catalogue)
- https://tailwindcss.com/docs/colors (opacity modifier rules)
- https://tailwindcss.com/docs/box-shadow (v4 shadow scale)
- https://tailwindcss.com/docs/responsive-design (mobile-first semantics)
- LESSONS.md L-003, L-004, L-005

Verified 2026-05-19.
