# tailwind-impl-tailwind-merge : Examples

End-to-end realistic patterns.

## Example 1 : The shadcn cn Helper

The single most common use of `tailwind-merge`. Sits in
`src/lib/utils.ts` of every shadcn/ui project.

```ts
// src/lib/utils.ts
import { type ClassValue, clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

### Usage

```tsx
import { cn } from "@/lib/utils";

function Card({ className, children }: { className?: string; children: React.ReactNode }) {
  return (
    <div className={cn("rounded-lg border border-zinc-200 bg-white p-4 shadow-sm", className)}>
      {children}
    </div>
  );
}

// Consumer
<Card className="p-8 bg-zinc-100">...</Card>
```

`p-4` is replaced by `p-8`. `bg-white` is replaced by `bg-zinc-100`.
`rounded-lg`, `border`, `border-zinc-200`, `shadow-sm` survive.

## Example 2 : Conditional Classes (clsx via cn)

```tsx
function Tag({ tone, className }: { tone: "info" | "warn" | "error"; className?: string }) {
  return (
    <span
      className={cn(
        "inline-flex items-center rounded-full px-2 py-0.5 text-xs font-medium",
        tone === "info"  && "bg-blue-100 text-blue-800",
        tone === "warn"  && "bg-amber-100 text-amber-800",
        tone === "error" && "bg-red-100 text-red-800",
        className,
      )}
    />
  );
}
```

`clsx` resolves the boolean expressions ; `twMerge` resolves any
conflicts the consumer override introduces.

## Example 3 : The cva Pattern (Variant-Based Component)

```ts
// src/components/Button.tsx
import { cva, type VariantProps } from "class-variance-authority";
import { cn } from "@/lib/utils";

const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-md font-medium transition-colors focus:outline-none focus:ring-2 focus:ring-offset-2 disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        default:   "bg-blue-600 text-white hover:bg-blue-700 focus:ring-blue-500",
        secondary: "bg-zinc-200 text-zinc-900 hover:bg-zinc-300 focus:ring-zinc-400",
        outline:   "border border-zinc-300 text-zinc-900 hover:bg-zinc-100 focus:ring-zinc-400",
        ghost:     "text-zinc-900 hover:bg-zinc-100 focus:ring-zinc-400",
        danger:    "bg-red-600 text-white hover:bg-red-700 focus:ring-red-500",
      },
      size: {
        sm: "h-8 px-3 text-sm",
        md: "h-10 px-4 text-base",
        lg: "h-12 px-6 text-lg",
        icon: "h-10 w-10",
      },
    },
    defaultVariants: { variant: "default", size: "md" },
  }
);

export type ButtonProps =
  React.ButtonHTMLAttributes<HTMLButtonElement> & VariantProps<typeof buttonVariants>;

export function Button({ className, variant, size, ...props }: ButtonProps) {
  return <button className={cn(buttonVariants({ variant, size }), className)} {...props} />;
}
```

### Consumer

```tsx
<Button variant="outline" size="lg" className="rounded-full px-10">
  Subscribe
</Button>
```

Variants emit their classes ; consumer `className` overrides whatever
conflicts. Result : an outline button at lg size with rounded-full
borders and px-10.

## Example 4 : Compound Variants

Some combinations need extra classes that only apply when MULTIPLE
variants match.

```ts
const buttonVariants = cva("rounded-md", {
  variants: {
    variant: { default: "bg-blue-600", outline: "border" },
    size:    { sm: "px-3", lg: "px-6" },
  },
  compoundVariants: [
    {
      variant: "outline",
      size: "lg",
      class: "border-2 px-8",   // overrides size lg px-6 when also outline
    },
  ],
  defaultVariants: { variant: "default", size: "sm" },
});
```

`cva` outputs ALL relevant classes in order ; `twMerge` resolves
`px-6` vs `px-8` (px-8 wins, last).

## Example 5 : Custom Prefix

When your Tailwind project uses a prefix (e.g. `tw-` to avoid
collision with another framework), tailwind-merge MUST know.

### Tailwind v3 config

```js
// tailwind.config.js
module.exports = {
  prefix: "tw-",
  content: ["./src/**/*.{js,ts,jsx,tsx}"],
};
```

### Custom merge at module scope

```ts
// src/lib/tw.ts
import { extendTailwindMerge } from "tailwind-merge";
import { clsx, type ClassValue } from "clsx";

export const twMerge = extendTailwindMerge({ prefix: "tw-" });

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

### Usage

```tsx
<button className={cn("tw-p-2 tw-bg-blue-500", "tw-p-4")}>
  // resolves to "tw-bg-blue-500 tw-p-4"
</button>
```

## Example 6 : Registering a Custom Utility

For utilities defined as Tailwind plugins or as bespoke CSS, register
them so they participate in conflict resolution.

### Tailwind side (v4 CSS-first)

```css
@import "tailwindcss";

@utility ring-glow-sm {
  box-shadow: 0 0 4px rgba(0, 122, 255, 0.5);
}
@utility ring-glow-md {
  box-shadow: 0 0 8px rgba(0, 122, 255, 0.6);
}
@utility ring-glow-lg {
  box-shadow: 0 0 16px rgba(0, 122, 255, 0.7);
}
```

### tailwind-merge side

```ts
import { extendTailwindMerge } from "tailwind-merge";

export const twMerge = extendTailwindMerge({
  extend: {
    classGroups: {
      "ring-glow": ["ring-glow-sm", "ring-glow-md", "ring-glow-lg"],
    },
  },
});

twMerge("ring-glow-sm", "ring-glow-lg");
// "ring-glow-lg"
```

Without the registration, both classes would survive in markup
because tailwind-merge had no idea they were in the same group.

## Example 7 : Theme-Linked Utility

```ts
import { extendTailwindMerge, fromTheme } from "tailwind-merge";

type ThemeGroups = "brand-color";

export const twMerge = extendTailwindMerge<never, ThemeGroups>({
  extend: {
    theme: { "brand-color": ["primary", "secondary", "accent"] },
    classGroups: {
      "ring-brand": [{ ring: [fromTheme<ThemeGroups>("brand-color")] }],
    },
  },
});

twMerge("ring-primary", "ring-secondary");
// "ring-secondary"
```

## Example 8 : Disabling Cache for Testing

```ts
import { extendTailwindMerge } from "tailwind-merge";

// In test setup
export const testTwMerge = extendTailwindMerge({ cacheSize: 0 });

// Each call runs the full classifier ; useful when asserting
// pure-function behaviour under varying configs in unit tests.
```

## Example 9 : Conflict Resolution Across Variants

Variants (`hover:`, `focus:`, `dark:`) are treated as part of the
conflict identity :

```ts
twMerge("p-2 hover:p-4", "hover:p-8");
// "p-2 hover:p-8"

twMerge("dark:bg-zinc-900", "dark:bg-black");
// "dark:bg-black"

twMerge("md:p-2 lg:p-4", "lg:p-8");
// "md:p-2 lg:p-8"
```

Different variants -> different conflict scope. Same variant + same
group -> last wins.

## Example 10 : Forwarding Variants Through Component Boundaries

```tsx
type CardProps = VariantProps<typeof cardVariants> & {
  className?: string;
};

function Card({ tone, size, className, children }: CardProps & { children: React.ReactNode }) {
  return (
    <div className={cn(cardVariants({ tone, size }), className)}>
      <CardHeader className={cn("border-b", className)}>{children}</CardHeader>
    </div>
  );
}
```

CAUTION : passing the parent's `className` to a nested element can
cause that override to apply twice. Better pattern :

```tsx
function Card({ tone, size, className, children }) {
  return (
    <div className={cn(cardVariants({ tone, size }), className)}>
      {children}
    </div>
  );
}
```

## Example 11 : Vue 3 Composition API

```vue
<script setup lang="ts">
import { twMerge } from "tailwind-merge";
import { computed } from "vue";

const props = defineProps<{ className?: string; variant?: "default" | "outline" }>();

const classes = computed(() =>
  twMerge(
    "rounded-md px-4 py-2",
    props.variant === "outline" ? "border border-zinc-300" : "bg-blue-600 text-white",
    props.className,
  ),
);
</script>

<template>
  <button :class="classes"><slot /></button>
</template>
```

## Example 12 : Svelte

```svelte
<script lang="ts">
  import { twMerge } from "tailwind-merge";
  export let className: string = "";
  export let variant: "default" | "outline" = "default";

  $: classes = twMerge(
    "rounded-md px-4 py-2",
    variant === "outline" ? "border border-zinc-300" : "bg-blue-600 text-white",
    className,
  );
</script>

<button class={classes}><slot /></button>
```

## Example 13 : Solid

```tsx
import { twMerge } from "tailwind-merge";

function Button(props: { class?: string; variant?: "default" | "outline"; children: any }) {
  return (
    <button
      class={twMerge(
        "rounded-md px-4 py-2",
        props.variant === "outline" ? "border border-zinc-300" : "bg-blue-600 text-white",
        props.class,
      )}
    >
      {props.children}
    </button>
  );
}
```

## Example 14 : Build-Time Pre-Compute

For server-rendered output with no runtime overrides, you can call
`twMerge` ONCE at build time and inline the result :

```ts
// scripts/build-card-class.ts
import { twMerge } from "tailwind-merge";
import fs from "fs";

const result = twMerge("p-2 bg-white", "p-4 bg-zinc-100");
fs.writeFileSync(
  "src/generated/card-class.ts",
  `export const cardClass = ${JSON.stringify(result)};`
);
```

Then at runtime :

```tsx
import { cardClass } from "@/generated/card-class";
<div className={cardClass} />
```

Trades build complexity for zero runtime cost. Suitable when the
combined class list is static and known at build time.
