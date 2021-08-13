---
pos: 3
title: GraphQL Client in Next.js
description:
prev:
  path: /setup/saleor-graphql-playground/
next:
  path: /setup/tailwind-css-styling/
---

As GraphQL returns JSON data over HTTP you can use any HTTP client to integrate with a GraphQL endpoint. It is, however, easier to use a dedicated library that comes with some additional features already configured such as in-memory cache, error handling etc.

You need to configure a GraphQL client in our Next.js application so that we can send queries and execute mutation against the Saleor API. In this tutorial we will be using Apollo, one of the most popular GraphQL clients for JavaScript. Since we are writing TypeScript, we will also leverage the GraphQL Code Generator library. This approach will allow us to automatically generate the TypeScript types for our application from the Saleor API schema.

## Configure Apollo Client

Install Apollo Client:

```
npm install @apollo/client graphql
```

Let's start by creating a client instance that has to configuration to connect with the Saleor GraphQL API. This client must be added to a component called `ApolloProvider`, that wraps the application with the connection to the GraphQL endpoint. You wrap your application with the client provider on the highest level in the component tree so that it's available for any child component. 

In our case, we will use `pages/_app.js`: 

```tsx
import type { AppProps } from 'next/app'

import { ApolloProvider, ApolloClient, InMemoryCache } from '@apollo/client';

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

The `ApolloClient` instance requires a `uri` pointing to a Saleor API and `cache` that in this case will be simply in-memory.

## Location for GraphQL Statements

We will put all GraphQL statements (queries, mutations and fragments) inside the `graphql/` directory `queries.graphql`. Later on, we may decide to split it so that each query or mutation is being kept seperate. 

Let's start with one of the simplest GraphQL queries in Saleor API i.e. getting names of first 10 products from our store. We call this query `First10ProductNames` and store it in `graphql/queries.graphql`

```graphql
query First10ProductNames {
  products(first: 10, channel: "default-channel") {
    edges {
      node {
        name
      }
    }
  }
}
```

##  Autogenerate React.js Hooks

Let's start by configuring the [GraphQL Code Generator](https://www.graphql-code-generator.com/) library to automatically generate types representing all *the commerce notions* available in the Saleor API. This library connects to a GraphQL endpoint and locates the API schema to execute the proess of generating corresponding TypeScript types.

Install the GraphQL Code Generator command-line tool along with few plugins, namely the Apollo GraphQL integration:

```
npm -D @graphql-codegen/cli \ 
  @graphql-codegen/introspection \
  @graphql-codegen/typescript \
  @graphql-codegen/typescript-operations \
  @graphql-codegen/typescript-react-apollo
```

The Apollo integration allows us to automatically generate React.js Hooks from GraphQL queries and mutations. We will be using this feature a lot in this tutorial.

Finally, let's put it all together using the `codegen.yml` configuration file that should be stored in the root of our project:

```yaml
overwrite: true
schema: "https://tutorial.saleor.cloud/graphql/"
documents: "graphql/**/*.{ts,graphql}"
generates:
  saleor/api.tsx:
    plugins:
      - "typescript"
      - "typescript-operations"
      - "typescript-react-apollo"
      - "typescript-apollo-client-helpers"
    config:
      dedupeOperationSuffix: true 
  ./graphql.schema.json:
    plugins:
      - "introspection"
```

This configuration files instructs the code generator to use the `graphql/` directory as the place where GraphQL statements are stored and it generates types in `saleor/api.tsx`.

We can now try to run the generation process using the following command:

```bash
npx graphql-codegen --config codegen.yml 
```

For convenience, we can put this command in `package.json` under the `scripts` section as `generate`

```json
{
  ...
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "generate": "graphql-codegen --config codegen.yml -w"
  }
}
```

Now we can run it as simply as this:

```
npm run generate
```

Once the command executes, you should have a relatively large file generated at `saleor/api.tsx`. We will be using it frequently to 





