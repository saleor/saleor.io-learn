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

1. Install Tailwind and its dependencies, i.e. PostCSS and autoprefixer:

```
npm install -D tailwindcss postcss autoprefixer
```

or

```
pnpm add -D tailwindcss postcss autoprefixer
```

2. Generate the configuration files for Tailwind and PostCSS:

```
npx tailwindcss init -p
```

or

```
pnpm dlx tailwindcss init -p
```

3. Configure your template paths.
   Add the paths to all of your template files in your `tailwind.config.js` file.

```js{3,4}
module.exports = {
  content: [
    './pages/**/*.{js,ts,jsx,tsx,mdx}',
    './components/**/*.{js,ts,jsx,tsx,mdx}',
    './ui/**/*.{js,ts,jsx,tsx,mdx}',
    './app/**/*.{js,ts,jsx,tsx,mdx}',
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

4. Go to `styles/globals.css` and replace the file content with the following snippet:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

5. Finally, make sure this global stylesheet is included in `pages/_app.tsx`:

```tsx{4}
import type { AppProps } from 'next/app'
import { ApolloProvider, ApolloClient, InMemoryCache } from '@apollo/client';

import '../styles/globals.css';

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

With such a setup, the Tailwind classes should work in any React component across the whole application.

Let's finish off by defining some custom, global styles that will be also available from any component. Using Tailwind's built-in `@apply` directive, we will define one style for links and two styles for buttons (primary & regular).

Open `styles/globals.css` and adjust it so it has the following content:

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
