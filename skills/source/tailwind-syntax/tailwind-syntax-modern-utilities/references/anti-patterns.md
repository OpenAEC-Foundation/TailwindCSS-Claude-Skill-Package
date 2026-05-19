# tailwind-syntax-modern-utilities : Anti-Patterns

## AP-1 : Animating `display: none` Without `transition-discrete`

**Symptom** : Popover or dialog snaps to visible / hidden instantly. No fade. No slide.

```html
<!-- WRONG -->
<div popover class="opacity-100 transition-opacity duration-300 starting:opacity-0">
  No animation
</div>
```

**Why** : The browser cannot transition `display` (or `visibility`) by default. Without `transition-behavior: allow-discrete`, the property snaps and the opacity transition has no time to play.

**Fix** :

```html
<div popover class="
  opacity-100 transition-all duration-300 transition-discrete
  starting:opacity-0
">
  Now animates
</div>
```

## AP-2 : Using `starting:` Without a Transition

**Symptom** : The initial style applies for one frame then jumps to the final state without animating.

```html
<!-- WRONG -->
<div class="starting:opacity-0 opacity-100">
  Snaps, not transitions
</div>
```

**Fix** : Always pair `starting:` with `transition-*` + `duration-*` :

```html
<div class="starting:opacity-0 opacity-100 transition-opacity duration-300">
  Smoothly fades in
</div>
```

## AP-3 : JS-Resize Hook Instead of `field-sizing-content`

**Symptom** : Janky textarea resize. Layout shift. Verbose component code.

```tsx
/* WRONG (v3 era hack) */
function AutoResize() {
  const ref = useRef<HTMLTextAreaElement>(null);
  useEffect(() => {
    const el = ref.current!;
    el.style.height = "auto";
    el.style.height = el.scrollHeight + "px";
  });
  return <textarea ref={ref} onInput={...} />;
}
```

**Fix** :

```tsx
<textarea className="field-sizing-content min-h-16 max-h-64" />
```

The browser handles the resize. No state, no refs, no effects.

## AP-4 : Setting `color-scheme` Inline When `scheme-*` Exists

```tsx
/* WRONG */
<input type="date" style={{ colorScheme: "light dark" }} />
```

**Fix** :

```tsx
<input type="date" className="scheme-light-dark" />
```

Inline styles bypass the cascade ; the utility participates in dark-mode swapping cleanly.

## AP-5 : Multiple `box-shadow` Custom CSS Instead of Layered Utilities

**Symptom** : Custom CSS file with 50+ lines stacking `inset` + outer shadows.

```css
/* WRONG */
.fancy-button {
  box-shadow:
    0 1px 2px rgba(0,0,0,0.1),
    inset 0 1px 0 rgba(255,255,255,0.1),
    0 0 0 1px rgba(0,0,0,0.05),
    inset 0 0 0 1px rgba(255,255,255,0.05);
}
```

**Fix** :

```html
<button class="
  shadow-sm
  inset-shadow-sm inset-shadow-white/10
  ring-1 ring-black/5
  inset-ring-1 inset-ring-white/5
">
  Same shadows, four utilities
</button>
```

Layered utilities compose into one `box-shadow` declaration. Dark-mode variants apply naturally.

## AP-6 : Using `font-stretch-*` Without a Variable Font

**Symptom** : `font-stretch-condensed` has no visual effect.

**Why** : Static fonts only have one width. `font-stretch: condensed` is silently ignored.

**Fix** :

1. Use a variable font that exposes a `wdth` axis :

```css
@font-face {
  font-family: "Inter Variable";
  src: url("/fonts/InterVariable.woff2") format("woff2");
  font-stretch: 50% 200%;
}
```

2. Verify the font supports the axis (check `font-variation-settings` capability) before relying on `font-stretch-*`.

## AP-7 : Backporting v4-Only Utilities to v3

**Symptom** : Build succeeds but classes have no effect in v3.

```html
<!-- WRONG when running v3 -->
<textarea class="field-sizing-content"></textarea>
```

**Why** : v3 does not generate these utilities. The class is meaningless from v3's perspective.

**Fix** : Skill compatibility frontmatter declares `v4.0+`. Check the project version. For v3 projects use custom CSS or a polyfill where possible.

## AP-8 : Mixing `starting:` Variant With JS-Driven Initial State

**Symptom** : Animation runs twice or in the wrong order.

```tsx
/* WRONG */
<motion.div
  initial={{ opacity: 0 }}
  animate={{ opacity: 1 }}
  className="starting:opacity-0 transition-opacity duration-300"
>
```

**Why** : Framer Motion sets the initial state via inline style. The `starting:` variant ALSO sets it. The two race.

**Fix** : Pick one strategy. For pure-CSS animations use `starting:` + `transition-*`. For JS-driven use the JS library only.

## AP-9 : `transition-discrete` Without Discrete-Property Transitions

**Symptom** : Class is wrapped in `transition-discrete` but does nothing because no discrete property is being transitioned.

```html
<!-- POINTLESS -->
<button class="transition-discrete bg-blue-500 hover:bg-blue-600">
  hover changes only colour
</button>
```

**Why** : `transition-discrete` only affects discrete properties (`display`, `content-visibility`, `overlay`). Continuous properties (`color`, `background`) ignore it.

**Fix** : Only apply when actually transitioning discrete properties. Skip otherwise (extra utility, no benefit).

## AP-10 : Forgetting `inset-shadow-*` Color Variant

**Symptom** : `inset-shadow-sm` produces a shadow but it inherits the default colour palette ; the highlight effect is invisible on a coloured background.

```html
<!-- WRONG : default inset shadow on blue button is invisible -->
<button class="bg-blue-500 inset-shadow-sm">Click</button>
```

**Fix** :

```html
<button class="bg-blue-500 inset-shadow-sm inset-shadow-white/20">Click</button>
```

The `/20` opacity modifier composes the colour with alpha. Visible on any base background.

## Verified Sources

- https://tailwindcss.com/docs/transition-behavior
- https://tailwindcss.com/docs/field-sizing
- https://tailwindcss.com/docs/color-scheme
- https://developer.mozilla.org/en-US/docs/Web/CSS/@starting-style

Last verified : 2026-05-19.
