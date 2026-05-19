# Anti-Patterns : Tailwind Dark Mode

Six canonical failures, each with symptom, root cause, and fix.

## 1. v3 `darkMode: 'class'` left in a v4 project

### Symptom

A v4 migration leaves `tailwind.config.js` with `darkMode: 'class'`. The user toggles `<html class="dark">` in the browser. NOTHING happens. All `dark:` utilities continue to follow OS preference.

### Root cause

v4 removed the `darkMode` config field entirely. Even if `@config "./tailwind.config.js"` is used to load a legacy JS config in v4, the `darkMode` key is **silently ignored**. The v4 way to override is via the `@custom-variant dark` directive in the main CSS file.

### Fix

Remove `darkMode` from the JS config (it does nothing in v4). Add to the main CSS file :

```css
@import "tailwindcss";
@custom-variant dark (&:where(.dark, .dark *));
```

ALWAYS run `npx @tailwindcss/upgrade` to do this automatically. The tool converts the JS config field to the CSS directive.

## 2. Theme flashes wrong colour on first paint (FOUC)

### Symptom

Page loads with light theme briefly, then snaps to dark. Users report a visible flicker. In Lighthouse this triggers a Largest Contentful Paint warning because the painted-then-changed content counts as a layout shift.

### Root cause

The toggle script runs AFTER the first paint. Sequence of events :
1. Browser parses HTML.
2. Browser parses linked CSS, applies default (light) styling.
3. Browser paints first frame in LIGHT.
4. JS bundle loads (deferred).
5. React / Vue / Svelte mount, `useEffect` / `onMount` runs.
6. Script adds `class="dark"` to `<html>`.
7. Browser repaints in DARK.

Steps 3 and 7 produce the visible flash.

### Fix

ALWAYS place an inline script in `<head>` BEFORE any stylesheet link or framework script :

```html
<head>
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
```

The script runs synchronously before CSS parses, so step 3 paints in the correct theme.

For Next.js, ALWAYS use `next-themes` which implements this inline-script pattern internally and handles SSR correctly. ALWAYS pair with `suppressHydrationWarning` on `<html>`.

NEVER place the toggle inside `useEffect`, `onMount`, or any post-mount lifecycle hook.

## 3. Class toggled on `<body>` instead of `<html>`

### Symptom

`<body class="dark">` is set; some elements react to `dark:` variants, others do not. Specifically, portaled elements (Radix popovers, headlessui dialogs, react-portal mounts at `document.body`) are sometimes outside `<body>` and never get the dark styling.

### Root cause

The default selector `:where(.dark, .dark *)` matches any element with class `dark` OR any descendant. A portal mount at `document.body` is a sibling of your `<div id="root">`, not a descendant. If the toggle is on `<body>`, the portal is a descendant and DOES get styled. If the toggle is on `<html>`, the portal is still a descendant of `<html>` and DOES get styled. If the toggle is on `<div id="root">`, the portal is OUTSIDE that subtree and DOES NOT get styled.

The portal failure mode is real for any toggle target NOT on `<html>` or `<body>`. `<html>` is the safest because it covers everything including `<body>` itself.

### Fix

ALWAYS toggle on `document.documentElement` (which is `<html>`) :

```javascript
document.documentElement.classList.toggle("dark", isDark)
```

NEVER on `document.body` :

```javascript
// WRONG
document.body.classList.toggle("dark", isDark)
```

## 4. Checking OS preference BEFORE localStorage in the toggle

### Symptom

User clicks "Dark" in the toggle UI. The page goes dark. User refreshes. The page is light again because their OS is set to light.

### Root cause

The toggle script checks `prefers-color-scheme` FIRST and uses that as the source of truth :

```javascript
// WRONG
const osDark = window.matchMedia("(prefers-color-scheme: dark)").matches
const isDark = osDark || localStorage.theme === "dark"
document.documentElement.classList.toggle("dark", isDark)
```

This means the OS preference (light) wins over the user's explicit choice (dark) stored in localStorage.

### Fix

ALWAYS check `localStorage` FIRST :

```javascript
// CORRECT
document.documentElement.classList.toggle(
  "dark",
  localStorage.theme === "dark" ||
    (!("theme" in localStorage) &&
      window.matchMedia("(prefers-color-scheme: dark)").matches)
)
```

The logic reads : "Dark if the user explicitly chose dark, OR if there is no explicit choice AND the OS prefers dark." User choice always wins.

## 5. Writing `dark:` variants when a token-swap design system exists

### Symptom

A codebase has 200+ `dark:bg-slate-900`, `dark:text-white`, `dark:border-slate-700` utility pairs. Designers change the dark-mode background colour; the change requires editing 200+ places.

### Root cause

The team did NOT set up theme-token variables. Every dark colour is hard-coded per element. Tailwind has no way to swap them in bulk.

### Fix

ALWAYS set up theme-token variables when a design-system exists. Then markup uses `bg-background`, `text-foreground` (no `dark:` variant), and the variable values flip when `.dark` is on `<html>`. One change in the CSS variable swap affects every consuming element.

See pattern 4 in SKILL.md and section 4 of references/methods.md for the full recipe.

Per-utility `dark:` variants remain valid for one-off overrides. NEVER use them as the primary mechanism in a token-driven project.

## 6. Mixing v3 `darkMode` config and v4 `@custom-variant dark` in one project

### Symptom

After running `npx @tailwindcss/upgrade`, the project has both :
- `tailwind.config.js` with `darkMode: 'selector'` (untouched because the tool moved it to CSS)
- `app.css` with `@custom-variant dark (&:where(.dark, .dark *));` (added by the tool)

The build succeeds. Dark mode works. Six months later a developer adds `darkMode: 'media'` to the JS config expecting it to switch behaviour. Nothing changes.

### Root cause

In v4, the JS-config `darkMode` field is silently ignored. The CSS `@custom-variant dark` directive is the sole source of truth. The lingering JS field misleads future maintainers into believing it has any effect.

### Fix

ALWAYS delete the `darkMode` field from `tailwind.config.js` after the v4 upgrade. The CSS directive owns the strategy.

Better : in v4, do not load a JS config at all unless legacy plugins absolutely require it. Move all configuration to `@theme`, `@custom-variant`, `@plugin`, `@source` directives in the main CSS file.

See [tailwind-impl-config-v4] and [tailwind-impl-migration-v3-v4].

## 7. Forgetting `suppressHydrationWarning` with React + `next-themes`

### Symptom

Console warning :

```
Warning: Prop `className` did not match. Server: "" Client: "dark"
```

In strict mode, React refuses to hydrate cleanly. Some elements briefly render with mismatched styles.

### Root cause

`next-themes` and similar libraries set the dark-mode class on `<html>` via an inline script BEFORE React hydrates. The server-rendered HTML (no class) and the client-side HTML (class added by script) differ. React's hydration check flags the mismatch.

### Fix

ALWAYS add `suppressHydrationWarning` to the `<html>` element when using `next-themes` :

```tsx
// Next.js app/layout.tsx
export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  )
}
```

`suppressHydrationWarning` is a React hint that says "I know this specific element's attributes will differ between server and client; do not warn about it." It is the canonical pattern for this exact scenario per shadcn-ui/ui issue #5552 (99 reactions) and the next-themes README.

NEVER add `suppressHydrationWarning` elsewhere as a way to silence other warnings; it is a narrow tool.

## Sources

- https://tailwindcss.com/docs/dark-mode (v4 directive, JS toggle pattern)
- https://v3.tailwindcss.com/docs/dark-mode (v3 config schema, deprecation path)
- https://tailwindcss.com/docs/upgrade-guide (v3-to-v4 darkMode removal)
- https://github.com/pacocoursey/next-themes (suppressHydrationWarning pattern)
- https://github.com/shadcn-ui/ui/issues/5552 (Theme Provider hydration error in Next.js 15)

Verified 2026-05-19.
