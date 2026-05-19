# tailwind-errors-specificity : Methods Reference

Complete control surface for cascade, layer order, important
modifier, and plugin ordering.

## Section 1 : The CSS @layer Cascade

Tailwind emits a native `@layer` block in compiled output :

```css
@layer base, components, utilities;

@layer base {
  /* Preflight reset, plugin-base styles */
}
@layer components {
  /* Tailwind components plugin output, .prose, .btn-* from plugins */
}
@layer utilities {
  /* p-4, bg-red-500, hover:bg-blue-700, etc. */
}

/* UNLAYERED region (NOT inside any @layer) */
.your-untagged-custom-rule {
  /* This wins over EVERYTHING above */
}
```

### Cascade priority (highest -> lowest)

| Rank | Source | Wins over |
|------|--------|-----------|
| 1 | Inline `style` attribute | Everything below |
| 2 | `!important` declarations | Non-important below |
| 3 | Unlayered CSS rules | Any @layer (within same specificity) |
| 4 | @layer utilities | components + base |
| 5 | @layer components | base |
| 6 | @layer base | Browser defaults |
| 7 | Browser defaults | (lowest) |

`!important` flips the comparison : an `!important` declaration in
ANY layer beats non-important in ANY higher layer.

## Section 2 : @layer Directive (v3)

### Declaring custom CSS in a layer

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  h1 { font-family: serif; }
}

@layer components {
  .btn { padding: 0.5rem 1rem; border-radius: 0.375rem; }
}

@layer utilities {
  .tab-4 { tab-size: 4; }
}
```

### What @layer does

- Moves the contained CSS into the corresponding @tailwind injection
  point.
- Enables variant generation (`hover:`, `md:`, `dark:`) for the class
  inside `@layer utilities` (in v3 only via the plugin API or by using
  `@layer utilities` with `addVariant`).
- Establishes cascade priority via native CSS @layer.

### What @layer does NOT do

- Does NOT change selector specificity. Two classes have the same
  0,0,1,0 specificity regardless of layer.
- Does NOT bypass `!important`.

## Section 3 : @layer Directive (v4)

```css
@import "tailwindcss";

@layer base {
  h1 { font-family: serif; }
}

@layer components {
  .btn { padding: 0.5rem 1rem; }
}
```

In v4, custom utilities should PREFER the `@utility` directive over
`@layer utilities` :

```css
@utility tab-4 {
  tab-size: 4;
}
```

`@utility` automatically participates in the variant pipeline. Custom
utilities authored in `@layer utilities` may not fully integrate with
all variants reliably.

## Section 4 : Important Modifier Syntax

### v3 : prefix

```html
<div class="!font-bold !bg-red-500 md:!text-xl hover:!underline">
```

Generated CSS :

```css
.\!font-bold { font-weight: 700 !important; }
.\!bg-red-500 { background-color: rgb(239 68 68) !important; }
.md\:\!text-xl { ... } /* media query wrap */
```

### v4 : suffix

```html
<div class="font-bold! bg-red-500! md:text-xl! hover:underline!">
```

Generated CSS :

```css
.font-bold\! { font-weight: 700 !important; }
.bg-red-500\! { background-color: var(--color-red-500) !important; }
```

### Global important: flag (v3 only)

```js
// tailwind.config.js
module.exports = {
  important: true,                          // ALL utilities become !important
  // OR
  important: "#app",                        // wrap all selectors with #app
};
```

v4 has NO equivalent global flag. Use per-class `!`-suffix.

## Section 5 : @apply Important Behavior

### v3 default

```css
.btn {
  @apply !p-4 !bg-blue-500;   /* !important STRIPPED in v3 */
}
```

Result :

```css
.btn { padding: 1rem; background-color: rgb(59 130 246); }
```

### v3 force important

```css
.btn {
  @apply p-4 bg-blue-500 !important;   /* MANUAL !important at end */
}
```

Result :

```css
.btn { padding: 1rem !important; background-color: rgb(59 130 246) !important; }
```

In SCSS use `#{!important}` because Sass parses `!important` differently.

### v4

```css
.btn {
  @apply p-4! bg-blue-500!;   /* PRESERVED */
}
```

Result :

```css
.btn { padding: 1rem !important; background-color: var(--color-blue-500) !important; }
```

## Section 6 : Declaration Order Inside @apply

Within ONE `@apply` line, conflicting properties resolve by
declaration order :

```css
.btn {
  @apply px-4 py-2 px-8;
}
```

Result :

```css
.btn {
  padding-left: 1rem; padding-right: 1rem;   /* px-4 */
  padding-top: 0.5rem; padding-bottom: 0.5rem;
  padding-left: 2rem; padding-right: 2rem;   /* px-8 wins, declared last */
}
```

px-8 wins because it comes later. Specificity is identical.

ALWAYS audit @apply for same-property duplicates :

```bash
# Lint helper : grep for likely conflicts
grep -rn "@apply.*p-.*p-" src/
grep -rn "@apply.*m-.*m-" src/
grep -rn "@apply.*bg-.*bg-" src/
```

## Section 7 : Plugin Order (v3 plugins Array)

```js
// tailwind.config.js
module.exports = {
  plugins: [
    require("@tailwindcss/typography"),  // FIRST -> lowest priority
    require("./my-custom-plugin"),       // SECOND -> overrides typography
    require("./theme-overrides"),        // LAST -> highest priority
  ],
};
```

ALL plugins emit into `@layer components` (or `@layer utilities` if
they use `addUtilities`). Within the layer, LATER entries appear LATER
in the compiled CSS file, so they win the cascade.

### Plugin order in v4

```css
@plugin "@tailwindcss/typography";
@plugin "./my-custom-plugin";
@plugin "./theme-overrides";
```

Same rule : LAST `@plugin` line wins.

## Section 8 : Selector Specificity (Tailwind utilities)

Every Tailwind utility uses a single-class selector. Specificity is
0,0,1,0.

```css
.p-4 { padding: 1rem; }              /* 0,0,1,0 */
.hover\:p-4:hover { padding: 1rem; } /* 0,0,2,0 */
.md\:p-4 { padding: 1rem; }          /* 0,0,1,0, wrapped in @media */
```

What this means in practice :

- `.my-component .p-4` (descendant combinator) -> 0,0,2,0 -> beats
  `.p-4` alone.
- A two-class selector you wrote (`.card.dark`) -> 0,0,2,0 -> beats
  a Tailwind utility.

Avoid wrapping Tailwind utilities in higher-specificity selectors. The
`@utility` directive (v4) and `@layer utilities` (both versions)
ensure utilities are emitted as single-class.

## Section 9 : @utility Directive (v4)

```css
@utility tab-4 {
  tab-size: 4;
}

@utility ring-glow-* {
  box-shadow: 0 0 var(--ring-glow-size, 4px) rgba(0,122,255,0.5);
}
```

`@utility` differences vs `@layer utilities` :

| Behaviour | @utility | @layer utilities |
|-----------|----------|------------------|
| Auto variants (hover, md, dark) | yes | partial |
| Functional `--value()`, `--modifier()` | yes | no |
| Overrides components when applied directly | yes | yes |
| Single-class selector emitted | yes | yes |
| Available in v3 | no | yes |

ALWAYS use `@utility` for new custom utilities in v4.

## Section 10 : @reference (v4 scoped styles)

```vue
<style>
  @reference "../app.css";
  h1 { @apply text-2xl font-bold; }
</style>
```

`@reference` IMPORTS theme variables, custom utilities, and variants
WITHOUT emitting their CSS into the scoped output. Required for :

- Vue `<style scoped>`
- Vue `<style module>`
- Svelte `<style>`
- CSS modules
- MDX inline `<style>`

Without `@reference` the scoped CSS has no theme context and `@apply`
fails with "Cannot apply unknown utility class".

## Section 11 : Debugging Cascade in DevTools

1. Select the element in Inspector.
2. Open Styles pane.
3. Look at the LIST of matching rules in cascade order :
   - Top entry = currently applied
   - Strike-through entry = overridden by a higher rule
4. Hover the strike-through rule -> shows which rule wins.
5. Click "Layers" in DevTools (Chrome 99+) to see layer order.

Common findings :

- Your utility is struck through by an UNLAYERED `.your-card` rule
  -> wrap that rule in `@layer components`.
- Your utility is struck through by an `!important` from third-party
  CSS -> apply `!`-modifier on your utility.
- Your utility is struck through by an INLINE `style` -> remove the
  inline style OR add `!`-modifier on your utility.

## Section 12 : Quick Audit Commands

```bash
# Find unlayered custom CSS in src/
grep -rL "@layer" src/styles/

# Find !-prefix usage in a v4 project (likely v3 holdover)
grep -rn 'class=".*![a-z]' src/

# Find !-suffix usage in a v3 project (likely v4 holdover)
grep -rn 'class=".*[a-z]!"' src/

# Find @apply lines with same-property conflicts
grep -rn "@apply.*p-.*p-\|@apply.*m-.*m-\|@apply.*bg-.*bg-" src/

# Inspect compiled CSS layer count
grep -c "@layer" dist/output.css
```
