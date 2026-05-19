# Methods : Tailwind Dark Mode

Complete configuration grammar for v3 and v4 dark-mode strategies, plus the theme-token swap recipe.

## 1. v3 `darkMode` config schema

Located in `tailwind.config.js` (or `.ts`, `.mjs`) at the top level of the export.

### Values

| Value | Selector emitted by Tailwind | When `dark:` applies |
|-------|-------------------------------|----------------------|
| `'media'` (default) | `@media (prefers-color-scheme: dark) { ... }` | OS preference |
| `'class'` (legacy) | `:is(.dark *)` (synthetic; specificity follows class) | Any `.dark` class on an ancestor |
| `'selector'` (v3.4.1+) | `&:where(.dark, .dark *)` | Same as `'class'` but with `:where()` to normalize specificity |
| `['selector', '<custom-selector>']` | `&:where(<custom-selector>)` | Any element matching the selector |
| `['variant', '<selector>']` | `<selector>` (raw, no `:where()` wrap) | Fully custom selector logic |
| `['variant', ['<sel1>', '<sel2>']]` | Multi-selector union | Multiple dark-mode triggers (e.g., both media AND class) |

### Full schema examples

```js
// Default media strategy (no config needed)
module.exports = {}

// Explicit media
module.exports = { darkMode: 'media' }

// Legacy class strategy
module.exports = { darkMode: 'class' }

// Selector strategy (recommended v3.4.1+)
module.exports = { darkMode: 'selector' }

// Attribute-based custom selector
module.exports = { darkMode: ['selector', '[data-theme="dark"]'] }

// Anti-toggle (light wins)
module.exports = { darkMode: ['variant', '&:not(.light *)'] }

// Combined media + class fallback
module.exports = {
  darkMode: ['variant', [
    '@media (prefers-color-scheme: dark) { &:not(.light *) }',
    '&:is(.dark *)',
  ]],
}
```

### Behavioural notes

- `'class'` and `'selector'` produce the same end-user behaviour. The difference is internal : `'selector'` wraps in `:where()` for predictable specificity. ALWAYS use `'selector'` in new v3.4.1+ code.
- `['selector', '[data-theme="dark"]']` is the canonical attribute pattern. Tailwind auto-wraps in `:where()`.
- `['variant', ...]` is the escape hatch for everything else; you control the selector entirely.

## 2. v4 `@custom-variant dark` directive

Located in the main CSS file (the one with `@import "tailwindcss"`). v4 removed the JS `darkMode` config entirely.

### Grammar

```
@custom-variant <variant-name> (<selector-expression>);
```

For dark mode specifically, `<variant-name>` is `dark` (this overrides the default media-query-based `dark` variant).

### Full schema examples

```css
/* Default media (no directive needed) */
@import "tailwindcss";

/* Class strategy */
@import "tailwindcss";
@custom-variant dark (&:where(.dark, .dark *));

/* Attribute strategy */
@import "tailwindcss";
@custom-variant dark (&:where([data-theme=dark], [data-theme=dark] *));

/* Custom selector (anti-toggle pattern) */
@import "tailwindcss";
@custom-variant dark (&:not(.light *));

/* Multi-selector union */
@import "tailwindcss";
@custom-variant dark (
  &:where(.dark, .dark *),
  &:where([data-theme=dark], [data-theme=dark] *)
);
```

### Behavioural notes

- `&` refers to the element targeted by the utility class.
- `:where(.dark, .dark *)` matches BOTH the element with class `dark` AND any descendant. This makes `<html class="dark"> ... <div class="dark:bg-slate-900">` work because the `<div>` is a descendant of an element matching `.dark`.
- `:where()` keeps specificity at 0, so `dark:` utilities cascade naturally and can be overridden by more-specific rules without `!important`.
- The `@custom-variant dark` line REPLACES the default `dark` variant entirely. After this directive, `dark:` no longer reacts to `prefers-color-scheme` unless you include it in the selector expression.

## 3. The `dark:` variant

Same syntax in v3 and v4. Apply as a prefix on any utility :

```html
<div class="bg-white dark:bg-slate-900 text-slate-900 dark:text-white">
```

Stacks with other variants :

```html
<button class="bg-blue-500 hover:bg-blue-600 dark:bg-blue-700 dark:hover:bg-blue-800">
```

Variant stacking order in v4 reads left-to-right (changed from v3's right-to-left for arbitrary variants; see LESSONS.md L-004) but `dark:` is unaffected because it is not an arbitrary-variant.

## 4. Theme-token swap recipe (production-grade pattern)

### v3 recipe

```css
/* app.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 255 255 255;
    --foreground: 15 23 42;
    --card: 255 255 255;
    --card-foreground: 15 23 42;
    --popover: 255 255 255;
    --popover-foreground: 15 23 42;
    --primary: 15 23 42;
    --primary-foreground: 248 250 252;
    --secondary: 241 245 249;
    --secondary-foreground: 15 23 42;
    --muted: 241 245 249;
    --muted-foreground: 100 116 139;
    --accent: 241 245 249;
    --accent-foreground: 15 23 42;
    --destructive: 239 68 68;
    --destructive-foreground: 248 250 252;
    --border: 226 232 240;
    --input: 226 232 240;
    --ring: 15 23 42;
    --radius: 0.5rem;
  }

  .dark {
    --background: 15 23 42;
    --foreground: 248 250 252;
    --card: 15 23 42;
    --card-foreground: 248 250 252;
    --popover: 15 23 42;
    --popover-foreground: 248 250 252;
    --primary: 248 250 252;
    --primary-foreground: 15 23 42;
    --secondary: 30 41 59;
    --secondary-foreground: 248 250 252;
    --muted: 30 41 59;
    --muted-foreground: 148 163 184;
    --accent: 30 41 59;
    --accent-foreground: 248 250 252;
    --destructive: 127 29 29;
    --destructive-foreground: 248 250 252;
    --border: 30 41 59;
    --input: 30 41 59;
    --ring: 226 232 240;
  }
}
```

```js
// tailwind.config.js
module.exports = {
  darkMode: 'selector',
  content: ['./src/**/*.{html,js,ts,jsx,tsx}'],
  theme: {
    extend: {
      colors: {
        background: 'rgb(var(--background) / <alpha-value>)',
        foreground: 'rgb(var(--foreground) / <alpha-value>)',
        card: {
          DEFAULT: 'rgb(var(--card) / <alpha-value>)',
          foreground: 'rgb(var(--card-foreground) / <alpha-value>)',
        },
        primary: {
          DEFAULT: 'rgb(var(--primary) / <alpha-value>)',
          foreground: 'rgb(var(--primary-foreground) / <alpha-value>)',
        },
        // ... repeat for secondary, muted, accent, destructive, border, input, ring
      },
      borderRadius: {
        lg: 'var(--radius)',
        md: 'calc(var(--radius) - 2px)',
        sm: 'calc(var(--radius) - 4px)',
      },
    },
  },
}
```

Tokens are stored as space-separated RGB triplets (not `rgb()` strings) so the `<alpha-value>` placeholder works with opacity modifiers like `bg-background/50`.

### v4 recipe

```css
@import "tailwindcss";
@custom-variant dark (&:where(.dark, .dark *));

@theme {
  --color-background: oklch(1 0 0);
  --color-foreground: oklch(0.15 0.02 250);
  --color-card: oklch(1 0 0);
  --color-card-foreground: oklch(0.15 0.02 250);
  --color-primary: oklch(0.2 0.04 250);
  --color-primary-foreground: oklch(0.98 0.005 250);
  --color-secondary: oklch(0.96 0.005 250);
  --color-secondary-foreground: oklch(0.2 0.04 250);
  --color-muted: oklch(0.96 0.005 250);
  --color-muted-foreground: oklch(0.55 0.02 250);
  --color-accent: oklch(0.96 0.005 250);
  --color-accent-foreground: oklch(0.2 0.04 250);
  --color-destructive: oklch(0.6 0.22 25);
  --color-destructive-foreground: oklch(0.98 0.005 250);
  --color-border: oklch(0.9 0.01 250);
  --color-input: oklch(0.9 0.01 250);
  --color-ring: oklch(0.2 0.04 250);
  --radius-lg: 0.5rem;
  --radius-md: calc(var(--radius-lg) - 2px);
  --radius-sm: calc(var(--radius-lg) - 4px);
}

.dark {
  --color-background: oklch(0.15 0.02 250);
  --color-foreground: oklch(0.98 0.005 250);
  --color-card: oklch(0.15 0.02 250);
  --color-card-foreground: oklch(0.98 0.005 250);
  --color-primary: oklch(0.98 0.005 250);
  --color-primary-foreground: oklch(0.2 0.04 250);
  --color-secondary: oklch(0.25 0.03 250);
  --color-secondary-foreground: oklch(0.98 0.005 250);
  --color-muted: oklch(0.25 0.03 250);
  --color-muted-foreground: oklch(0.7 0.02 250);
  --color-accent: oklch(0.25 0.03 250);
  --color-accent-foreground: oklch(0.98 0.005 250);
  --color-destructive: oklch(0.45 0.18 25);
  --color-destructive-foreground: oklch(0.98 0.005 250);
  --color-border: oklch(0.25 0.03 250);
  --color-input: oklch(0.25 0.03 250);
  --color-ring: oklch(0.85 0.02 250);
}
```

v4 ships colours in OKLCH wide-gamut. The opacity modifier (`bg-background/50`) works automatically because v4 uses `color-mix()` under the hood.

## 5. The JS toggle anatomy (exact docs snippet)

```javascript
// Phase 1 : determine theme to apply
const userPref = localStorage.theme         // "dark" | "light" | undefined
const osPrefersDark = window.matchMedia("(prefers-color-scheme: dark)").matches
const isDark = userPref === "dark" || (userPref === undefined && osPrefersDark)

// Phase 2 : apply
document.documentElement.classList.toggle("dark", isDark)
```

Condensed (verbatim from v4 docs) :

```javascript
document.documentElement.classList.toggle(
  "dark",
  localStorage.theme === "dark" ||
    (!("theme" in localStorage) &&
      window.matchMedia("(prefers-color-scheme: dark)").matches)
)
```

### User actions

```javascript
// User picks Light
localStorage.theme = "light"

// User picks Dark
localStorage.theme = "dark"

// User picks System (delete the override)
localStorage.removeItem("theme")
```

After mutating `localStorage`, re-run the Phase 2 line OR re-run the full Phase-1-and-2 block.

### Listening for OS preference changes (optional)

```javascript
window
  .matchMedia("(prefers-color-scheme: dark)")
  .addEventListener("change", (e) => {
    if (!("theme" in localStorage)) {
      document.documentElement.classList.toggle("dark", e.matches)
    }
  })
```

ALWAYS guard with the `!("theme" in localStorage)` check; otherwise the OS-preference change would override the user's explicit choice.

## 6. Removed in v4 (migration map)

| v3 mechanism | v4 equivalent |
|---------------|---------------|
| `darkMode: 'media'` (default) | Omit any directive (default) |
| `darkMode: 'class'` | `@custom-variant dark (&:where(.dark, .dark *));` |
| `darkMode: 'selector'` | Same as class above |
| `darkMode: ['selector', '[data-theme="dark"]']` | `@custom-variant dark (&:where([data-theme=dark], [data-theme=dark] *));` |
| `darkMode: ['variant', '&:not(.light *)']` | `@custom-variant dark (&:not(.light *));` |
| `darkMode: ['variant', [array]]` | `@custom-variant dark (sel1, sel2);` |

ALWAYS run `npx @tailwindcss/upgrade` which handles this rename automatically.

## Sources

- https://tailwindcss.com/docs/dark-mode (v4)
- https://v3.tailwindcss.com/docs/dark-mode (v3)
- https://tailwindcss.com/docs/functions-and-directives (`@custom-variant`)
- https://tailwindcss.com/docs/upgrade-guide (migration map)

Verified 2026-05-19.
