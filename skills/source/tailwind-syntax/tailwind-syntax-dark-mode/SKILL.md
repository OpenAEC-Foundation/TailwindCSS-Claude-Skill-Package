---
name: tailwind-syntax-dark-mode
description: >
  Use when adding, debugging, or migrating dark-mode support in a Tailwind
  project: choosing between the default `prefers-color-scheme` strategy and
  user-toggleable class or attribute strategies, wiring the JavaScript
  toggle, swapping CSS-variable token values for light vs dark, or
  porting v3 `darkMode` configuration to the v4 `@custom-variant dark`
  directive.
  Prevents the four canonical dark-mode failures: assuming `dark:`
  utilities need configuration before they work, leaving v3's
  `darkMode: 'class'` config in a v4 project (silently ignored),
  forgetting the v4 `@custom-variant dark` directive when migrating
  from v3 `darkMode: 'class'`, and shipping a toggle that flashes
  the wrong theme on first paint because the script runs after CSS.
  Covers: default `media` strategy, class-based toggling (v3
  `darkMode: 'class'` and 'selector', v4 `@custom-variant dark
  (&:where(.dark, .dark *))`), attribute-based toggling
  (`[data-theme="dark"]`), custom variant strategies, theme-token
  swap via CSS variables, and the canonical JavaScript toggle
  snippet using `localStorage` plus `matchMedia`. Always shows
  v3 and v4 in side-by-side blocks; never fuses a single snippet
  that hides the version difference (per D-008).
  Keywords: dark mode, dark variant, dark:bg-, dark:text-,
  prefers-color-scheme, darkMode class, darkMode selector,
  custom-variant dark, where dark class, data-theme dark,
  toggle dark mode, localStorage theme, matchMedia,
  next-themes, light dark system, color scheme,
  how do I add dark mode, dark mode not working,
  dark utilities ignored, flash of wrong theme, FOUC,
  flash of unstyled content, OS dark mode, system theme,
  v3 to v4 dark mode migration, darkMode config removed.
license: MIT
compatibility: "Designed for Claude Code. Requires Tailwind CSS v3.4 or v4.0+."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# Tailwind CSS : Dark Mode

Three strategies, four moving parts. ALWAYS pick a strategy first, then implement all four parts (CSS configuration, markup, toggle script, optional theme-token swap) consistently. NEVER mix v3 config syntax with v4 directive syntax in one project.

## Quick Reference

### The three strategies

| Strategy | When the `dark:` variant applies | Pick when |
|----------|-----------------------------------|-----------|
| `media` (default) | OS-level `prefers-color-scheme: dark` | NEVER need a manual user toggle; ALWAYS respect OS preference verbatim |
| `class` / `selector` | A class on `<html>` (default `.dark`) | User has a UI toggle; want to override OS preference |
| `attribute` (custom) | A data attribute (e.g., `data-theme="dark"`) | Existing design system already uses attribute-based theming |

### Three invariants

1. ALWAYS apply `dark:` utilities to the element that should react. NEVER attempt a global stylesheet swap; Tailwind's variant model is per-utility.
2. ALWAYS run the JS toggle BEFORE the page paints. NEVER place the toggle inside a deferred `<script>` or wait for React hydration; a flash-of-wrong-theme is the visible failure.
3. ALWAYS keep the strategy choice consistent across v3 and v4 boundaries. NEVER leave a v3 `darkMode: 'class'` config in a v4 project: v4 silently ignores it.

### v3 vs v4 configuration (side-by-side)

The strategy is the same conceptually; only the configuration mechanism differs.

| Strategy | v3 (in `tailwind.config.js`) | v4 (in main CSS file) |
|----------|------------------------------|------------------------|
| `media` (default) | Omit or `darkMode: 'media'` | Omit (default) |
| Class on `<html>` | `darkMode: 'class'` (legacy) or `darkMode: 'selector'` (v3.4.1+) | `@custom-variant dark (&:where(.dark, .dark *));` |
| Custom data attribute | `darkMode: ['selector', '[data-theme="dark"]']` | `@custom-variant dark (&:where([data-theme=dark], [data-theme=dark] *));` |
| Fully custom selector | `darkMode: ['variant', '&:not(.light *)']` | `@custom-variant dark (&:not(.light *));` |

ALWAYS read the table top-to-bottom : pick a strategy first, then apply the v3 column OR the v4 column, never mix.

## Decision Tree 1 : Which strategy?

```
Q1. Does the project need a user-facing toggle (light / dark / system)?
    no  → ALWAYS choose `media` (default). Done. The `dark:` variant
          already works against OS preference; no config needed.
    yes → Q2

Q2. Does the existing design system already use a data attribute
    (e.g., shadcn / Radix Themes / next-themes default)?
    no  → ALWAYS use class strategy : v3 `darkMode: 'selector'`
          (or legacy 'class'), v4 `@custom-variant dark (&:where(.dark, .dark *))`.
          The user toggles `<html class="dark">`.
    yes → ALWAYS use attribute strategy : v3
          `darkMode: ['selector', '[data-theme="dark"]']`, v4
          `@custom-variant dark (&:where([data-theme=dark], [data-theme=dark] *))`.
          The user toggles `<html data-theme="dark">`.
```

## Decision Tree 2 : Where do dark styles LIVE?

```
Q1. Does the same DOM element have light AND dark visual rules?
    yes → ALWAYS use `dark:` variant on the utility :
          `class="bg-white text-slate-900 dark:bg-slate-900 dark:text-white"`
    no  → Q2

Q2. Do you have a design system with semantic tokens
    (e.g., `--background`, `--foreground`) that should swap?
    yes → ALWAYS swap the CSS variable VALUES inside a `.dark` selector
          (or `[data-theme=dark]`). See pattern 4. Then markup stays
          `class="bg-background text-foreground"` with no `dark:`
          variants needed.
    no  → write `dark:` variants per utility
```

## Decision Tree 3 : Toggle script placement

```
Q1. Does the framework support inline script in `<head>` (Next.js, Astro,
    Vite + index.html, Remix, plain HTML)?
    yes → ALWAYS place the toggle as an inline script in `<head>`
          BEFORE any CSS link. This runs synchronously before paint,
          eliminating the flash.
    no  → Q2

Q2. Does the framework have an SSR escape hatch (Next.js Script
    `beforeInteractive`, Astro fragment, SvelteKit `+layout.svelte`
    with `data-sveltekit-preload-data`)?
    yes → ALWAYS use it.
    no  → Use `next-themes` or equivalent that solves the
          flash via the same inline-script trick under the hood.
```

NEVER place the toggle inside a deferred React `useEffect`, Vue `mounted()`, or Svelte `onMount` : it runs after the first paint and visibly flashes the wrong theme.

## Patterns

### Pattern 1 : Default `media` strategy

ALWAYS use this when no user toggle is required.

v3 (`tailwind.config.js`) :

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ['./src/**/*.{html,js,ts,jsx,tsx}'],
  // darkMode defaults to 'media' ; can also be explicit :
  // darkMode: 'media',
}
```

v4 (`app/globals.css` or equivalent) :

```css
@import "tailwindcss";
/* No darkMode config needed ; `dark:` variants respect prefers-color-scheme by default */
```

Markup :

```html
<div class="bg-white text-slate-900 dark:bg-slate-900 dark:text-white">
  Reacts to OS preference automatically.
</div>
```

### Pattern 2 : Class strategy (user toggle on `<html class="dark">`)

v3 (recommended `selector` form since v3.4.1) :

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ['./src/**/*.{html,js,ts,jsx,tsx}'],
  darkMode: 'selector',   // v3.4.1+ recommended over the legacy 'class'
}
```

v3 legacy form (still valid but superseded) :

```js
module.exports = {
  darkMode: 'class',
}
```

v4 :

```css
@import "tailwindcss";
@custom-variant dark (&:where(.dark, .dark *));
```

Markup (both versions) :

```html
<html class="dark">
  <body>
    <div class="bg-white text-slate-900 dark:bg-slate-900 dark:text-white">
      Reacts to the `.dark` class on <html>.
    </div>
  </body>
</html>
```

ALWAYS toggle the class on `<html>` (not `<body>`) for canonical Tailwind behavior. The `:where(.dark, .dark *)` selector matches BOTH the ancestor AND descendants, so the variant applies regardless of nesting depth.

### Pattern 3 : Attribute strategy (`data-theme="dark"`)

v3 :

```js
module.exports = {
  darkMode: ['selector', '[data-theme="dark"]'],
}
```

v4 :

```css
@import "tailwindcss";
@custom-variant dark (&:where([data-theme=dark], [data-theme=dark] *));
```

Markup :

```html
<html data-theme="dark">
  <body>
    <div class="bg-background text-foreground dark:bg-slate-900 dark:text-white">
      Reacts to data-theme.
    </div>
  </body>
</html>
```

ALWAYS use this strategy when integrating with `next-themes` (`attribute="data-theme"` config), shadcn ui (which uses class by default but supports attribute via `next-themes`), or any system that already toggles a data attribute.

### Pattern 4 : Theme-token swap (CSS variables)

ALWAYS pair this pattern with strategy 2 or 3 when a multi-theme design system exists. NEVER hand-write `dark:` variants on every utility in such a system : the variable swap covers the whole tree.

v3 (CSS file imported alongside Tailwind) :

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 255 255 255;
    --foreground: 15 23 42;
    --primary: 59 130 246;
    --primary-foreground: 255 255 255;
  }

  .dark {
    --background: 15 23 42;
    --foreground: 248 250 252;
    --primary: 96 165 250;
    --primary-foreground: 15 23 42;
  }
}
```

Plus a corresponding `tailwind.config.js` theme extension :

```js
module.exports = {
  darkMode: 'selector',
  theme: {
    extend: {
      colors: {
        background: 'rgb(var(--background) / <alpha-value>)',
        foreground: 'rgb(var(--foreground) / <alpha-value>)',
        primary: {
          DEFAULT: 'rgb(var(--primary) / <alpha-value>)',
          foreground: 'rgb(var(--primary-foreground) / <alpha-value>)',
        },
      },
    },
  },
}
```

v4 (single CSS file, no JS config needed) :

```css
@import "tailwindcss";
@custom-variant dark (&:where(.dark, .dark *));

@theme {
  --color-background: oklch(1 0 0);
  --color-foreground: oklch(0.15 0.02 250);
  --color-primary: oklch(0.55 0.18 250);
  --color-primary-foreground: oklch(1 0 0);
}

.dark {
  --color-background: oklch(0.15 0.02 250);
  --color-foreground: oklch(0.98 0.005 250);
  --color-primary: oklch(0.7 0.16 250);
  --color-primary-foreground: oklch(0.15 0.02 250);
}
```

Markup (both versions) :

```html
<html class="dark">
  <body class="bg-background text-foreground">
    <button class="bg-primary text-primary-foreground">Save</button>
  </body>
</html>
```

ALWAYS keep the same token names across light and dark; only the VALUES change inside the `.dark` selector. NEVER introduce a `dark-primary` token; that defeats the swap model.

### Pattern 5 : The canonical JavaScript toggle (verbatim from v4 docs)

ALWAYS run this snippet AS EARLY AS POSSIBLE in the page lifecycle. Inline in `<head>` is the only safe location for SSR / static HTML; framework-specific equivalents below.

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>App</title>
  <script>
    document.documentElement.classList.toggle(
      "dark",
      localStorage.theme === "dark" ||
        (!("theme" in localStorage) &&
          window.matchMedia("(prefers-color-scheme: dark)").matches)
    );
  </script>
  <link rel="stylesheet" href="/styles.css" />
</head>
<body>
  <button id="theme-light">Light</button>
  <button id="theme-dark">Dark</button>
  <button id="theme-system">System</button>
  <script>
    document.getElementById("theme-light").onclick = () => {
      localStorage.theme = "light";
      document.documentElement.classList.remove("dark");
    };
    document.getElementById("theme-dark").onclick = () => {
      localStorage.theme = "dark";
      document.documentElement.classList.add("dark");
    };
    document.getElementById("theme-system").onclick = () => {
      localStorage.removeItem("theme");
      document.documentElement.classList.toggle(
        "dark",
        window.matchMedia("(prefers-color-scheme: dark)").matches
      );
    };
  </script>
</body>
</html>
```

ALWAYS check `localStorage.theme` FIRST (explicit user choice), THEN fall back to `prefers-color-scheme` (OS preference). NEVER check the OS preference first; that ignores the user's explicit override.

### Pattern 6 : Framework-specific toggle placement

| Framework | Where the inline script goes |
|-----------|-------------------------------|
| Next.js (App Router) | In `app/layout.tsx` `<head>` via a literal `<script>` element OR use `next-themes` package |
| Next.js (Pages Router) | In `pages/_document.tsx` `<Head>` |
| Vite + React | In `index.html` `<head>` before the `<div id="root">` |
| Astro | In the layout component `<head>`, before any client component |
| SvelteKit | In `src/app.html` `<head>` |
| Remix / React Router v7 | In `root.tsx` `<head>` |
| Plain HTML | In `<head>` before the stylesheet link |

ALWAYS place the inline script BEFORE any stylesheet link to guarantee the class is set when the first CSS parse happens. NEVER place it after the stylesheet : the browser may paint once with the wrong theme.

## Anti-patterns (summary; full list in references/anti-patterns.md)

NEVER do these :

1. NEVER leave a v3 `darkMode: 'class'` config in a v4 project. v4 ignores the field silently; ALWAYS migrate to `@custom-variant dark (...)` in CSS.
2. NEVER run the toggle inside a `useEffect` / `onMount` / `mounted()` lifecycle hook : the wrong theme will flash on first paint.
3. NEVER toggle the class on `<body>` instead of `<html>` : the `:where(.dark, .dark *)` selector targets the ancestor chain; `body.dark` works only if every styled element is a descendant, which fails for portals.
4. NEVER assume `dark:` variants need configuration before they work in v4 : the default `media` behaviour is automatic; configuration is needed ONLY to switch from `media` to class or attribute strategy.
5. NEVER check `prefers-color-scheme` before `localStorage` in the toggle script : the user's explicit override must win over the OS preference.
6. NEVER write a `darkMode: ['variant', ...]` v3 config and a `@custom-variant dark` v4 directive in the same project. Pick one entrypoint per version.

## Reference Links

- [references/methods.md](references/methods.md) : exact v3 config schema, v4 directive grammar, theme-token-swap recipe
- [references/examples.md](references/examples.md) : full HTML files for each strategy, framework-specific toggle wiring
- [references/anti-patterns.md](references/anti-patterns.md) : flash-of-wrong-theme, body vs html, v3 config in v4, late-hook toggle

## Cross-references

- [tailwind-core-architecture](../../tailwind-core/tailwind-core-architecture/SKILL.md) : utility-first plus design tokens; why theme-token swap beats per-utility `dark:` variants in token-driven projects
- [tailwind-core-design-system](../../tailwind-core/tailwind-core-design-system/SKILL.md) : token-naming conventions that survive the swap
- [tailwind-core-v3-vs-v4](../../tailwind-core/tailwind-core-v3-vs-v4/SKILL.md) : full v3-to-v4 breaking-change catalogue including `darkMode` removal
- [tailwind-impl-config-v3](../../tailwind-impl/tailwind-impl-config-v3/SKILL.md) : full `darkMode` config field including `['variant', ...]`
- [tailwind-impl-config-v4](../../tailwind-impl/tailwind-impl-config-v4/SKILL.md) : `@custom-variant`, `@theme`, `@source` in CSS-first config
- [tailwind-impl-migration-v3-v4](../../tailwind-impl/tailwind-impl-migration-v3-v4/SKILL.md) : automated migration via `npx @tailwindcss/upgrade`
- [tailwind-syntax-variants](../tailwind-syntax-variants/SKILL.md) : `dark:` as one variant among `hover:`, `focus:`, `motion-safe:`, etc.

## Sources

All claims trace to URLs in `SOURCES.md` :

- https://tailwindcss.com/docs/dark-mode (v4 directive syntax, JS toggle snippet, attribute alternative)
- https://v3.tailwindcss.com/docs/dark-mode (v3 `darkMode` config options : media, class, selector, variant)
- https://tailwindcss.com/docs/functions-and-directives (`@custom-variant` grammar)
- https://tailwindcss.com/docs/upgrade-guide (v3-to-v4 darkMode migration path)

Verified 2026-05-19.
