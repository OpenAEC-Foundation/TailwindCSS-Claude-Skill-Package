# tailwind-errors-specificity : Examples

Real diagnostic walk-throughs for cascade and specificity failures.

## Example 1 : Unlayered Custom CSS Wins (Most Common)

### Reported symptom

"I have `class="card p-8"` but the padding is the `.card` rule's
1rem, not the `p-8` utility's 2rem."

### Source

```css
/* app.css */
@import "tailwindcss";

.card {
  padding: 1rem;
}
```

### DevTools shows

```
.card { padding: 1rem; }                /* APPLIED */
@layer utilities { .p-8 { padding: 2rem; } }   /* STRIKED */
```

### Fix

```css
@import "tailwindcss";

@layer components {
  .card {
    padding: 1rem;
  }
}
```

Now `.card` is in `@layer components`. `.p-8` from `@layer utilities`
wins. Result : 2rem padding.

## Example 2 : Component Class From Plugin Beats Utility

### Reported symptom

"I apply `prose-lg` and then add `text-base`. Text is still `lg`."

### Cause

`@tailwindcss/typography` emits `prose-lg` -> children `p` get
`font-size: 1.125rem`. But the selector is `.prose-lg p`, specificity
0,0,2,0. `.text-base` on the `p` element is 0,0,1,0 -> loses.

### Wrong (cascade explanation)

```css
.prose-lg p { font-size: 1.125rem; }   /* 0,0,2,0 wins */
.text-base { font-size: 1rem; }        /* 0,0,1,0 */
```

### Fix (v4) : use `!`-suffix

```html
<article class="prose prose-lg">
  <p class="text-base!">This is base size</p>
</article>
```

### Fix (v3) : use `!`-prefix

```html
<article class="prose prose-lg">
  <p class="!text-base">This is base size</p>
</article>
```

### Better fix : use prose element-modifier

```html
<article class="prose prose-lg prose-p:text-base">
  <p>Now ALL p elements get text-base via prose-p modifier</p>
</article>
```

## Example 3 : @apply Strips !important in v3

### Reported symptom

"I wrote `@apply !bg-red-500` but the output is not `!important`."

### Source (v3)

```css
.danger-card {
  @apply !bg-red-500;   /* !important stripped on purpose */
}
```

### Compiled output

```css
.danger-card { background-color: rgb(239 68 68); }
```

No `!important`. Per v3 spec.

### Fix (v3)

```css
.danger-card {
  @apply bg-red-500 !important;
}
```

### Compiled output

```css
.danger-card { background-color: rgb(239 68 68) !important; }
```

### v4 behaviour : `@apply` PRESERVES `!`-suffix

```css
.danger-card {
  @apply bg-red-500!;
}
```

Compiled :

```css
.danger-card { background-color: var(--color-red-500) !important; }
```

## Example 4 : Wrong Important Syntax After v3 -> v4 Upgrade

### Reported symptom

"After upgrading to Tailwind v4, `!font-bold` no longer works
anywhere."

### Source

```html
<h2 class="!font-bold">Title</h2>
```

### Cause

v4 expects `!`-SUFFIX. v3 syntax is silently ignored : the `!font-bold`
token does not match any utility, so no CSS is emitted.

### Fix

```html
<h2 class="font-bold!">Title</h2>
```

### Codemod for migration

```bash
# Find all v3 !-prefix usages
grep -rEn 'class="[^"]*\b!\w+' src/

# Replace via sed (cautious : check matches first)
sed -i -E 's/!([a-z][a-z0-9:-]+)/\1!/g' src/**/*.tsx
```

ALWAYS review before applying. The official v4 codemod handles this :

```bash
npx @tailwindcss/upgrade@next
```

## Example 5 : Plugin Order Matters

### Reported symptom

"Both my custom plugin and `@tailwindcss/typography` emit `.prose-lg`.
My custom rules sometimes win, sometimes don't, across builds."

### Source

```js
// tailwind.config.js
module.exports = {
  plugins: [
    require("./my-custom-typography"),       // FIRST
    require("@tailwindcss/typography"),      // SECOND -> wins
  ],
};
```

### Fix : swap to make custom plugin LAST

```js
module.exports = {
  plugins: [
    require("@tailwindcss/typography"),      // FIRST
    require("./my-custom-typography"),       // LAST -> wins
  ],
};
```

Tailwind appends plugins in array order ; later entries are emitted
later in the CSS file, winning the cascade.

### Same in v4

```css
@plugin "@tailwindcss/typography";
@plugin "./my-custom-typography";   /* LAST -> wins */
```

## Example 6 : Same-Property Conflict in @apply

### Reported symptom

"I wrote `@apply px-4 py-2 px-8` thinking I'd override padding-x to 8,
but the layout uses 4."

### Source

```css
.btn {
  @apply px-4 py-2 px-8;
}
```

### Compiled output

```css
.btn {
  padding-left: 1rem; padding-right: 1rem;
  padding-top: 0.5rem; padding-bottom: 0.5rem;
  padding-left: 2rem; padding-right: 2rem;
}
```

px-8 wins. So why does layout show px-4 ? Because in YOUR test you
had `class="btn px-4"` ON THE ELEMENT :

```html
<button class="btn px-4">Click</button>
```

`.px-4` on the element is in `@layer utilities`, source-order later
than `.btn` rules. So utility px-4 wins over the .btn's padding-x: 2rem
because they have the same specificity AND px-4 is emitted later.

### Fix

Don't apply conflicting utilities on the same element. Pick one :

```html
<button class="btn">Click</button>   <!-- uses .btn's px-8 -->
```

OR define .btn without padding-x and put it on the element :

```css
.btn {
  @apply py-2 rounded-md font-medium;
}
```

```html
<button class="btn px-4">Click</button>   <!-- px-4 applies cleanly -->
```

## Example 7 : Third-Party CSS Beats Tailwind

### Reported symptom

"I'm migrating from Bootstrap to Tailwind. Bootstrap's `.btn` keeps
overriding my Tailwind utilities."

### Cause

Bootstrap's CSS is loaded AFTER Tailwind (or it's unlayered). Either
way, its `.btn` rule wins.

### Fix : wrap legacy CSS in a lower layer

```css
@import "tailwindcss";

@layer base {
  @import "bootstrap/dist/css/bootstrap.css";
}

/* OR wrap in a custom layer below tailwind utilities */
@layer base, components, utilities, app;
@layer base {
  @import "bootstrap/dist/css/bootstrap.css";
}
```

OR : remove Bootstrap class names from markup as you migrate.

### Last-resort fix : !-modifier on every overriding utility

```html
<button class="btn p-4! bg-blue-500! text-white!">Click</button>
```

Functional but a code smell : every override needs the modifier.
ALWAYS prefer the layer fix.

## Example 8 : Inline style Wins Over Class

### Reported symptom

"`<div style="padding: 10px" class="p-8">` renders with 10px padding."

### Cause

Inline `style` has specificity 1,0,0,0 -> beats every class-level
selector.

### Fix (v4)

```html
<div style="padding: 10px" class="p-8!">   <!-- p-8! beats inline -->
```

### Better fix : remove inline style

```jsx
// React
<div className="p-8">   {/* Use Tailwind, drop inline style */}
```

## Example 9 : Custom @utility Wins Cascade Correctly

### v4 example

```css
@import "tailwindcss";

@utility tab-4 {
  tab-size: 4;
}
```

Markup :

```html
<pre class="tab-4 hover:tab-2">code goes here</pre>
```

Both `tab-4` and `hover:tab-2` work. `tab-4` overrides any
`@layer components` rule for `.tab-4`. Hover variant works because
`@utility` integrates with variant pipeline.

### v3 example using @layer utilities

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer utilities {
  .tab-4 { tab-size: 4; }
}
```

Markup :

```html
<pre class="tab-4">code</pre>
```

`tab-4` wins because it's in @layer utilities. BUT `hover:tab-4`
does NOT auto-work in v3 without plugin-API integration (`addUtilities`).

## Example 10 : Scoped @apply Fails in Vue Without @reference

### Reported symptom

"In a Vue SFC `<style scoped>` block, `@apply text-2xl` throws
'Cannot apply unknown utility class'."

### Source

```vue
<template><h1>Hello</h1></template>
<style scoped>
  h1 { @apply text-2xl font-bold; }
</style>
```

### Cause

Scoped styles are processed in isolation from `app.css`. The Tailwind
processor has no theme context for `text-2xl`.

### Fix

```vue
<style scoped>
  @reference "../assets/app.css";
  h1 { @apply text-2xl font-bold; }
</style>
```

Or import default theme :

```vue
<style scoped>
  @reference "tailwindcss";
  h1 { @apply text-2xl font-bold; }
</style>
```

## Example 11 : Variants Strip !important Incorrectly

### Reported symptom

"In v3, `md:!text-xl` does not apply at md breakpoint."

### Source

```html
<p class="text-base md:!text-xl">Text</p>
```

### Diagnosis

In v3 the `!`-prefix goes BEFORE the utility, AFTER the variant chain :

```
md:!text-xl  -> CORRECT
!md:text-xl  -> WRONG (treats md: as part of the !utility name)
```

### Fix

ALWAYS place the `!` between the last variant and the utility name in
v3. In v4 the `!` is a suffix, simpler :

```html
<p class="text-base md:text-xl!">Text (v4)</p>
```

## Example 12 : DevTools Cascade Audit

For any "my class isn't applying" report :

1. Open the page in Chrome.
2. Right-click the element -> Inspect.
3. Look at the Styles pane on the right.
4. Find your class name in the matching rules.
5. If it's STRIKED THROUGH : a higher rule overrides it.
   - Click the higher rule to see its source location.
   - File path tells you where to fix : your own CSS ? plugin output ?
     unlayered ? @layer utilities ?
6. If it's MISSING : the class did not enter compiled CSS.
   -> See `tailwind-errors-build-failures`.

This single workflow resolves 95% of specificity reports.
