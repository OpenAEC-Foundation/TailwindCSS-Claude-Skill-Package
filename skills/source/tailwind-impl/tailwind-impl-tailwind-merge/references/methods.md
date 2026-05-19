# tailwind-impl-tailwind-merge : Methods Reference

Complete export surface of `tailwind-merge` v3 (Tailwind v4) and
v2 (Tailwind v3). Where the API differs between versions, both are
documented.

## Section 1 : Export Index

| Export | Type | Purpose |
|--------|------|---------|
| `twMerge` | function | Merge class strings with last-wins conflict resolution |
| `twJoin` | function | Join class strings without conflict resolution |
| `extendTailwindMerge` | function | Create a custom merge with extra class groups |
| `createTailwindMerge` | function | Create a merge from scratch (no defaults) |
| `getDefaultConfig` | function | Inspect or seed the default config |
| `mergeConfigs` | function | Combine multiple configs |
| `fromTheme` | function | Reference theme scales inside classGroups |
| `validators` | namespace | Helper validators for arbitrary value detection |
| `Config` | type | Top-level config shape |
| `ConfigExtension` | type | Shape passed to `extendTailwindMerge` |
| `ClassNameValue` | type | Accepted arg type for `twMerge` and `twJoin` |
| `DefaultClassGroupIds` | type | Union of all built-in class group IDs |
| `DefaultThemeGroupIds` | type | Union of all built-in theme group IDs |

## Section 2 : twMerge

```ts
function twMerge(
  ...classLists: ClassNameValue[]
): string;

type ClassNameValue = string | undefined | null | false | 0 | ClassNameValue[];
```

### Semantics

- Each argument is split on whitespace into class tokens.
- Tokens are categorised into class groups (e.g. `p-2` -> `padding`).
- Within ONE group, later tokens override earlier ones.
- Across groups, ALL non-conflicting tokens survive.
- Falsy values (`undefined`, `null`, `false`, `0`) are ignored.
- Nested arrays are flattened.

### Examples

```ts
twMerge("p-2", "p-4");               // "p-4"
twMerge("p-2 bg-white", "p-4");      // "bg-white p-4"
twMerge("hover:p-2", "hover:p-4");   // "hover:p-4"  (variant respected)
twMerge("p-2", "hover:p-4");         // "p-2 hover:p-4"  (different group)
twMerge("text-sm", "text-base");     // "text-base"  (text-size group)
twMerge("p-2", undefined, "p-4");    // "p-4"
twMerge(["p-2", false, "p-4"]);      // "p-4"
twMerge("text-sm leading-6", "text-base");
// "leading-6 text-base"  (text-base overrides text-sm AND its leading)
```

### Cache

Internal LRU cache with default size 500. Hits are O(1). Misses run
the full classifier.

## Section 3 : twJoin

```ts
function twJoin(
  ...classLists: ClassNameValue[]
): string;
```

Identical signature to `twMerge`. NO conflict resolution. NO Tailwind
knowledge. Pure string concatenation with whitespace and falsy
filtering. Subset of `clsx` (no object support).

### Examples

```ts
twJoin("p-2", "p-4");                // "p-2 p-4"  (no merge)
twJoin("base", hasActive && "active"); // "base active" or "base"
twJoin(["a", "b"], [["c"]]);         // "a b c"
```

ALWAYS prefer `twJoin` over `twMerge` when you know nothing will
conflict ; saves bundle size and CPU.

## Section 4 : extendTailwindMerge

```ts
function extendTailwindMerge<
  AdditionalClassGroupIds extends string = never,
  AdditionalThemeGroupIds extends string = never,
>(
  configExtension: ConfigExtension<
    DefaultClassGroupIds | AdditionalClassGroupIds,
    DefaultThemeGroupIds | AdditionalThemeGroupIds
  >,
  ...createConfig: ((config: GenericConfig) => GenericConfig)[]
): TailwindMerge;
```

### ConfigExtension shape

```ts
{
  cacheSize?: number;          // override LRU size, 0 disables
  prefix?: string;             // Tailwind prefix, e.g. "tw-"
  separator?: string;          // Tailwind separator, default ":"

  override?: {
    theme?: Record<string, ThemeScale>;
    classGroups?: Record<string, ClassGroup>;
    conflictingClassGroups?: Record<string, string[]>;
    conflictingClassGroupModifiers?: Record<string, string[]>;
    postfixLookupClassGroups?: string[];
    orderSensitiveModifiers?: string[];
  };

  extend?: {                   // additive (same shape as override)
    theme?: Record<string, ThemeScale>;
    classGroups?: Record<string, ClassGroup>;
    conflictingClassGroups?: Record<string, string[]>;
    conflictingClassGroupModifiers?: Record<string, string[]>;
    postfixLookupClassGroups?: string[];
    orderSensitiveModifiers?: string[];
  };
}
```

### Class group shape

A class group is an array. Each element is either :

- A string : `"animate-shimmer"` (one literal class)
- A function : `(value) => boolean` (matches any class part)
- A nested object : `{ animate: ["shimmer", "pulse"] }` (creates
  `animate-shimmer`, `animate-pulse`)
- A theme getter : `fromTheme("colors")` (reads from theme scales)

### Example : prefix + custom utility

```ts
import { extendTailwindMerge } from "tailwind-merge";

export const twMerge = extendTailwindMerge({
  prefix: "tw-",
  extend: {
    classGroups: {
      "btn-size": ["btn-sm", "btn-md", "btn-lg"],
      shadow: ["shadow-100", "shadow-200", "shadow-300"],
    },
  },
});
```

### Example : conflict between groups

```ts
extendTailwindMerge({
  extend: {
    classGroups: {
      "card-bg": ["card-bg-light", "card-bg-dark"],
    },
    conflictingClassGroups: {
      "card-bg": ["background-color"],   // card-bg-* conflicts with bg-*
    },
  },
});
```

### Example : validator-based group

```ts
import { extendTailwindMerge, validators } from "tailwind-merge";

extendTailwindMerge({
  extend: {
    classGroups: {
      "aspect-w": [{ "aspect-w": [validators.isNumber] }],
      "aspect-h": [{ "aspect-h": [validators.isNumber] }],
    },
  },
});
```

## Section 5 : createTailwindMerge

```ts
function createTailwindMerge(
  ...createConfig: ((config: GenericConfig) => GenericConfig)[]
): TailwindMerge;
```

NO inheritance from defaults. Use ONLY when the project has zero
Tailwind core classes and a fully custom utility set.

```ts
import { createTailwindMerge, getDefaultConfig } from "tailwind-merge";

export const twMerge = createTailwindMerge(() => ({
  cacheSize: 500,
  classGroups: {
    "my-padding": [{ p: [validators.isLength] }],
  },
  conflictingClassGroups: {},
  conflictingClassGroupModifiers: {},
  theme: {},
}));
```

## Section 6 : getDefaultConfig

```ts
function getDefaultConfig(): Config<DefaultClassGroupIds, DefaultThemeGroupIds>;
```

Returns the (large) default config. Useful for inspection :

```ts
import { getDefaultConfig } from "tailwind-merge";
console.log(Object.keys(getDefaultConfig().classGroups));
// [ 'aspect', 'container', 'columns', 'break-after', ... ]
```

## Section 7 : mergeConfigs

```ts
function mergeConfigs(
  base: Config,
  ...extensions: ConfigExtension[]
): Config;
```

Combines configs deterministically. Used internally by
`extendTailwindMerge`. Useful when composing design-system configs
from multiple packages.

```ts
import { extendTailwindMerge, mergeConfigs } from "tailwind-merge";

const designSystemA = { extend: { classGroups: { ... } } };
const designSystemB = { extend: { classGroups: { ... } } };

export const twMerge = extendTailwindMerge(
  mergeConfigs(designSystemA, designSystemB)
);
```

## Section 8 : fromTheme

```ts
function fromTheme<
  AdditionalThemeGroupIds extends string = never,
  DefaultThemeGroupIdsInner extends string = DefaultThemeGroupIds,
>(key: NoInfer<DefaultThemeGroupIdsInner | AdditionalThemeGroupIds>): ThemeGetter;
```

Returns a function used inside `classGroups` to reference theme
scales. Lets the same custom utility consume different theme values
without re-declaring them.

```ts
import { extendTailwindMerge, fromTheme } from "tailwind-merge";

type ThemeGroups = "custom-color";

extendTailwindMerge<never, ThemeGroups>({
  extend: {
    theme: { "custom-color": ["primary", "secondary"] },
    classGroups: {
      badge: [{ badge: [fromTheme<ThemeGroups>("custom-color")] }],
    },
  },
});
// recognises badge-primary, badge-secondary
```

## Section 9 : validators

`import { validators } from "tailwind-merge";`

| Validator | Matches |
|-----------|---------|
| `isNumber` | Decimal numbers : `5`, `3.14` |
| `isLength` | CSS lengths : `5px`, `1.5rem`, `0` |
| `isArbitraryValue` | `[...]` bracket syntax |
| `isArbitraryLength` | `[5px]`, `[1rem]` |
| `isArbitraryPosition` | `[top_left]`, `[50%_50%]` |
| `isArbitrarySize` | `[16/9]`, `[100%]` |
| `isArbitraryUrl` | `[url(...)]` |
| `isArbitraryShadow` | `[0_2px_4px_rgba(...)]` |
| `isFraction` | `1/2`, `16/9` |
| `isInteger` | `5`, `-3`, NOT `3.14` |
| `isPercent` | `50%`, `100%` |
| `isTshirtSize` | `xs`, `sm`, `md`, `lg`, `xl`, `2xl`, `3xl`, ... |
| `isAny` | Always returns true (debug) |

```ts
import { extendTailwindMerge, validators } from "tailwind-merge";

extendTailwindMerge({
  extend: {
    classGroups: {
      "shadow-elevation": [{ shadow: [validators.isTshirtSize] }],
    },
  },
});
```

## Section 10 : TypeScript Types

```ts
import type {
  Config,
  ConfigExtension,
  ClassNameValue,
  DefaultClassGroupIds,
  DefaultThemeGroupIds,
  ClassGroup,
  TailwindMerge,
} from "tailwind-merge";
```

`Config` shape (paraphrased) :

```ts
interface Config<TClassGroupIds extends string, TThemeGroupIds extends string> {
  cacheSize: number;
  prefix?: string;
  separator: string;
  theme: Record<TThemeGroupIds, ThemeScale>;
  classGroups: Record<TClassGroupIds, ClassGroup>;
  conflictingClassGroups: Record<TClassGroupIds, TClassGroupIds[]>;
  conflictingClassGroupModifiers: Record<TClassGroupIds, TClassGroupIds[]>;
  postfixLookupClassGroups: TClassGroupIds[];
  orderSensitiveModifiers: string[];
}
```

## Section 11 : Cache Behaviour

- Default `cacheSize`: 500 entries.
- LRU eviction.
- Cache key is the concatenation of all input strings (post-falsy-filter).
- Set `cacheSize: 0` to disable (debug or memory-constrained env).

Tuning :

```ts
extendTailwindMerge({ cacheSize: 2000 });   // hot path with many distinct combos
extendTailwindMerge({ cacheSize: 0 });      // disable for testing
```

## Section 12 : Bundle Size

Approximate sizes (minified + gzipped) :

| Function | Size |
|----------|------|
| `twMerge` (full default config) | ~6.5 kB |
| `twJoin` only | ~0.8 kB |
| `clsx` (peer) | ~0.3 kB |
| `cn = twMerge(clsx(...))` | ~6.8 kB |

ALWAYS tree-shake by importing only what you need :

```ts
import { twMerge } from "tailwind-merge";       // ok
import * as tailwindMerge from "tailwind-merge"; // imports everything
```
