---
name: tailwind-impl-tailwind-merge
description: >
  Use when composing Tailwind class strings dynamically across React,
  Vue, Svelte, Solid, or any framework where a parent component passes
  override classes to a child via a className prop, and the child
  combines them with its own defaults. Solves the duplicate-utility
  problem (twMerge('p-2 p-4') -> 'p-4') so the LAST value wins
  deterministically instead of the cascade emitting both and the
  outcome depending on stylesheet ordering. Prevents the
  ineffective-override trap (passing 'p-4' to a child whose default is
  'p-2 text-blue-500' leaves BOTH padding classes in the markup, and
  whichever the bundler ships later wins), the stale-cache trap
  (calling extendTailwindMerge inside a render path on every render
  rebuilds the big data structure), the wrong-package-version trap
  (tailwind-merge v2.x is for Tailwind v3, v3.x is for Tailwind v4),
  and the prefix-mismatch trap (custom Tailwind prefix tw- requires
  configuring tailwind-merge with that prefix or merges silently
  return wrong output).
  Covers twMerge / twJoin / extendTailwindMerge / createTailwindMerge /
  getDefaultConfig / mergeConfigs / fromTheme / validators, the
  shadcn/ui cn helper pattern (clsx + twMerge), class-variance-authority
  (cva) integration with base + variants + compoundVariants +
  defaultVariants, the VariantProps TypeScript type, custom utility
  registration for design-system tokens, prefix support, cacheSize
  tuning, when to use twMerge vs twJoin vs plain string concatenation.
  Keywords: tailwind-merge, twMerge, twJoin, clsx tailwind, cn helper,
  shadcn cn, class-variance-authority, cva, VariantProps, classnames,
  extendTailwindMerge, createTailwindMerge, getDefaultConfig,
  mergeConfigs, fromTheme, validators, conflict resolution, last wins
  tailwind, override className prop, conditional class names react,
  cacheSize, prefix tailwind merge, custom utility merge, shadcn ui
  button, my padding class not applied, tailwind classes both appearing,
  override className not working, how to merge tailwind classes, react
  className prop, dynamic tailwind classes, cva button pattern,
  variant pattern react, install tailwind-merge, tailwind-merge v2,
  tailwind-merge v3, tailwind-merge for tailwind v4, tailwind-merge for
  tailwind v3, memoize class merge, render path performance
license: MIT
compatibility: "Designed for Claude Code. Requires Tailwind CSS v3.4 or v4.0+."
metadata:
  author: OpenAEC-Foundation
  version: "1.0"
---

# tailwind-merge : Deterministic Class Override

`tailwind-merge` is a runtime library that takes any number of
Tailwind class strings and returns ONE string where conflicting
utilities are resolved with a LAST-WINS rule.

It is the foundation of the shadcn/ui `cn()` helper, the cva-based
component variant pattern, and any React/Vue/Svelte component that
accepts a `className` prop and combines it with internal defaults.

Companion skills :

- `tailwind-impl-config-v3` and `tailwind-impl-config-v4` : the
  Tailwind side of the equation.
- `tailwind-impl-plugins-official` : how the official plugins emit
  class groups that tailwind-merge needs to know about.
- shadcn-ui skill package : the upstream design system that
  popularised the `cn` helper.

## Quick Reference : The Problem

```jsx
// Component default
function Card({ className }) {
  return <div className={`p-2 bg-white ${className}`} />;
}

// Caller passes override
<Card className="p-4" />
```

Output markup contains `p-2 bg-white p-4`. The compiled CSS contains
BOTH `.p-2` and `.p-4`. Whichever appears later in the stylesheet
wins. With JIT this is non-deterministic across builds.

## Quick Reference : The Fix

```jsx
import { twMerge } from "tailwind-merge";

function Card({ className }) {
  return <div className={twMerge("p-2 bg-white", className)} />;
}

<Card className="p-4" />
```

Output markup contains `bg-white p-4`. Deterministic. `p-2` is removed
because it conflicts with `p-4` in the same `padding` class group.

## Quick Reference : Install

| Tailwind version | tailwind-merge version | npm install |
|------------------|------------------------|-------------|
| v3.x | 2.x (latest 2.6.0) | `npm install tailwind-merge@2` |
| v4.x | 3.x (current) | `npm install tailwind-merge@3` |

ALWAYS pin to the major matching your Tailwind version. v2.x knows
about v3-era class groups ; v3.x knows about v4-era class groups. A
mismatch silently produces wrong merges for utilities the library
does not recognise.

## Quick Reference : The shadcn cn Helper

```ts
// src/lib/utils.ts
import { type ClassValue, clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

Used everywhere :

```tsx
<button className={cn("px-3 py-1 rounded", isPrimary && "bg-blue-600", className)}>
```

`clsx` resolves conditionals + arrays + objects into a single string ;
`twMerge` then resolves Tailwind-level conflicts.

## Decision Tree : Do I Need tailwind-merge

```
Does the class list ever come from props or user input ?
  ├── NO (always static, hard-coded in JSX) -> NO twMerge needed
  │     Example: <div className="p-4 bg-blue-500">  -> just write it
  │
  └── YES (className prop, variant logic, theme switching, etc.)
        -> YES, use twMerge OR cn(clsx(...))

Are the classes conditional but never conflict ?
  ├── YES (e.g. show/hide a class only) -> twJoin or clsx are enough
  │     Example: clsx("base", isActive && "bg-blue-500")
  │
  └── YES (different sizes/variants override each other)
        -> twMerge IS required
```

## Decision Tree : twMerge vs twJoin vs clsx vs cn

| Tool | Conflict resolution | Conditionals | Bundle |
|------|---------------------|--------------|--------|
| `twMerge` | yes (LAST WINS) | only strings | larger (full data structure) |
| `twJoin` | no | strings + arrays + nullables | smaller |
| `clsx` | no | strings + arrays + objects + nullables | smallest |
| `cn` (shadcn) | yes | everything clsx accepts | largest |

ALWAYS pick the lightest tool that does the job :

```
Static classes only                          -> just write the string
Conditional, no conflict possible            -> clsx OR twJoin
Conditional, conflicts possible              -> cn(clsx(...))
External override via className prop         -> twMerge OR cn
```

## API : Primary Functions

### `twMerge(...classLists)`

```ts
function twMerge(
  ...classLists: Array<string | undefined | null | false | 0 | ClassLists>
): string;
```

Accepts variadic strings and nullables. Returns one merged string.
LAST conflicting class wins. Falsy values are ignored. Nested arrays
are flattened.

### `twJoin(...classLists)`

```ts
function twJoin(
  ...classLists: Array<string | undefined | null | false | 0 | ClassLists>
): string;
```

Same signature, NO conflict resolution. Just concatenation with
whitespace and falsy filtering. Faster + smaller than `twMerge`. Use
when you know nothing will conflict.

### `extendTailwindMerge(configExtension, ...createConfig)`

Returns a CUSTOM `twMerge` function that knows about additional class
groups, theme scales, prefix, or conflict rules. ALWAYS call this
ONCE at module scope, NEVER inside a render path.

```ts
import { extendTailwindMerge } from "tailwind-merge";

export const twMerge = extendTailwindMerge({
  prefix: "tw-",
  extend: {
    classGroups: {
      "btn-size": ["btn-sm", "btn-md", "btn-lg"],
    },
  },
});
```

### `createTailwindMerge(...createConfig)`

Returns a `twMerge` that uses ONLY your config (no inheritance from
defaults). Used when you have NO Tailwind core classes at all and a
fully bespoke utility set. Rare.

### `getDefaultConfig()`

Returns the default config object. Use as a starting point for
`mergeConfigs` or to inspect what tailwind-merge knows about.

### `mergeConfigs(...configs)`

Combines multiple configs. Useful when composing extensions from
several design-system packages.

### `fromTheme(key)`

Returns a "theme getter" function used inside `classGroups` to
reference theme scales. Lets `extendTailwindMerge` resolve dynamic
values from the Tailwind theme.

### `validators`

A module of helper validators for class-group patterns. Examples :
`isNumber`, `isLength`, `isArbitraryValue`, `isArbitraryLength`,
`isPercent`, `isTshirtSize`. Pass into `classGroups` to accept any
matching arbitrary value.

## CRITICAL : Performance Rule

`extendTailwindMerge` builds a large internal data structure. Calling
it on every render costs milliseconds per call AND defeats the
internal LRU cache.

### Wrong (defeats memoisation)

```tsx
function Button({ className }) {
  const twMerge = extendTailwindMerge({ prefix: "tw-" });   // rebuilt every render
  return <button className={twMerge("px-3 py-1", className)} />;
}
```

### Right

```tsx
// src/lib/tw-merge.ts (module scope)
import { extendTailwindMerge } from "tailwind-merge";
export const twMerge = extendTailwindMerge({ prefix: "tw-" });

// Component
import { twMerge } from "@/lib/tw-merge";
function Button({ className }) {
  return <button className={twMerge("px-3 py-1", className)} />;
}
```

The default cache is 500 entries (LRU). Tune via `cacheSize` if
profiling shows lots of cache misses on a hot path.

## CRITICAL : Prefix Configuration

If your Tailwind config has a `prefix` (e.g. `tw-`), tailwind-merge
MUST be told :

```ts
// v3 Tailwind config has prefix: 'tw-'
export const twMerge = extendTailwindMerge({ prefix: "tw-" });
```

Without this, `tw-p-2 tw-p-4` is NOT recognised as a padding conflict
and both classes survive.

## class-variance-authority (cva) Pairing

`cva` defines variant-based component class strings as data. Combine
with `twMerge` so consumer overrides still win.

### Install

```bash
npm install class-variance-authority clsx tailwind-merge
```

### Define a button variant set

```ts
// src/components/button.tsx
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/lib/utils";

const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-md font-medium transition-colors",
  {
    variants: {
      variant: {
        default: "bg-blue-600 text-white hover:bg-blue-700",
        outline: "border border-zinc-300 text-zinc-900 hover:bg-zinc-100",
        ghost: "text-zinc-900 hover:bg-zinc-100",
      },
      size: {
        sm: "h-8 px-3 text-sm",
        md: "h-10 px-4 text-base",
        lg: "h-12 px-6 text-lg",
      },
    },
    defaultVariants: { variant: "default", size: "md" },
  }
);

export type ButtonProps = VariantProps<typeof buttonVariants> & {
  className?: string;
  children: React.ReactNode;
};

export function Button({ className, variant, size, children, ...rest }: ButtonProps) {
  return (
    <button className={cn(buttonVariants({ variant, size }), className)} {...rest}>
      {children}
    </button>
  );
}
```

### Consumer override

```tsx
<Button variant="outline" size="lg" className="rounded-full px-10">
  Click me
</Button>
```

`rounded-md` from the base is replaced by `rounded-full` because they
conflict in the `border-radius` group. `px-6` from `size: lg` is
replaced by `px-10`. Other classes survive.

## CRITICAL : When NOT to Use tailwind-merge

```
Static utility list with no possible conflicts ?
  -> Don't use it. Just write the string.

A single condition that adds or removes one class ?
  -> Use clsx or twJoin. Lighter.

Server-rendered output with NO client interaction ?
  -> Consider precomputing at build time, not at request time.

Email template renderer (HTML to many clients) ?
  -> tailwind-merge runs in the renderer fine. Tailwind itself does
     not (no JS in email clients).
```

Every `twMerge` call has a runtime cost (cache lookup or full parse).
Avoid the hot-path tax when not needed.

## Cross-Link : shadcn/ui Pattern

The shadcn/ui design system standardises ALL components on the
`cn` helper that wraps `twMerge` and `clsx`. If you adopt shadcn :

- `src/lib/utils.ts` exposes `cn`.
- Every component uses `cn` for its root className.
- cva is used for variant-based components (Button, Badge, Alert).
- See the companion shadcn-ui Claude Skill Package for the full
  component set and the install scripts.

## What This Skill Does NOT Cover

- Writing your own Tailwind utility classes (use
  `tailwind-impl-config-v3` or `tailwind-impl-config-v4`).
- The Tailwind cascade itself (use `tailwind-syntax-cascade`).
- Tailwind plugin authorship (use `tailwind-impl-plugins-custom`).
- shadcn/ui component recipes (use the shadcn-ui skill package).

## References

- `references/methods.md` : full API surface, all functions, all
  validators, all options.
- `references/examples.md` : shadcn cn helper, cva button, custom
  prefix, design-system variant patterns.
- `references/anti-patterns.md` : in-render extendTailwindMerge,
  version mismatch, prefix-mismatch, twMerge on static classes,
  unnecessary cn wrapping.

## Official Source Links

- Repo : https://github.com/dcastil/tailwind-merge
- API reference v3 : https://github.com/dcastil/tailwind-merge/blob/v3/docs/api-reference.md
- Configuration v3 : https://github.com/dcastil/tailwind-merge/blob/v3/docs/configuration.md
- class-variance-authority : https://github.com/joe-bell/cva
- cva docs : https://cva.style
