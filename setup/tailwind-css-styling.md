---
pos: 5
title: Tailwind CSS for Styling 
description:
prev:
  path: /setup/typescript-path-aliases/
next:
  path: /setup/setup-summary/
---

We will use Tailwind CSS for the styling in this application. Let's see how we can set it up in a Next.js project.

Let's start by installing Tailwind and its dependencies, i.e. PostCSS and autoprefixer:

```
npm install -D tailwindcss postcss autoprefixer
```

Next, generate the configuration files for Tailwind and PostCSS:

```
npx tailwindcss init -p
```

In newly generated `tailwind.config.js`, configure the `purge` option with the paths to Next.js pages and React.js components. This will allow Tailwind to tree-shake unused styles in production builds:

```js
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

Finally, include Tailwind in `pages/_app.tsx`:

```tsx{5}
import type { AppProps } from 'next/app'

import { ApolloProvider, ApolloClient, InMemoryCache } from '@apollo/client';

import 'tailwindcss/tailwind.css'

const client = new ApolloClient({
  uri: "https://tutorial.saleor.cloud/graphql/",
  cache: new InMemoryCache(),
});

function MyApp({ Component, pageProps }: AppProps) {
  return (
    <ApolloProvider client={client}>
      <Component {...pageProps} />
    </ApolloProvider>
  )
}
```