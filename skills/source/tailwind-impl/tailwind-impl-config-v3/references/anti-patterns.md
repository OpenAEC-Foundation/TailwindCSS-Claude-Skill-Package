# tailwind-impl-config-v3 : Anti-Patterns

Common `tailwind.config.js` mistakes with WHY they fail and the fix.

## AP-1 : `theme: { colors: {...} }` Without `extend`

**Symptom** : `bg-red-500`, `text-slate-900`, every default colour utility stops working.

```js
/* WRONG */
theme: {
  colors: { brand: "#3b82f6" },
}
```

**Why** : `theme.<namespace>` REPLACES the entire default namespace. The default colour palette is wiped.

**Fix** :

```js
theme: {
  extend: {
    colors: { brand: "#3b82f6" },   // keeps defaults, adds brand
  },
}
```

## AP-2 : Content Glob Leaking Into `node_modules`

**Symptom** : Build takes minutes. Editor lags. Out-of-memory errors.

```js
/* WRONG */
content: ["./**/*.{js,html}"]
```

**Why** : Glob walks every nested directory including `node_modules`, scanning thousands of files.

**Fix** :

```js
/* RIGHT */
content: ["./src/**/*.{js,ts,jsx,tsx,html}", "./index.html"]
```

ALWAYS root the glob at a specific dir (`./src`, `./app`, `./pages`). NEVER use `./**/*` at the repo root.

## AP-3 : `darkMode: 'class'` Without Class on `<html>`

**Symptom** : Adding `dark:bg-black` to a button does nothing. The dark variant compiles but never matches.

```js
/* config */
darkMode: 'class',
```

```html
<!-- WRONG : no class anywhere -->
<html>
  <body>
    <button class="bg-white dark:bg-black">Hi</button>
  </body>
</html>
```

**Why** : `darkMode: 'class'` requires a `.dark` ancestor for the `dark:` variant to fire. Without it the variant never matches.

**Fix** :

```html
<html class="dark">
  ...
</html>
```

Or a runtime toggle via JS.

## AP-4 : Next.js Catch-All Glob With Literal Brackets

**Symptom** : Classes inside `app/[...slug]/page.tsx` never get picked up.

```js
/* WRONG */
content: ["./app/[...slug]/**/*.{js,ts,jsx,tsx}"]
```

**Why** : The glob engine treats `[...slug]` as a character class (a set of characters), not a literal directory name.

**Fix** :

```js
/* RIGHT : let the broader glob match the dynamic route */
content: ["./app/**/*.{js,ts,jsx,tsx}"]
```

Or escape the brackets explicitly if you must scope precisely :

```js
content: ["./app/\\[...slug\\]/**/*.{js,ts,jsx,tsx}"]
```

## AP-5 : Mixing CommonJS and ESM in Same Config

**Symptom** : Node throws on load OR the config is silently ignored.

```js
/* WRONG */
import typography from "@tailwindcss/typography";

module.exports = {
  plugins: [typography],
};
```

**Why** : `import` is ESM, `module.exports` is CJS. Node refuses to mix them in the same file.

**Fix** : Pick one and stick to it.

```js
/* RIGHT : CJS */
const typography = require("@tailwindcss/typography");

module.exports = {
  plugins: [typography],
};
```

```ts
/* RIGHT : ESM (file extension .mjs OR package "type": "module") */
import type { Config } from "tailwindcss";
import typography from "@tailwindcss/typography";

export default { plugins: [typography] } satisfies Config;
```

## AP-6 : `safelist` Used With v4 Migration Coming

**Symptom** : v4 build silently drops safelisted classes ; production CSS is missing them.

```js
/* OK in v3 but BREAKS on v4 migration */
safelist: ["bg-red-500", { pattern: /bg-(red|blue)-(100|500)/ }],
```

**Why** : `safelist` is REMOVED in v4. Its config-time evaluation has no CSS-side equivalent.

**Fix for v4 migration** : Move to `@source inline(...)` in CSS :

```css
/* main.css */
@import "tailwindcss";
@source inline("bg-red-500");
@source inline("bg-{red,blue}-{100,500}");
```

Plan the migration in advance : delete `safelist` from v3 config in the same commit that adds `@source inline` to CSS.

## AP-7 : Disabling Core Plugins Before v4 Migration

```js
/* OK in v3 but DEAD on v4 migration */
corePlugins: { float: false }
```

**Why** : `corePlugins` is REMOVED in v4. There is NO replacement. v4 cannot disable specific utility families.

**Fix** :

1. Audit which utilities you're disabling and WHY.
2. Replace the rationale (lint rule, stylelint, CI check) with a non-Tailwind enforcement mechanism.
3. Remove `corePlugins` before migrating to v4.

## AP-8 : `separator: '_'` for "Cleaner" Class Names

**Symptom** : Project works in v3, breaks on v4 migration ; all `_` separators silently revert to `:`.

```js
/* WRONG (and gone in v4) */
separator: "_",
```

**Why** : `separator` is REMOVED in v4. v4 fixes the separator at `:`.

**Fix** : Use the default `:`. Refactor any tooling that expects `_`.

## AP-9 : Using `theme()` Inside Arbitrary Values at Runtime

**Symptom** : `style={{ padding: theme('spacing.4') }}` throws at runtime because `theme()` is a Tailwind config-time function, not a JS runtime API.

**Why** : `theme()` only works inside the Tailwind config file (passed to plugins). At runtime it does not exist.

**Fix** : Use CSS variables OR resolved values :

```js
// Pre-resolve at build : import from config or use resolveConfig()
import resolveConfig from "tailwindcss/resolveConfig";
import tailwindConfig from "../../tailwind.config.js";

const fullConfig = resolveConfig(tailwindConfig);
const spacing4 = fullConfig.theme.spacing[4];
```

Or read CSS variables emitted by Tailwind into root.

## AP-10 : Forgetting `relative: true` for Monorepo Subdirs

**Symptom** : Classes in `../packages/ui` are scanned but referenced paths in built CSS are wrong (resolve to `node_modules` instead of source).

```js
/* sometimes WRONG in monorepos */
content: ["../packages/ui/src/**/*.tsx"]
```

**Fix** :

```js
content: {
  relative: true,
  files: ["../packages/ui/src/**/*.tsx"],
}
```

`relative: true` makes globs resolve against the config file location, not CWD.

## AP-11 : Using `prefix: 'tw-'` Without Coordinating Editor Tools

**Symptom** : IntelliSense breaks (extension expects unprefixed classes), prettier-plugin-tailwindcss does not sort, eslint-plugin-tailwindcss flags every utility.

**Fix** : Configure every editor tool to honour the prefix :

```json
{
  "tailwindCSS.classAttributes": ["class", "className", "ngClass"],
  "tailwindCSS.experimental.classRegex": [["cva\\(([^)]*)\\)", "[\"'`]([^\"'`]*).*?[\"'`]"]],
  "tailwindCSS.includeLanguages": { "plaintext": "html" }
}
```

For prettier : `prefix: "tw-"` is auto-detected from `tailwind.config.js` if `prettier-plugin-tailwindcss` is installed.

## Verified Sources

- https://v3.tailwindcss.com/docs/content-configuration
- https://v3.tailwindcss.com/docs/configuration
- https://v3.tailwindcss.com/docs/dark-mode
- https://github.com/tailwindlabs/tailwindcss/issues/18136 (dynamic class names)

Last verified : 2026-05-19.
