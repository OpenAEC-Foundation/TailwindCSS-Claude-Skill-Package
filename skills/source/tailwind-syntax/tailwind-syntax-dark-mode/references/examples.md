# Examples : Tailwind Dark Mode

End-to-end working examples for each strategy. v3 and v4 columns kept separate per D-008.

## 1. Default `media` strategy (zero config)

### Files

`src/styles.css` (v4) :

```css
@import "tailwindcss";
```

`tailwind.config.js` (v3) :

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ['./src/**/*.{html,js,ts,jsx,tsx}'],
  // darkMode: 'media' is the default ; omit
}
```

### Markup

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <link rel="stylesheet" href="/styles.css" />
</head>
<body class="bg-white text-slate-900 dark:bg-slate-900 dark:text-white">
  <h1 class="text-2xl font-bold">Hello</h1>
  <p>This reacts to OS preference automatically.</p>
</body>
</html>
```

ALWAYS confirm the OS preference works by toggling system dark mode in your OS settings, then refreshing.

## 2. Class strategy with user toggle (full HTML page)

### v4 version

`src/styles.css` :

```css
@import "tailwindcss";
@custom-variant dark (&:where(.dark, .dark *));

@theme {
  --color-background: oklch(1 0 0);
  --color-foreground: oklch(0.15 0.02 250);
  --color-card: oklch(0.98 0.005 250);
}

.dark {
  --color-background: oklch(0.15 0.02 250);
  --color-foreground: oklch(0.98 0.005 250);
  --color-card: oklch(0.2 0.03 250);
}
```

`index.html` :

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>Dark mode demo</title>
  <script>
    document.documentElement.classList.toggle(
      "dark",
      localStorage.theme === "dark" ||
        (!("theme" in localStorage) &&
          window.matchMedia("(prefers-color-scheme: dark)").matches)
    );
  </script>
  <link rel="stylesheet" href="/src/styles.css" />
</head>
<body class="bg-background text-foreground min-h-screen">
  <header class="border-b border-slate-200 dark:border-slate-700 p-4 flex justify-between">
    <h1 class="text-xl font-bold">My app</h1>
    <div class="flex gap-2">
      <button onclick="setTheme('light')" class="rounded-md border border-slate-300 dark:border-slate-600 px-3 py-1 text-sm">Light</button>
      <button onclick="setTheme('dark')" class="rounded-md border border-slate-300 dark:border-slate-600 px-3 py-1 text-sm">Dark</button>
      <button onclick="setTheme('system')" class="rounded-md border border-slate-300 dark:border-slate-600 px-3 py-1 text-sm">System</button>
    </div>
  </header>

  <main class="p-8">
    <article class="bg-card rounded-lg shadow-md p-6 max-w-2xl">
      <h2 class="text-2xl font-semibold">Card title</h2>
      <p class="mt-2 text-sm text-slate-600 dark:text-slate-300">Body content using both token-swap (bg-card) and explicit dark: variants for fine-grained overrides.</p>
    </article>
  </main>

  <script>
    function setTheme(mode) {
      if (mode === "system") {
        localStorage.removeItem("theme");
        const osDark = window.matchMedia("(prefers-color-scheme: dark)").matches;
        document.documentElement.classList.toggle("dark", osDark);
      } else {
        localStorage.theme = mode;
        document.documentElement.classList.toggle("dark", mode === "dark");
      }
    }
  </script>
</body>
</html>
```

### v3 version (same component, v3 config)

`tailwind.config.js` :

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  darkMode: 'selector',
  content: ['./index.html', './src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      colors: {
        background: 'rgb(var(--background) / <alpha-value>)',
        foreground: 'rgb(var(--foreground) / <alpha-value>)',
        card: 'rgb(var(--card) / <alpha-value>)',
      },
    },
  },
}
```

`src/styles.css` :

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 255 255 255;
    --foreground: 15 23 42;
    --card: 248 250 252;
  }
  .dark {
    --background: 15 23 42;
    --foreground: 248 250 252;
    --card: 30 41 59;
  }
}
```

`index.html` is IDENTICAL between v3 and v4 except the `<link rel="stylesheet">` href.

## 3. Attribute strategy with next-themes (Next.js App Router)

### v4 setup

`app/globals.css` :

```css
@import "tailwindcss";
@custom-variant dark (&:where([data-theme=dark], [data-theme=dark] *));

@theme {
  --color-background: oklch(1 0 0);
  --color-foreground: oklch(0.15 0.02 250);
}

[data-theme=dark] {
  --color-background: oklch(0.15 0.02 250);
  --color-foreground: oklch(0.98 0.005 250);
}
```

`app/providers.tsx` :

```tsx
'use client'
import { ThemeProvider } from 'next-themes'

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <ThemeProvider attribute="data-theme" defaultTheme="system" enableSystem disableTransitionOnChange>
      {children}
    </ThemeProvider>
  )
}
```

`app/layout.tsx` :

```tsx
import { Providers } from './providers'
import './globals.css'

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body className="bg-background text-foreground">
        <Providers>{children}</Providers>
      </body>
    </html>
  )
}
```

ALWAYS add `suppressHydrationWarning` on `<html>` to avoid the React 18+ hydration mismatch warning that `next-themes` triggers (issue #5552 in shadcn-ui/ui tracker confirms 99 reactions).

### Theme toggle component

```tsx
'use client'
import { useTheme } from 'next-themes'

export function ThemeToggle() {
  const { theme, setTheme } = useTheme()
  return (
    <div className="flex gap-2">
      <button onClick={() => setTheme('light')} className="rounded-md border px-3 py-1 text-sm">Light</button>
      <button onClick={() => setTheme('dark')} className="rounded-md border px-3 py-1 text-sm">Dark</button>
      <button onClick={() => setTheme('system')} className="rounded-md border px-3 py-1 text-sm">System</button>
    </div>
  )
}
```

`next-themes` sets `<html data-theme="dark">` (or `light`) and handles the inline-script-in-head pattern internally; no flash.

## 4. Vite + React (without next-themes)

`index.html` :

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>Vite app</title>
  <script>
    document.documentElement.classList.toggle(
      "dark",
      localStorage.theme === "dark" ||
        (!("theme" in localStorage) &&
          window.matchMedia("(prefers-color-scheme: dark)").matches)
    );
  </script>
</head>
<body>
  <div id="root"></div>
  <script type="module" src="/src/main.tsx"></script>
</body>
</html>
```

`src/styles.css` (v4) :

```css
@import "tailwindcss";
@custom-variant dark (&:where(.dark, .dark *));
```

`src/ThemeToggle.tsx` :

```tsx
import { useEffect, useState } from 'react'

type Mode = 'light' | 'dark' | 'system'

function getInitial(): Mode {
  if (typeof localStorage === 'undefined') return 'system'
  const v = localStorage.theme
  return v === 'light' || v === 'dark' ? v : 'system'
}

function apply(mode: Mode) {
  if (mode === 'system') {
    localStorage.removeItem('theme')
    const osDark = window.matchMedia('(prefers-color-scheme: dark)').matches
    document.documentElement.classList.toggle('dark', osDark)
  } else {
    localStorage.theme = mode
    document.documentElement.classList.toggle('dark', mode === 'dark')
  }
}

export function ThemeToggle() {
  const [mode, setMode] = useState<Mode>(getInitial)

  useEffect(() => {
    apply(mode)
  }, [mode])

  return (
    <div className="flex gap-2">
      <button onClick={() => setMode('light')} className="rounded-md border px-3 py-1 text-sm">Light</button>
      <button onClick={() => setMode('dark')} className="rounded-md border px-3 py-1 text-sm">Dark</button>
      <button onClick={() => setMode('system')} className="rounded-md border px-3 py-1 text-sm">System</button>
    </div>
  )
}
```

The inline script in `index.html` runs BEFORE React boots, eliminating the flash. The React component owns the user-action handling AFTER hydration.

## 5. The anti-toggle pattern (light wins)

When the default should be dark and the user can override to light :

### v4

```css
@import "tailwindcss";
@custom-variant dark (&:not(.light *));
```

### v3

```js
module.exports = {
  darkMode: ['variant', '&:not(.light *)'],
}
```

Markup :

```html
<!-- Default : dark mode applies -->
<html>
  <body class="bg-slate-900 text-white dark:bg-slate-950 dark:text-slate-100">
    ...
  </body>
</html>

<!-- Light override : add .light to the toggle target -->
<html class="light">
  <body class="bg-slate-900 text-white dark:bg-slate-950 dark:text-slate-100">
    ...
  </body>
</html>
```

`dark:` utilities apply UNLESS an ancestor has class `light`. Useful for portfolios / showcases with a dark-by-default aesthetic.

## 6. Multi-trigger selector (media OR class)

ALWAYS use this when you want OS preference to be respected, but want a user toggle to override.

### v4

```css
@import "tailwindcss";
@custom-variant dark (
  &:where(.dark, .dark *),
  &:where([data-theme=dark], [data-theme=dark] *),
  @media (prefers-color-scheme: dark) { &:not(.light, .light *) }
);
```

### v3

```js
module.exports = {
  darkMode: ['variant', [
    '@media (prefers-color-scheme: dark) { &:not(.light *) }',
    '&:is(.dark *)',
  ]],
}
```

The user adds `.light` to `<html>` to override OS dark to light, `.dark` to force dark regardless of OS, and removes both to inherit OS preference.

## 7. Server-side rendering (Astro)

`src/layouts/Layout.astro` :

```astro
---
const { title } = Astro.props
---
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>{title}</title>
  <script is:inline>
    document.documentElement.classList.toggle(
      "dark",
      localStorage.theme === "dark" ||
        (!("theme" in localStorage) &&
          window.matchMedia("(prefers-color-scheme: dark)").matches)
    );
  </script>
  <link rel="stylesheet" href="/src/styles.css" />
</head>
<body class="bg-background text-foreground">
  <slot />
</body>
</html>
```

ALWAYS use `is:inline` to keep the script outside Astro's bundler, so it runs synchronously in `<head>` before any other resource loads.

## 8. Token-swap-only (no per-utility `dark:` variants)

```html
<html class="dark">
  <body class="bg-background text-foreground">
    <main class="p-8">
      <article class="bg-card text-card-foreground rounded-lg shadow-md p-6">
        <h2 class="text-2xl font-semibold">Token-swap example</h2>
        <p class="text-muted-foreground mt-2">No dark: variants needed. The CSS variable values flip when .dark is on html.</p>
        <button class="bg-primary text-primary-foreground rounded-md px-4 py-2 mt-4">
          Save
        </button>
      </article>
    </main>
  </body>
</html>
```

Every utility resolves through a CSS variable. When `.dark` is on `<html>`, ALL variables update at once. No per-utility `dark:` variant in markup.

ALWAYS pair this pattern with `--color-*` variables in `@theme` (v4) or `extend.colors` with `<alpha-value>` placeholder (v3).
