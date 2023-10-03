---
pos: 2
title: Creating Typesense Collection
description:
prev:
  path: /typesense/overview/
next:
  path: /typesense/creating-search-ui/
---

## Prerequisites

Typesense provides us with a few options when it comes to setting up the search server. In this tutorial we are using the Typesense Cloud server for storing collections. So, before getting started with crafting the app, you need to [create an account](https://cloud.typesense.org/) at Typesense Cloud. It is totally free for a month. However, it is also possible to set up a local instance of the Typesense server. You may find the installation instructions in [the docs](https://typesense.org/docs/guide/install-typesense.html#option-1-typesense-cloud).

## Installing Typesense

So far, our application displays a number of products, which are fetched using the Apollo’s `useLatestProductsQuery()` hook. Now, let us add [Typesense.js](https://github.com/typesense/typesense-js) to our project, so that we can start creating a real search experience for the users.
In your terminal, type in:

`pnpm add -D typesense`
`pnpm add -D @babel/runtime`

## Creating the main function

In this step we will scaffold a function that fetches data from a GraphQL API and converts it into a Typesense index, in the format specific to Typesense.

1. In the root folder of our app, create a folder called `scripts`.
2. Inside `scripts` create a new file called `populate.ts`.
3. In `populate.ts` write the following lines of code:

```js
//@ts-ignore
const main = async () => {};
main();
```

## Setting up Apollo Client

We will use [Apollo client](https://www.apollographql.com/docs/react/) to fetch data from the API using GraphQL.

1. Right above the main function instantiate the `apolloClient` object:

```js
const apolloClient = new ApolloClient({
  link: new HttpLink({ uri: "https://vercel.saleor.cloud/graphql/", fetch }),
  cache: new InMemoryCache(),
  ssrMode: true,
});

const main = async () => {};
main();
```

The `ApolloClient` constructor takes in a `HttpLink` object with specified custom fetch function (in this case we use the ‘_cross-fetch_’ library which is able to run in the Node environment), a caching mechanism, and SSR mode which prevents Apollo Client from refetching queries unnecessarily.

2.  Add the necessary imports:

```js
import { ApolloClient, HttpLink, InMemoryCache } from "@apollo/client";
import fetch from "cross-fetch";
```

## Getting data from Saleor API

Let us grab some data using the newly created `apolloClient` object.

1. In `populate.ts` file, inside our `main()` function, type in:

```javascript
const main = async () => {
  const data: ApolloQueryResult<LatestProductsQuery | undefined> =
    await apolloClient.query({
      query: LatestProductsDocument,
    });
};
```

The `LatestProductsQuery` type, `LatestProductsDocument` query as well as all the other types and hooks related to the products GraphQL schema have been automatically generated using [GraphQL Code Generator](https://www.graphql-code-generator.com/).

2. Underneath the `apolloClient` query type in:

```js
const products = data.data?.products?.edges.map(({ node }) => {
  return {
    name: node.name,
    description: node.description,
    imageSrc: node.thumbnail?.url,
    category: node.category?.name,
    price: node.pricing?.priceRange?.start?.gross.amount,
  };
});
```

3. In the `LatestProductCollection.graphql` include missing fields in the GraphQL query:

```graphql
query LatestProducts {
  products(first: 12, channel: "default-channel") {
    edges {
      node {
        id
        name
        description
        thumbnail {
          url
        }
        pricing {
          priceRange {
            start {
              gross {
                amount
                }
              }
            }
          }
        category {
          name
        }
      }
    }
  }
  }
};
```

4. In your terminal run:

`pnpm generate`

The command will regenerate all the types and hooks for your new GraphQL schema. This will give us a new `LatestProductsDocument`:

```js
export const LatestProductsDocument = gql`
  query LatestProducts {
    products(first: 12, channel: "default-channel") {
      edges {
        node {
          id
          name
          description
          thumbnail {
            url
          }
          pricing {
            priceRange {
              start {
                gross {
                  amount
                }
              }
            }
          }
          category {
            name
          }
        }
      }
    }
  }
`;
```

So far, our `main()` function should look like that:

```js
const main = async () => {
  const data: ApolloQueryResult<LatestProductsQuery | undefined> =
    await apolloClient.query({
      query: LatestProductsDocument,
    });
  const products = data.data?.products?.edges.map(({ node }) => {
    return {
      name: node.name,
      description: node.description,
      imageSrc: node.thumbnail?.url,
      category: node.category?.name,
      price: node.pricing?.priceRange?.start?.gross.amount,
    };
  });
};
main();
```

Let us move on to setting up Typesense.

## Connecting to Typesense Cloud

Once we have our products array, it is time to populate Typesense server with our data. In order to do that we need to set up the Typesense Client.
Still in the `main()` function, type in:

```js
const typesenseClient = new Typesense.Client({
  nodes: [
    {
      host: "xxx.a1.typesense.net", // fill-in with your own info
      port: 443,
      protocol: "https",
    },
  ],
  apiKey: "YOUR ADMIN API KEY", //fill-in with your own info
  connectionTimeoutSeconds: 2,
});
```

After signing into Typesense Cloud you’ll have the possibility to create the necessary credentials under the Create API Key button inside the API keys tab.

## Creating and importing collections

In order to create a search collection, we need to transform data fetched from the GraphQL query into a shape that Typesense can work with.

1. Create a `productsSchema` inside the `main()`:

```js
let productsSchema: CollectionCreateSchema = {
  name: "products",
  fields: [
    { name: "name", type: "string" },
    { name: "description", type: "string" },
    { name: "category", type: "string", facet: true },
    { name: "price", type: "float" },
    { name: "imageSrc", type: "string" },
  ],
};
```

Bear in mind that we have defined `productsSchema` with:

- the exact same fields as our `product` node coming from Saleor API
- a proper type for every field.

2. Create the Typesense collection.

```js
await typesenseClient.collections().create(productsSchema);
```

3. Index the products into the collection:

```js
try {
  if (products) {
    await typesense.collections("products").documents().import(products);
  }
} catch (e) {
  console.log(e);
}
```

Our `main()` function is complete now and it should look as follows:

```jsx
//@ts-ignore
const main = async () => {
  const data: ApolloQueryResult<LatestProductsQuery | undefined> =
    await apolloClient.query({
      query: LatestProductsDocument,
    });
  const products = data.data?.products?.edges.map(({ node }) => {
    return {
      name: node.name,
      description: node.description,
      imageSrc: node.thumbnail?.url,
      category: node.category?.name,
      price: node.pricing?.priceRange?.start?.gross.amount,
    };
  });

  const typesenseClient = new Typesense.Client({
    nodes: [
      {
        host: "xxx.a1.typesense.net", // fill-in with your own info
        port: 443,
        protocol: "https",
      },
    ],
    apiKey: "YOUR ADMIN API KEY", //fill-in with your own info
    connectionTimeoutSeconds: 2,
  });

  let productsSchema: CollectionCreateSchema = {
    name: "products",
    fields: [
      { name: "name", type: "string" },
      { name: "description", type: "string" },
      { name: "category", type: "string" },
      { name: "price", type: "float" },
      { name: "imageSrc", type: "string" },
    ],
  };

  await typesenseClient.collections().create(productsSchema);

  try {
    if (products) {
      await typesenseClient
        .collections("products")
        .documents()
        .import(products);
    }
  } catch (e) {
    console.log(e);
  }
};
```

And we should have all the imports as follows:

```js
import {
  ApolloClient,
  ApolloQueryResult,
  HttpLink,
  InMemoryCache,
} from "@apollo/client";
import {
  LatestProductsDocument,
  LatestProductsQuery,
} from "../generated/saleor";
import fetch from "cross-fetch";
import Typesense from "typesense";
import { CollectionCreateSchema } from "typesense/lib/Typesense/Collections";
```

## Running the `populate.ts` script

Our code is ready and now it is time to run it.

1. Go to `package.json` file and within `"scripts"` add another command called `"populate"`.

`"populate": "cd scripts && tsc populate.ts && node populate.js"`

This command will transpile `populate.ts` into a plain javascript file and run it in Node.

2. To void the error saying that “–jsx” flag is not set, change the `saleor.tsx` file to `saleor.ts`.

3. Run the script in your terminal:

`pnpm populate`

You may now check the Typesense Cloud account and inspect the newly created collection.
