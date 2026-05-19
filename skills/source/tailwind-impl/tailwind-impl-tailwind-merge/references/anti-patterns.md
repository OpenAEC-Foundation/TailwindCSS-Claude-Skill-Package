# tailwind-impl-tailwind-merge : Anti-Patterns

The traps that break tailwind-merge integration.

## Anti-Pattern 1 : Wrong major version for the Tailwind version

### Symptom

`twMerge("p-2", "p-4")` returns `"p-2 p-4"` instead of `"p-4"`.
Or returns something subtly different on certain classes.

### Wrong (Tailwind v4 project)

```json
{
  "dependencies": {
    "tailwindcss": "^4.0.0",
    "tailwind-merge": "^2.6.0"
  }
}
```

### Right (Tailwind v4 project)

```bash
npm install tailwind-merge@3
```

### Right (Tailwind v3 project)

```bash
npm install tailwind-merge@2
```

### Root cause

tailwind-merge ships its OWN copy of class-group knowledge. v2.x
knows about v3-era class groups (e.g. `aspect-w-*` plugin) ; v3.x
knows about v4-era class groups (e.g. native container queries).
Mixing versions silently produces wrong merges on the unknown classes.

ALWAYS pin to the major version matching your Tailwind major.

## Anti-Pattern 2 : Calling extendTailwindMerge inside a render path

### Symptom

Profiler shows excessive CPU on every render of a component. App is
sluggish. Memory grows over time.

### Wrong

```tsx
function Button({ className }) {
  const twMerge = extendTailwindMerge({ prefix: "tw-" });   // rebuilds every render
  return <button className={twMerge("tw-px-3 tw-py-1", className)} />;
}
```

### Right

```ts
// src/lib/tw.ts (module scope, ONCE)
import { extendTailwindMerge } from "tailwind-merge";
export const twMerge = extendTailwindMerge({ prefix: "tw-" });
```

```tsx
import { twMerge } from "@/lib/tw";

function Button({ className }) {
  return <button className={twMerge("tw-px-3 tw-py-1", className)} />;
}
```

### Root cause

`extendTailwindMerge` allocates a large data structure (the merged
config). Calling it per render is O(n) where n is the size of the
config (~6 kB of class-group definitions). It also defeats the LRU
cache because each call creates a fresh cache. ALWAYS call once at
module scope.

## Anti-Pattern 3 : Missing prefix configuration

### Symptom

Project uses `prefix: "tw-"` in Tailwind config. `twMerge("tw-p-2", "tw-p-4")`
returns `"tw-p-2 tw-p-4"` with BOTH classes intact. Padding
non-deterministic.

### Wrong

```ts
// Tailwind config has prefix: "tw-"
import { twMerge } from "tailwind-merge";
// default twMerge does NOT know about the prefix
```

### Right

```ts
import { extendTailwindMerge } from "tailwind-merge";
export const twMerge = extendTailwindMerge({ prefix: "tw-" });
```

### Root cause

Default `twMerge` assumes bare utilities like `p-2`. With a prefix,
the parser cannot find `tw-` in its class group dictionary, so it
treats every prefixed class as an unknown class that does not conflict
with anything. ALWAYS configure `prefix` to match your Tailwind setup.

## Anti-Pattern 4 : Using twMerge for static class lists

### Symptom

Hot-path runtime overhead in lists/loops. Bundle size larger than
necessary.

### Wrong

```tsx
{items.map((item) => (
  <div key={item.id} className={twMerge("rounded border p-2")} />
))}
```

### Right

```tsx
{items.map((item) => (
  <div key={item.id} className="rounded border p-2" />
))}
```

### Root cause

`twMerge` exists to resolve conflicts. A single static string contains
no conflicts. Calling `twMerge` adds runtime cost and bundle weight
for ZERO benefit. ALWAYS prefer raw string literals when there are no
overrides or conditionals.

## Anti-Pattern 5 : Wrapping conditional-only classes with twMerge

### Symptom

Same as anti-pattern 4 : unnecessary runtime cost.

### Wrong

```tsx
<div className={twMerge("base", isActive && "ring-2 ring-blue-500")} />
```

`isActive` adds ONE class that doesn't conflict with the base. No
merge needed.

### Right

```tsx
import { clsx } from "clsx";
<div className={clsx("base", isActive && "ring-2 ring-blue-500")} />
```

OR if no array/object support needed :

```tsx
import { twJoin } from "tailwind-merge";
<div className={twJoin("base", isActive && "ring-2 ring-blue-500")} />
```

### Root cause

`twMerge` is heavier than `clsx` and `twJoin`. Use it only when class
override + conflict resolution is the goal. For pure conditional
concatenation, `clsx` is enough.

## Anti-Pattern 6 : Not registering a custom utility in tailwind-merge

### Symptom

```ts
twMerge("btn-sm", "btn-lg");
// "btn-sm btn-lg"   (BOTH survive, layout broken)
```

Custom utility classes work in browser ONLY because the last
declaration in the cascade wins. Across builds the order can shift.

### Wrong

```ts
// custom utility set NOT registered with tailwind-merge
import { twMerge } from "tailwind-merge";
```

### Right

```ts
import { extendTailwindMerge } from "tailwind-merge";

export const twMerge = extendTailwindMerge({
  extend: {
    classGroups: {
      "btn-size": ["btn-sm", "btn-md", "btn-lg"],
    },
  },
});

twMerge("btn-sm", "btn-lg");
// "btn-lg"
```

### Root cause

tailwind-merge only resolves conflicts inside its known class groups.
Custom utilities must be declared as a group so the library knows
they conflict with each other.

## Anti-Pattern 7 : Passing the same className twice through nested components

### Symptom

Consumer override mysteriously applies twice. Hard-to-debug
double-rendering of certain classes.

### Wrong

```tsx
function Card({ className, children }) {
  return (
    <div className={cn("rounded-lg", className)}>
      <CardBody className={cn("p-4", className)}>   {/* duplicates className */}
        {children}
      </CardBody>
    </div>
  );
}
```

### Right

```tsx
function Card({ className, children }) {
  return (
    <div className={cn("rounded-lg", className)}>
      <CardBody className="p-4">
        {children}
      </CardBody>
    </div>
  );
}
```

### Root cause

Forwarding the same `className` prop to multiple descendants makes
the override apply at every level. The DOM may render the consumer's
intended override on a child element where it makes no sense, or
override styles you wanted preserved.

## Anti-Pattern 8 : Forgetting the cn helper exists

### Symptom

Verbose, repeated `twMerge(clsx(...))` everywhere in the codebase.

### Wrong (everywhere)

```tsx
<button className={twMerge(clsx("base", isActive && "active", className))} />
```

### Right (lift it once)

```ts
// src/lib/utils.ts
import { type ClassValue, clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

```tsx
<button className={cn("base", isActive && "active", className)} />
```

### Root cause

The combination of `clsx` for conditionals and `twMerge` for conflict
resolution is so common that EVERY shadcn/ui project (and most
React+Tailwind projects) ships a `cn` helper. Adopt it.

## Anti-Pattern 9 : Using twMerge on non-Tailwind classes

### Symptom

```ts
twMerge("my-custom-class my-custom-other", "completely-unrelated");
// "my-custom-class my-custom-other completely-unrelated"
```

The library passes unknown classes through unchanged. Use of `twMerge`
adds cost for zero behaviour change.

### Right

For non-Tailwind class strings, use plain string concatenation or
`clsx` / `twJoin`. Reserve `twMerge` for actual Tailwind utility
strings.

## Anti-Pattern 10 : Mixed cva + twMerge but className applied BEFORE variants

### Symptom

Consumer's `className` is overridden by variants instead of overriding
them.

### Wrong

```tsx
<button className={cn(className, buttonVariants({ variant, size }))} />
// variants come SECOND, so they win
```

### Right

```tsx
<button className={cn(buttonVariants({ variant, size }), className)} />
// className comes LAST, so consumer wins
```

### Root cause

twMerge is LAST-WINS. Order matters. ALWAYS put the consumer's
`className` AT THE END so it can override variant defaults.

## Anti-Pattern 11 : twJoin where conflicts can happen

### Symptom

```ts
twJoin("p-2", "p-4");
// "p-2 p-4"   (no merge ; cascade-dependent)
```

Component renders with non-deterministic padding because both
classes ship to the browser.

### Right

```ts
twMerge("p-2", "p-4");
// "p-4"
```

### Root cause

`twJoin` does NOT do conflict resolution. Use it ONLY when you are
certain no conflicts can occur (e.g. a base class plus an unrelated
state class). When in doubt, use `twMerge`.

## Anti-Pattern 12 : Caching twMerge results manually

### Symptom

```tsx
const cachedClasses = useMemo(() => twMerge("a b c", className), [className]);
```

Premature optimisation that adds complexity without benefit.

### Right

```tsx
<div className={twMerge("a b c", className)} />
```

### Root cause

`twMerge` already memoises internally with a 500-entry LRU cache.
Manual memoisation via `useMemo` adds React overhead and another
cache layer that just duplicates work. Trust the built-in cache.

## Anti-Pattern 13 : Treating arbitrary values as different groups

### Symptom

```ts
twMerge("p-2", "p-[10px]");
```

Some developers expect `p-2` to survive because the second is "an
arbitrary value, different category". WRONG : both are in the
`padding` group and conflict.

### Result

```ts
// "p-[10px]"
```

twMerge correctly resolves arbitrary values vs named scale values as
the SAME group. Behaviour is correct ; expectation can be wrong.
Trust the library.

## Anti-Pattern 14 : Forgetting tailwind-merge in SSR/SSG output

### Symptom

Markup contains duplicate utilities (`p-2 p-4`). Stylesheet is correct
but the same class twice in the DOM looks unprofessional and confuses
Lighthouse audits.

### Right

Run `twMerge` (or `cn`) at render time, both server and client. The
library is isomorphic and SSR-safe. There is NO reason to skip it on
the server.

```tsx
// Both Next.js getServerSideProps and client render call the same component.
<div className={cn("p-2 bg-white", className)} />
```

## Anti-Pattern 15 : Treating tailwind-merge as a code formatter

### Symptom

Expecting `twMerge` to sort classes into "official order" or
normalise whitespace beyond the bare minimum.

### Reality

`twMerge` resolves conflicts but does NOT impose a canonical order on
non-conflicting classes. It preserves input order. For canonical
ordering, use the official Tailwind Prettier plugin :

```bash
npm install -D prettier prettier-plugin-tailwindcss
```

`prettier-plugin-tailwindcss` is a SEPARATE tool. tailwind-merge runs
at runtime, Prettier runs in the editor or pre-commit. They solve
different problems.
