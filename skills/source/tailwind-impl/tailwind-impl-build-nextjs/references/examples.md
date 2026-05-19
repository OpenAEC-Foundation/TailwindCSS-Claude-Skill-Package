# Examples : Tailwind CSS in Next.js

Working code, fully copy-pasteable. Each example is a complete file set.

## Example 1 : v4 + App Router + TypeScript (Minimal)

`package.json` :

```json
{
  "name": "tw-next-v4-app",
  "version": "0.0.1",
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  },
  "dependencies": {
    "next": "^15.0.0",
    "react": "^19.0.0",
    "react-dom": "^19.0.0"
  },
  "devDependencies": {
    "tailwindcss": "^4.0.0",
    "@tailwindcss/postcss": "^4.0.0",
    "postcss": "^8.4.0",
    "typescript": "^5.0.0",
    "@types/react": "^19.0.0",
    "@types/node": "^22.0.0"
  }
}
```

`postcss.config.mjs` :

```js
const config = {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
export default config;
```

`app/globals.css` :

```css
@import "tailwindcss";
```

`app/layout.tsx` :

```tsx
import "./globals.css";

export const metadata = {
  title: "TW Next v4",
  description: "Minimal v4 + App Router",
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body className="bg-zinc-950 text-zinc-50 antialiased">{children}</body>
    </html>
  );
}
```

`app/page.tsx` :

```tsx
export default function Home() {
  return (
    <main className="mx-auto max-w-3xl px-6 py-24">
      <h1 className="text-5xl font-bold tracking-tight">Hello Tailwind v4</h1>
      <p className="mt-4 text-lg text-zinc-400">
        Running on the App Router.
      </p>
    </main>
  );
}
```

## Example 2 : v4 + App Router + next/font/google

Same `package.json`, `postcss.config.mjs`, and `app/page.tsx` as Example 1.

`app/globals.css` :

```css
@import "tailwindcss";

@theme inline {
  --font-sans: var(--font-inter), ui-sans-serif, system-ui, sans-serif;
  --font-mono: var(--font-jetbrains), ui-monospace, monospace;
}
```

`app/layout.tsx` :

```tsx
import "./globals.css";
import { Inter, JetBrains_Mono } from "next/font/google";

const inter = Inter({
  subsets: ["latin"],
  variable: "--font-inter",
  display: "swap",
});

const jetbrains = JetBrains_Mono({
  subsets: ["latin"],
  variable: "--font-jetbrains",
  display: "swap",
});

export const metadata = { title: "TW Next v4 with Fonts" };

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en" className={`${inter.variable} ${jetbrains.variable}`}>
      <body className="bg-zinc-950 font-sans text-zinc-50 antialiased">
        {children}
      </body>
    </html>
  );
}
```

A `<code className="font-mono">` element on any page will render in
JetBrains Mono. A bare `<p>` inherits Inter via `font-sans` on `<body>`.

## Example 3 : v4 + Pages Router

`postcss.config.mjs` identical to Example 1.

`styles/globals.css` :

```css
@import "tailwindcss";
```

`pages/_app.tsx` :

```tsx
import "@/styles/globals.css";
import type { AppProps } from "next/app";

export default function App({ Component, pageProps }: AppProps) {
  return <Component {...pageProps} />;
}
```

`pages/index.tsx` :

```tsx
export default function Home() {
  return (
    <main className="mx-auto max-w-3xl px-6 py-24">
      <h1 className="text-5xl font-bold tracking-tight">Pages Router</h1>
    </main>
  );
}
```

## Example 4 : v3 + App Router (Pinned)

`package.json` :

```json
{
  "name": "tw-next-v3-app",
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start"
  },
  "dependencies": {
    "next": "^14.0.0",
    "react": "^18.0.0",
    "react-dom": "^18.0.0"
  },
  "devDependencies": {
    "tailwindcss": "^3.4.0",
    "postcss": "^8.4.0",
    "autoprefixer": "^10.4.0"
  }
}
```

`tailwind.config.js` :

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./app/**/*.{js,ts,jsx,tsx,mdx}",
    "./pages/**/*.{js,ts,jsx,tsx,mdx}",
    "./components/**/*.{js,ts,jsx,tsx,mdx}",
    "./src/**/*.{js,ts,jsx,tsx,mdx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

`postcss.config.js` :

```js
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};
```

`app/globals.css` :

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

`app/layout.tsx` :

```tsx
import "./globals.css";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>{children}</body>
    </html>
  );
}
```

## Example 5 : v3 + App Router + next/font/google

Extends Example 4 by adding Inter wiring.

`tailwind.config.js` :

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./app/**/*.{js,ts,jsx,tsx,mdx}",
    "./pages/**/*.{js,ts,jsx,tsx,mdx}",
    "./components/**/*.{js,ts,jsx,tsx,mdx}",
  ],
  theme: {
    extend: {
      fontFamily: {
        sans: [
          "var(--font-inter)",
          "ui-sans-serif",
          "system-ui",
          "sans-serif",
        ],
      },
    },
  },
  plugins: [],
};
```

`app/layout.tsx` :

```tsx
import "./globals.css";
import { Inter } from "next/font/google";

const inter = Inter({
  subsets: ["latin"],
  variable: "--font-inter",
  display: "swap",
});

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en" className={inter.variable}>
      <body className="font-sans">{children}</body>
    </html>
  );
}
```

## Example 6 : v3 + Pages Router

Same `package.json`, `tailwind.config.js`, `postcss.config.js` as
Example 4.

`styles/globals.css` :

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

`pages/_app.tsx` :

```tsx
import "@/styles/globals.css";
import type { AppProps } from "next/app";

export default function App({ Component, pageProps }: AppProps) {
  return <Component {...pageProps} />;
}
```

## Example 7 : Catch-All Route Glob Fix (v3)

Project with a dynamic blog route at `app/blog/[...slug]/page.tsx`. The
classes on that page do not generate CSS with the default content glob.

`tailwind.config.js` fix :

```js
module.exports = {
  content: [
    "./app/**/*.{js,ts,jsx,tsx,mdx}",
    "./app/[[]**[]]/*.{js,ts,jsx,tsx,mdx}",
    "./app/[[]**[]]/**/*.{js,ts,jsx,tsx,mdx}",
  ],
  theme: { extend: {} },
  plugins: [],
};
```

The escaped-bracket pattern `[[]` matches a literal `[` and `[]]` matches
a literal `]`. Combined they target folders whose name starts with `[`
(catch-all and dynamic segments).

## Example 8 : Catch-All Route Glob Fix (v4)

Same project, v4 setup. Add `@source` :

```css
@import "tailwindcss";

@source "./app/[[]**[]]/**/*.{js,ts,jsx,tsx,mdx}";
```

## Example 9 : Turbopack Workaround for Arbitrary Values

Component that previously used `aspect-[12/5]` :

```tsx
// app/hero/page.tsx
export default function Hero() {
  return (
    <section
      className="w-full bg-zinc-900"
      style={{ aspectRatio: "12/5" }}
    >
      <h1 className="text-5xl font-bold text-white">Wide hero</h1>
    </section>
  );
}
```

Alternative : safelist the exact arbitrary-value token in CSS so the
scanner emits it regardless of Turbopack incremental scan :

```css
@import "tailwindcss";

@source inline("aspect-[12/5] z-[100] h-[80vh]");
```

After safelisting, the original `className="aspect-[12/5]"` works again.

## Example 10 : Server Component With Tailwind (No `"use client"`)

```tsx
// app/products/[id]/page.tsx
async function fetchProduct(id: string) {
  const res = await fetch(`https://api.example.com/products/${id}`, {
    next: { revalidate: 60 },
  });
  return res.json();
}

export default async function ProductPage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  const product = await fetchProduct(id);

  return (
    <article className="mx-auto max-w-2xl px-6 py-12">
      <h1 className="text-3xl font-semibold tracking-tight">{product.name}</h1>
      <p className="mt-4 text-zinc-600">{product.description}</p>
      <p className="mt-6 text-2xl font-bold text-emerald-600">
        ${product.price}
      </p>
    </article>
  );
}
```

No `"use client"`. Server-rendered. Tailwind classes apply identically.
Zero JS shipped for styling.
