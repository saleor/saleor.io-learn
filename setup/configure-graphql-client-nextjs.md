---
pos: 3
title: GraphQL Client in Next.js
description:
prev:
  path: /setup/saleor-graphql-playground/
next:
  path: /setup/graphql-code-generator/
---

As GraphQL returns JSON data over HTTP, you can use any HTTP client to integrate with a GraphQL endpoint. It is, however, easier to use a dedicated library that comes with some additional features already configured, such as in-memory cache, error handling, etc.

You need to configure a GraphQL client in our Next.js application in order to send queries and execute mutation against the Saleor API. In this tutorial, we will be using Apollo, one of the most popular GraphQL clients for JavaScript. Since we are writing TypeScript, we will also leverage the GraphQL Code Generator library. This approach will allow us to automatically generate the TypeScript types for our application from the Saleor API schema.

## Configure Apollo Client

Install Apollo Client:

```
npm install @apollo/client graphql
```

```
pnpm add @apollo/client graphql
```

Let's start by creating an Apollo client instance that points to the Saleor GraphQL API. In this tutorial, we will use the following API endpoint provided by the Saleor Cloud. Feel free to change it to your own Saleor server instance.


```
https://tutorial.saleor.cloud/graphql/
```

The GraphQL client will be provided to our application via the `ApolloProvider` component that wraps the application with the connection to the GraphQL endpoint. We wrap our application with the Apollo provider at the highest level in the component tree so that it's available for any child component below. 

In our case, this will be `pages/_app.tsx`: 

```tsx
import type { AppProps } from 'next/app'

import { ApolloProvider, ApolloClient, InMemoryCache } from '@apollo/client';

import '../styles/globals.css'

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

The `ApolloClient` instance requires a `uri` pointing to a Saleor API endpoint and `cache` that in this case will be simply in-memory.
