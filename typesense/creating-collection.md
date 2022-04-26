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

`yarn add --dev typesense`
`yarn add --dev @babel/runtime`

## Creating the main function

In this step, in the root folder of our app, we will create a folder called `scripts` and inside it we will create a new file called `populate.ts`. In this file we will build a function that fetches data from a GraphQL API and converts it into a Typesense index, in the format specific to Typesense.
In `populate.ts` write the following lines of code:

```javascript
//@ts-ignore
const main = async () => {};
main();
```

## Setting up Apollo Client

We will use [Apollo client](https://www.apollographql.com/docs/react/) to fetch data from the API using GraphQL. Let us instantiate the `apolloClient` object right above the main function.

```javascript
const apolloClient = new ApolloClient({
  link: new HttpLink({ uri: "https://vercel.saleor.cloud/graphql/", fetch }),
  cache: new InMemoryCache(),
  ssrMode: true,
});

const main = async () => {};
main();
```

The `ApolloClient` constructor takes in a `HttpLink` object with specified custom fetch function (in this case we use the ‘_cross-fetch_’ library which is able to run in the Node environment), a caching mechanism, and SSR mode which prevents Apollo Client from refetching queries unnecessarily.
Then, add the necessary imports:

```javascript
import { ApolloClient, HttpLink, InMemoryCache } from "@apollo/client";
import fetch from "cross-fetch";
```

## Getting data from Saleor API

Still in `populate.ts` file, inside our `main()` function let us grab some data using the newly created `apolloClient` object.

```javascript
const main = async () => {
  const data: ApolloQueryResult<LatestProductsQuery | undefined> =
    await apolloClient.query({
      query: LatestProductsDocument,
    });
};
```

The `LatestProductsQuery` type, `LatestProductsDocument` query as well as all the other types and hooks related to the products GraphQL schema have been automatically generated using [GraphQL Code Generator](https://www.graphql-code-generator.com/).
Now, based on our `LatestProductsDocument` GraphQL query, which we can inspect below:

```javascript
export const LatestProductsDocument = gql`
  query LatestProducts {
    products(first: 12, channel: "default-channel") {
      edges {
        node {
          id
          name
          thumbnail {
            url
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

we may start extracting the latest products using `map()` . Underneath the `apolloClient` query type in:

```javascript
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

Once you type price and category you will notice that Typescript yields these as `undefined`. That’s because we haven’t included these fields in our GraphQL query. Let’s fix that. In the `LatestProductCollection.graphql` file change the query to:

```js
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

Then, in your terminal run:

    yarn generate

which will regenerate all the types and hooks for your new GraphQL schema. This will give us a new `LatestProductsDocument`:

```javascript
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

Now, all the errors should be dealt with and we should have our products extracted properly. So far, our `main()` function should look like that:

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
Still in the `main()` function, let’s type in:

```javascript
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

In order to create a search collection, we need to transform data fetched from the GraphQL query into something that Typesense can work with. Let us then create a `productsSchema` inside the `main()`:

```javascript
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

Bear in mind that we have defined `productsSchema` with the exact same fields as our `product` node coming from Saleor API and we have assigned every field a proper type.
Once we have the schema ready, we can start creating our Typesense collection.

```javascript
await typesenseClient.collections().create(productsSchema);
```

Now, we can insert it into the server:

```javascript
try {
  if (products) {
    await typesense.collections("products").documents().import(products);
  }
} catch (e) {
  console.log(e);
}
```

Our `main()` function is complete now and it should look as follows:

```javascript
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

```javascript
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

Our code is ready and now it is time to run it. Go to `package.json` file and within `"scripts"` add another command called `"populate"`, which will transpile `populate.ts` into a plain javascript file and run it in Node.

    "populate": "cd scripts && tsc populate.ts && node populate.js"

Then, run the script in your terminal:

    yarn run populate

Unfortunately, you may encounter an error saying that “–jsx” flag is not set:
![](https://lh3.googleusercontent.com/cTx7byCnURQPzHn4bBRuFeU3j4exiRBSG-MbDYZNGmxzJsIIN5uElZkNRGoO9jCyOkmYR5xUpB3FUoMtZSYxcnqXT395Q88DkJ4ig0KJG2uT1R8RmJEVH-wrB2Qm0Nso7CWxlStz)

One of the ways to get rid of this error is to simply change the `saleor.tsx` file to `saleor.ts`

After running `yarn populate` again we should have everything set up.
You may now check the Typesense Cloud account and inspect the newly created collection.
