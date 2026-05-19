# Gradients : Anti-Patterns

Each entry pairs a real failure with root cause and verified fix.
Sources : https://tailwindcss.com/docs/background-image,
https://v3.tailwindcss.com/docs/background-image,
https://tailwindcss.com/blog/tailwindcss-v4.

## AP-1 : `bg-gradient-*` Used In v4 (the Rename Trap)

### Symptom
Project upgrades to v4 but kept v3 markup. Gradients still render
correctly... except the upgrade tool emits a warning, and any new utility
combination on top (angles, interpolation) does not apply.

### Root cause
v4 renamed every `bg-gradient-*` to `bg-linear-*`. v4 ships a back-compat
shim that maps the old names to the new ones for the default eight
directions, but the new features (angle, conic, radial, interpolation
modifier) are wired only to the new prefix.

### NEVER
```html
<!-- v4 project keeping v3 markup -->
<div class="bg-gradient-to-r from-blue-500 to-purple-500"></div>
```

### ALWAYS run the codemod
```bash
npx @tailwindcss/upgrade
```

It rewrites every direction in one pass. After upgrade :
```html
<div class="bg-linear-to-r from-blue-500 to-purple-500"></div>
```

Source : https://tailwindcss.com/blog/tailwindcss-v4 ("we've renamed
`bg-gradient-*` to `bg-linear-*`").

## AP-2 : Muddy Gray Midpoint Between Complementary Colours

### Symptom
A red-to-green or blue-to-orange gradient shows a dingy gray band at the
50% point. The two endpoints look saturated ; the middle looks dead.

### Root cause
CSS's default `linear-gradient(...)` interpolates in sRGB. sRGB is a
non-perceptual colour space ; mixing complementary colours produces
desaturated midpoints. v4's default interpolation is OKLab, which is
perceptually uniform, but the default still applies the `in oklab`
modifier ONLY when the utility version is used. Plain `linear-gradient(...)`
in arbitrary form does NOT auto-add it.

### NEVER
```html
<!-- arbitrary form skips the v4 default interpolation -->
<div class="bg-[linear-gradient(to_right,#dc2626,#16a34a)]"></div>

<!-- explicit sRGB on a complementary pair -->
<div class="bg-linear-to-r/srgb from-red-500 to-green-500"></div>
```

### ALWAYS use v4 utilities (auto-applies oklab) OR opt into oklch
```html
<div class="bg-linear-to-r from-red-500 to-green-500"></div>                <!-- /oklab default -->
<div class="bg-linear-to-r/oklch from-red-500 to-green-500"></div>           <!-- vivid -->
```

Source : https://tailwindcss.com/blog/tailwindcss-v4 ("Using polar color
spaces like OKLCH or HSL can lead to much more vivid gradients when the
from-* and to-* colors are far apart on the color wheel").

## AP-3 : Lonely `from-` Without `to-`

### Symptom
`<div class="bg-linear-to-r from-blue-500"></div>` shows a gradient that
fades to nothing instead of holding a solid colour.

### Root cause
`to-` defaults to transparent. A `from-blue` `to-transparent` gradient
fades the right edge to invisible, which is what you see.

### NEVER ship a gradient utility with only `from-`. NEVER
```html
<div class="bg-linear-to-r from-blue-500"></div>
```

### ALWAYS pair `from-` with `to-`
```html
<div class="bg-linear-to-r from-blue-500 to-purple-500"></div>
```

If a fade-to-transparent IS the intent, make it explicit :
```html
<div class="bg-linear-to-r from-blue-500 to-transparent"></div>
```

## AP-4 : Conic Without `from-` (Invisible Gradient)

### Symptom
`<div class="bg-conic"></div>` paints nothing visible.

### Root cause
Without any `from-`, `via-`, or `to-` utility, the gradient resolves to
`transparent` everywhere. No colour stops means no visible gradient.

### NEVER
```html
<div class="size-32 rounded-full bg-conic"></div>
```

### ALWAYS provide at least `from-` AND `to-`
```html
<div class="size-32 rounded-full bg-conic from-red-500 via-yellow-500 to-red-500"></div>
```

For a smooth rainbow, repeat the same colour as start and end and use a
hue-walking interpolation :
```html
<div class="size-32 rounded-full bg-conic/[in_hsl_longer_hue] from-red-600 to-red-600"></div>
```

## AP-5 : `bg-radial-*` On v3

### Symptom
v3 project copies a v4 example and uses `<div class="bg-radial from-white to-zinc-900"></div>`.
The element renders no gradient.

### Root cause
v3 has NO native `bg-radial-*` utility. The class generates no CSS.

### NEVER use v4-only utilities in a v3 project.

### ALWAYS in v3 : use arbitrary
```html
<div class="bg-[radial-gradient(circle,theme(colors.white),theme(colors.zinc.900))]"></div>
```

OR upgrade to v4 to get the native utility.

## AP-6 : Stop Position Without Corresponding Colour

### Symptom
`<div class="bg-linear-to-r from-10% to-90%"></div>` produces nothing.

### Root cause
`from-10%` sets the POSITION ONLY ; it does not set the colour. Without a
companion `from-{color}`, the from-stop is transparent.

### NEVER
```html
<div class="bg-linear-to-r from-10% to-90%"></div>
```

### ALWAYS include BOTH colour and position
```html
<div class="bg-linear-to-r from-blue-500 from-10% to-pink-500 to-90%"></div>
```

The two utilities (colour + position) compose into a single CSS rule via
the `--tw-gradient-*` variables.

## AP-7 : `theme(colors.blue.500)` In v4 Arbitrary

### Symptom
v4 project copies v3 arbitrary gradient :
```html
<div class="bg-[linear-gradient(45deg,theme(colors.blue.500),theme(colors.purple.500))]"></div>
```
Build fails ; v4 does not recognise the `theme()` helper.

### Root cause
v4 dropped `theme()` for arbitrary values in favour of CSS variables.
The new equivalent is `var(--color-blue-500)`.

### NEVER in v4
```html
<div class="bg-[linear-gradient(45deg,theme(colors.blue.500),theme(colors.purple.500))]"></div>
```

### ALWAYS in v4 : use CSS variables OR the new utility
```html
<!-- with CSS variables -->
<div class="bg-[linear-gradient(45deg,var(--color-blue-500),var(--color-purple-500))]"></div>

<!-- much shorter : the new angle utility -->
<div class="bg-linear-45 from-blue-500 to-purple-500"></div>
```

## AP-8 : Stacking Variants Without Repeating The Gradient Direction

### Symptom
A button has a hover state changing the gradient colours :
```html
<button class="bg-linear-to-r from-blue-500 to-purple-500 hover:from-blue-600 hover:to-purple-600"></button>
```
Hover correctly changes the colours, but the user wants hover to ALSO
change direction to `bg-linear-to-br`. Adding only `hover:bg-linear-to-br`
breaks the hover-colour transition.

### Root cause
The direction utility and the colour stops are independent CSS rules. The
`hover:` direction change WORKS, but the cascade interaction with the
colour-stop hover means the rendered hover state combines them. The hover
transition IS correct ; this is more often a perceptual surprise than a
real bug. The real fix is to confirm intent : did you want hover to change
ALL THREE (direction + both colours)?

### ALWAYS write every aspect you want hover-reactive

```html
<button class="
  bg-linear-to-r hover:bg-linear-to-br
  from-blue-500 hover:from-blue-600
  to-purple-500 hover:to-purple-600
"></button>
```

## AP-9 : Underscore Instead Of Space In Arbitrary

### Symptom
`bg-radial-[at top left]` does not compile.

### Root cause
Arbitrary value brackets do not accept literal spaces. Use underscores ;
Tailwind converts them to spaces at output.

### NEVER
```html
<div class="bg-radial-[at top left] from-A to-B"></div>
```

### ALWAYS
```html
<div class="bg-radial-[at_top_left] from-A to-B"></div>
```

The output CSS becomes `radial-gradient(at top left in oklab, ...)`.

## AP-10 : Forgetting `bg-none` Before A Background Image Swap

### Symptom
Hover replaces a gradient with a solid background colour :
```html
<div class="bg-linear-to-r from-blue-500 to-purple-500 hover:bg-white"></div>
```
On hover, the gradient is STILL VISIBLE under the white background. The
two layers stack.

### Root cause
`bg-white` sets `background-color`, not `background-image`. The gradient
sits on `background-image` and is unaffected.

### NEVER assume `bg-{color}` removes a gradient.

### ALWAYS clear with `bg-none` before applying a solid

```html
<div class="
  bg-linear-to-r from-blue-500 to-purple-500
  hover:bg-none hover:bg-white
"></div>
```

`bg-none` sets `background-image: none`. Combine with `hover:bg-{color}`
for the solid background.

## AP-11 : Wrong Interpolation Modifier Order

### Symptom
`<div class="from-blue-500/oklch to-pink-500"></div>` does not change
interpolation. The slash modifier on a colour utility is OPACITY, not
interpolation.

### Root cause
The interpolation modifier goes on the GRADIENT UTILITY, not on the colour
stops. `from-blue-500/50` is "50% opacity blue 500". `from-blue-500/oklch`
is a typo : Tailwind tries to apply `oklch` as an opacity value.

### NEVER
```html
<div class="bg-linear-to-r from-blue-500/oklch to-pink-500"></div>
```

### ALWAYS put the interpolation modifier on the gradient utility
```html
<div class="bg-linear-to-r/oklch from-blue-500 to-pink-500"></div>
```

## AP-12 : Combining `via-` With `transparent` Endpoints

### Symptom
A glassmorphism-style overlay uses
`bg-linear-to-b from-transparent via-black/50 to-transparent`. The middle
band appears... gray, not the expected sharp dark.

### Root cause
Interpolating between transparent and a semi-opaque colour involves both
alpha AND colour channels. In sRGB, the alpha pre-multiplication produces
a gray midpoint that doesn't match the expected dark glow.

### ALWAYS prefer oklab/oklch for alpha-bearing gradients
```html
<div class="bg-linear-to-b/oklch from-transparent via-black/50 to-transparent"></div>
```

OR use solid colours with `to-transparent` only at the very edges :
```html
<div class="bg-linear-to-b from-black/50 to-transparent"></div>
```

## AP-13 : Codemod Confidence (Forgetting It Doesn't Rewrite Arbitrary)

### Symptom
After running the v3-to-v4 upgrade codemod, a project still has
`bg-[linear-gradient(45deg,#A,#B)]` instances. The codemod did not
convert them.

### Root cause
The codemod renames `bg-gradient-*` to `bg-linear-*` (utility-to-utility).
It does NOT generally rewrite ARBITRARY-VALUE strings into the new
parameterised utilities, because parsing arbitrary CSS is brittle.

### ALWAYS plan a manual pass after the codemod

```bash
# After the codemod, find every remaining arbitrary gradient
grep -r 'bg-\[linear-gradient' src/
grep -r 'bg-\[radial-gradient' src/
grep -r 'bg-\[conic-gradient' src/
```

Then rewrite each one to the new utility form where it improves clarity.

## AP-14 : `bg-conic-90` Expecting Rotation On Linear

### Symptom
A developer writes `bg-conic-90 from-blue-500 to-purple-500` expecting a
linear gradient at 90 degrees but conic-rendered. The result is a sweep
pattern, not a band.

### Root cause
`bg-conic-{N}` is for CONIC gradients (sweep around a centre point). For
a linear gradient at 90 degrees, use `bg-linear-90`.

### NEVER confuse conic and linear at the same angle.

### ALWAYS pick the right family

```html
<!-- conic sweep -->
<div class="size-32 bg-conic-90 from-blue-500 via-purple-500 to-blue-500"></div>

<!-- linear band -->
<div class="h-32 bg-linear-90 from-blue-500 to-purple-500"></div>
```

## AP-15 : Forgetting Browser Baseline For Conic Gradients

### Symptom
A v3 project using arbitrary `bg-[conic-gradient(...)]` ships fine on
modern Chrome but appears blank on Safari 12.

### Root cause
Conic gradients require Safari 12.1+, Chrome 69+, Firefox 83+. Older
Safari versions ignore the entire rule.

### ALWAYS verify browser support before relying on conic gradients in v3.

### ALWAYS in v4 : the baseline (Safari 16.4+) covers conic already, so
this is a v3 concern only.
