# tailwind-impl-build-postcss-cli : Examples

End-to-end working setups by framework and version. Every example is
copy-paste runnable.

## Example 1 : Next.js 14+ (App Router) with Tailwind v4

### Install

```bash
npm install tailwindcss @tailwindcss/postcss postcss
```

### Files

```js
// postcss.config.mjs
export default {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
```

```css
/* app/globals.css */
@import "tailwindcss";
```

```tsx
// app/layout.tsx
import "./globals.css";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

### Run

```bash
npm run dev    # Next.js compiles via PostCSS automatically
```

No CLI invocation needed. Next.js owns the build pipeline.

## Example 2 : Next.js 14+ with Tailwind v3

### Install

```bash
npm install -D tailwindcss@3 postcss autoprefixer
npx tailwindcss init -p
```

### Files

```js
// postcss.config.js
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

```js
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./app/**/*.{js,ts,jsx,tsx}",
    "./pages/**/*.{js,ts,jsx,tsx}",
    "./components/**/*.{js,ts,jsx,tsx}",
  ],
  theme: { extend: {} },
  plugins: [],
};
```

```css
/* app/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## Example 3 : Remix (v4)

### Install

```bash
npm install tailwindcss @tailwindcss/postcss postcss
```

### Files

```js
// postcss.config.cjs (Remix defaults to CommonJS)
module.exports = {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
```

```css
/* app/tailwind.css */
@import "tailwindcss";
```

```tsx
// app/root.tsx
import styles from "./tailwind.css?url";
export const links: LinksFunction = () => [
  { rel: "stylesheet", href: styles },
];
```

## Example 4 : Astro (v4)

Use the Vite plugin path inside Astro.

### Install

```bash
npm install tailwindcss @tailwindcss/vite
```

### Files

```js
// astro.config.mjs
import { defineConfig } from "astro/config";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  vite: { plugins: [tailwindcss()] },
});
```

```css
/* src/styles/global.css */
@import "tailwindcss";
```

```astro
---
// src/layouts/Layout.astro
import "../styles/global.css";
---
<html>
  <body><slot /></body>
</html>
```

## Example 5 : Plain HTML, Tailwind v4 CLI

Static site, no bundler, no Node frameworks.

### Install

```bash
npm init -y
npm install tailwindcss @tailwindcss/cli
```

### Files

```css
/* src/input.css */
@import "tailwindcss";
```

```html
<!-- public/index.html -->
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="stylesheet" href="./dist/output.css" />
  </head>
  <body>
    <h1 class="text-3xl font-bold text-blue-600 underline">Hello world!</h1>
  </body>
</html>
```

```json
// package.json (scripts excerpt)
{
  "scripts": {
    "dev": "npx @tailwindcss/cli -i ./src/input.css -o ./public/dist/output.css --watch",
    "build": "npx @tailwindcss/cli -i ./src/input.css -o ./public/dist/output.css --minify"
  }
}
```

### Run

```bash
npm run dev    # development : watcher recompiles on save
npm run build  # production : single minified output
```

## Example 6 : Rails 7 with Tailwind v4 standalone binary

Goal : no Node.js dependency in the Rails image.

### Install binary

```bash
mkdir -p bin
curl -sLO https://github.com/tailwindlabs/tailwindcss/releases/latest/download/tailwindcss-linux-x64
chmod +x tailwindcss-linux-x64
mv tailwindcss-linux-x64 bin/tailwindcss
```

### Files

```css
/* app/assets/stylesheets/application.tailwind.css */
@import "tailwindcss";
@source "../../../app/views";
@source "../../../app/helpers";
```

```ruby
# lib/tasks/tailwind.rake
namespace :tailwind do
  desc "Build Tailwind CSS"
  task :build do
    sh "./bin/tailwindcss " \
       "-i app/assets/stylesheets/application.tailwind.css " \
       "-o app/assets/builds/application.css --minify"
  end

  desc "Watch Tailwind CSS"
  task :watch do
    sh "./bin/tailwindcss " \
       "-i app/assets/stylesheets/application.tailwind.css " \
       "-o app/assets/builds/application.css --watch"
  end
end

# Hook into Rails asset pipeline
Rake::Task["assets:precompile"].enhance(["tailwind:build"])
```

## Example 7 : Laravel Blade with v4 standalone CLI

### Install via npm (Node already in Laravel projects)

```bash
npm install tailwindcss @tailwindcss/cli
```

### Files

```css
/* resources/css/app.css */
@import "tailwindcss";
@source "../views";
```

```blade
{{-- resources/views/layouts/app.blade.php --}}
<!doctype html>
<html>
  <head>
    <link rel="stylesheet" href="{{ asset('css/app.css') }}" />
  </head>
  <body>@yield('content')</body>
</html>
```

```json
// package.json scripts
{
  "scripts": {
    "dev": "npx @tailwindcss/cli -i ./resources/css/app.css -o ./public/css/app.css --watch",
    "build": "npx @tailwindcss/cli -i ./resources/css/app.css -o ./public/css/app.css --minify"
  }
}
```

## Example 8 : Django with v4 standalone binary

### Install binary

```bash
curl -sLO https://github.com/tailwindlabs/tailwindcss/releases/latest/download/tailwindcss-linux-x64
chmod +x tailwindcss-linux-x64
mv tailwindcss-linux-x64 ./bin/tailwindcss
```

### Files

```css
/* static/src/input.css */
@import "tailwindcss";
@source "../../templates";
@source "../../**/templates";
```

```python
# manage.py-adjacent helper (build_css.py)
import subprocess
subprocess.check_call([
    "./bin/tailwindcss",
    "-i", "static/src/input.css",
    "-o", "static/dist/output.css",
    "--minify",
])
```

```html
<!-- templates/base.html -->
{% load static %}
<link rel="stylesheet" href="{% static 'dist/output.css' %}" />
```

## Example 9 : Go html/template with v3 standalone binary

### Install (Linux)

```bash
curl -sLO https://github.com/tailwindlabs/tailwindcss/releases/download/v3.4.0/tailwindcss-linux-x64
chmod +x tailwindcss-linux-x64
mv tailwindcss-linux-x64 ./bin/tailwindcss
```

### Files

```js
// tailwind.config.js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./templates/**/*.html"],
  theme: { extend: {} },
  plugins: [],
};
```

```css
/* assets/input.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

```makefile
# Makefile
css-dev:
	./bin/tailwindcss -i ./assets/input.css -o ./static/css/output.css --watch

css-prod:
	./bin/tailwindcss -i ./assets/input.css -o ./static/css/output.css --minify
```

```go
// main.go (excerpt)
http.Handle("/static/", http.StripPrefix("/static/", http.FileServer(http.Dir("static"))))
```

## Example 10 : Phoenix LiveView with v4 standalone binary

### Install via Mix dep (esbuild/tailwind pattern)

Phoenix has an official `tailwind` Hex package wrapping the binary.

```elixir
# mix.exs
defp deps do
  [
    {:tailwind, "~> 0.2", runtime: Mix.env() == :dev},
  ]
end
```

### Config

```elixir
# config/config.exs
config :tailwind,
  version: "4.0.0",
  default: [
    args: ~w(
      --input=assets/css/app.css
      --output=priv/static/assets/app.css
    ),
    cd: Path.expand("..", __DIR__)
  ]
```

### Files

```css
/* assets/css/app.css */
@import "tailwindcss";
@source "../js";
@source "../../lib/*_web";
```

### Run

```bash
mix tailwind default --watch    # dev
mix tailwind default --minify   # prod
```

## Example 11 : SvelteKit with v4 PostCSS

(Vite plugin is preferred, but PostCSS works for legacy chains.)

```bash
npm install tailwindcss @tailwindcss/postcss postcss
```

```js
// postcss.config.js
export default {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
```

```css
/* src/app.css */
@import "tailwindcss";
```

```svelte
<!-- src/routes/+layout.svelte -->
<script>
  import "../app.css";
</script>
<slot />
```

Scoped styles in v4 need `@reference` :

```svelte
<style>
  @reference "../app.css";
  h1 { @apply text-2xl font-bold; }
</style>
```

## Example 12 : Webpack (v3) custom setup

```bash
npm install -D tailwindcss@3 postcss autoprefixer postcss-loader style-loader css-loader
```

```js
// postcss.config.js
module.exports = { plugins: { tailwindcss: {}, autoprefixer: {} } };
```

```js
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader", "postcss-loader"],
      },
    ],
  },
};
```

```js
// tailwind.config.js
module.exports = {
  content: ["./src/**/*.{html,js}"],
  theme: { extend: {} },
  plugins: [],
};
```

```css
/* src/styles.css */
@tailwind base;
@tailwind components;
@tailwind utilities;
```

```js
// src/index.js
import "./styles.css";
```

## Example 13 : CI minify on production deploy

GitHub Actions snippet :

```yaml
# .github/workflows/build.yml
- name: Install Tailwind
  run: npm install tailwindcss @tailwindcss/cli
- name: Build CSS
  run: npx @tailwindcss/cli -i ./src/input.css -o ./dist/output.css --minify
```

Pinning a binary version in CI is preferred over `latest` to keep
builds reproducible. Match the binary version to your local dev
install :

```yaml
- name: Install binary
  run: |
    curl -sLO https://github.com/tailwindlabs/tailwindcss/releases/download/v4.0.0/tailwindcss-linux-x64
    chmod +x tailwindcss-linux-x64
    mv tailwindcss-linux-x64 /usr/local/bin/tailwindcss
- name: Build CSS
  run: tailwindcss -i ./src/input.css -o ./dist/output.css --minify
```

## Example 14 : Output to stdout, pipe to a custom transform

```bash
npx @tailwindcss/cli -i ./src/input.css --minify \
  | some-css-postprocessor \
  > ./dist/output.css
```

Omit `--output`/`-o` to write to stdout. Useful when a downstream tool
(critical-CSS extractor, RTL flipper, custom optimiser) consumes the
stream.
