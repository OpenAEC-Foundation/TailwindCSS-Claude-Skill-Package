# Lessons Learned

Observations and findings captured during skill package development.

---

## L-001 : v4 removes the three @tailwind directives entirely

The familiar v3 boilerplate of `@tailwind base; @tailwind components; @tailwind utilities;` is **gone** in v4. A single `@import "tailwindcss";` replaces all three. This impacts every install skill (`impl-build-*`) and the v3-to-v4 migration skill : every example must be split into a v3 column and a v4 column, never a single shared snippet.

Source : https://tailwindcss.com/blog/tailwindcss-v4 and https://tailwindcss.com/docs/upgrade-guide

## L-002 : `@apply` silently breaks in Vue/Svelte/CSS-modules `<style>` blocks in v4

When v4 compiles a scoped stylesheet in isolation (Vue SFC `<style>`, Svelte component `<style>`, CSS modules), the theme context is **not implicitly available**. `@apply text-2xl` fails with "Cannot apply unknown utility class". The v4-only fix is `@reference "../app.css";` (or `@reference "tailwindcss";`) as the first line of the scoped block. This is a brand-new directive with no v3 equivalent, and it MUST be documented prominently in any `impl-build-vue`, `impl-build-svelte`, or `impl-apply-directive` skill.

Source : https://tailwindcss.com/docs/functions-and-directives and https://github.com/tailwindlabs/tailwindcss/issues/16346

## L-003 : v4 removes `corePlugins`, `safelist`, and `separator` config options outright

These three v3 config options are simply gone in v4 (silently ignored if present in a `@config "./tailwind.config.js"` file). `safelist` has a CSS-native replacement (`@source inline("classnames")` with brace expansion). `corePlugins` and `separator` have NO replacement at all. The `errors-v4-migration` skill must explicitly warn users that disabling specific utilities (a common v3 customisation) is no longer supported : the closest workaround is `@source not "path"` to scope content scanning.

Source : https://tailwindcss.com/docs/upgrade-guide

## L-004 : v4 changes variant stacking order from right-to-left to left-to-right

A v3 selector like `first:*:pt-0` (read right-to-left : direct children, first one) compiles in v4 to `*:first:pt-0` (read left-to-right). This is a silent behavioural change : the upgrade tool catches common cases but bespoke arbitrary-variant stacks (e.g. `[&>div]:hover:bg-red-500` patterns combined with structural selectors) need manual review. Document this loudly in `syntax-variants` and `errors-v4-migration`.

Source : https://tailwindcss.com/docs/upgrade-guide section 14

## L-005 : Default border colour changed from `gray-200` to `currentColor`, and ring width changed from 3px to 1px

Two of the most invasive v4 visual breakages affect *every* existing project. The fix is either explicit colour utilities on every bordered/ringed element, OR a one-time `@layer base` shim that restores the v3 defaults. The migration skill should ship both options : the shim for quick visual parity, the explicit-utility approach for clean long-term code. The same applies to `shadow`, `blur`, `rounded`, `drop-shadow`, and `backdrop-blur` scale renames where everything shifted by one size step (`shadow-sm` is now smaller than v3's `shadow-sm` because `shadow-xs` was inserted below it).

Source : https://tailwindcss.com/docs/upgrade-guide sections 5 and 6

## L-006 : Dynamic class names from server-side data is the single most common anti-pattern, with a v4-specific safelist solution

The "I built a `bg-${color}-500` template helper and now nothing works in production" pattern is the most reported issue in the Tailwind tracker (issue 18136 is the canonical v4 instance, and the same class of problem dominates v3 issues too). The v4-specific solution `@source inline("{hover:,}bg-{red,blue,green}-{50,{100..900..100},950}")` is powerful but undocumented in most secondary sources. Every `errors-purge-issues` and `errors-build-failures` skill MUST cover : (a) why detection fails (plain-text token scan), (b) the static-map fix (preferred), (c) the inline-style fallback for truly runtime values, and (d) the `@source inline()` safelist with brace expansion as the v4 escape hatch. The v3 equivalent is the `safelist: [...]` config option, which is removed in v4.

Source : https://github.com/tailwindlabs/tailwindcss/issues/18136 and https://tailwindcss.com/docs/detecting-classes-in-source-files
