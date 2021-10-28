---
pos: 6
title: Tailwind CSS for Styling 
description:
prev:
  path: /setup/typescript-path-aliases/
next:
  path: /setup/setup-summary/
---

<p class="lead">
We will use <a href="https://tailwindcss.com/" target="_blank">Tailwind CSS</a> for the styling in this application. Let's see how we can set it up in a Next.js project.
</p>


Let's start by installing Tailwind and its dependencies, i.e. PostCSS and autoprefixer:

```
npm install -D tailwindcss postcss autoprefixer
```

```
pnpm add -D tailwindcss postcss autoprefixer
```

Next, generate the configuration files for Tailwind and PostCSS:

```
npx tailwindcss init -p
```
```
pnpm dlx tailwindcss init -p
```

In newly generated `tailwind.config.js`, configure the `purge` option with the paths to Next.js pages and React.js components. This will allow Tailwind to tree-shake unused styles in production builds:

```js{3,4}
module.exports = {
  purge: [
    './pages/**/*.{js,ts,jsx,tsx}', 
    './components/**/*.{js,ts,jsx,tsx}'
  ],
  darkMode: false, 
  theme: {
    extend: {},
  },
  variants: {
    extend: {},
  },
  plugins: [],
}
```

Next, go to `styles/main.css` and replace the file content with the following snippet:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Finally, include this main stylesheet in `pages/_app.tsx`:

```tsx{4}
import type { AppProps } from 'next/app'
import { ApolloProvider, ApolloClient, InMemoryCache } from '@apollo/client';

import '../styles/main.css';

const client = new ApolloClient({
  uri: "https://tutorial.saleor.cloud/graphql/",
  cache: new InMemoryCache(),
});

export default function MyApp({ Component, pageProps }: AppProps) {
  return (
    <ApolloProvider client={client}>
      <Component {...pageProps} />
    </ApolloProvider>
  )
}
```

With such setup, the Tailwind classes should work in any React component across the whole application.

Let's finish off with defining some custom, global styles that will be also available from any component. Using Tailwind's built-in `@apply` directive, we will define one style for links and two styles for buttons (primary & regular). 

Open `styles/main.css` and adjust it so it has the following content:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

.button {
  @apply inline-flex items-center px-4 py-2 border text-sm font-medium rounded-md text-gray-700 bg-gray-50 hover:border-blue-300 cursor-pointer;
}

.primary-button {
  @apply max-w-xs w-full bg-blue-500 border border-transparent rounded-md py-3 px-8 flex items-center justify-center text-white hover:bg-blue-600 focus:outline-none;
}

.link {
  @apply text-sm text-blue-600 hover:text-blue-500;
}
```