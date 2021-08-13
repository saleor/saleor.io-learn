---
pos: 2
title: Fetching Products 
description: 
stackblitz: saleor-tutorial-start 
github:
---

In Saleor, you can fetch:
- a collection of products by using the `products` query, or
- a single product by using the `product` query that requires the product `ID` as input. 

The former operation also allows to filter and sort a collection of products by specifing either the filter criteria, or the sorting parameters.

## Fetching a collection 

Let's start with a simple collection of 12 elements that returns a product `id` and `name`. Put the following GraphQL query inside the GraphQL Playground of your Saleor endpoint and execute it using the play button in the middle. 

```graphql
query FetchTwelveProducts {
  products(first: 12, channel: "default-channel") {
    edges {
      node {
        id
        name
      }
    }
  }
}
```

<Notice>
It's a good practice to always name your GraphQL queries even if it's not required by the GraphQL engine as it helps with caching and debugging. 
</Notice>

The `first` parameter specifies how many elements we want to fetch from the Saleor API. This parameter is required to avoid fetching too much data in case we have many products in our backend. 

Here's how a typical response to that query may look like:

```json
{
  "data": {
    "products": {
      "edges": [
        {
          "node": {
            "id": "UHJvZHVjdDo3Mg==",
            "name": "Apple Juice"
          }
        },
        {
          "node": {
            "id": "UHJvZHVjdDo3NA==",
            "name": "Banana Juice"
          }
        },
        // ... 10 more
      ]
    }
  }
}
```

Another way of controlling the number of elements to fetch is through pagination. We will discuss it at the end of this section.

## Autogenerate Products React.js Hook

Now we can put this query in our application under the `graphql/` directory as `FetchTwelveProducts.graphql`. 

Run `generate` to generate the corresponding React.js hooks:

```
npm run generate
```

<Notice>
The `generate` script we configured in the setup section uses the `-w` option that instructs the code generation to keep running and watch for changes in GraphQL files. You can leave it running in a terminal for convenience.
</Notice>

## The `ProductCollection` component

Let's create our first React.js components. Call it `ProductCollection.tsx` and place the file in `components/`: 

```tsx
import React from 'react';
import Link from 'next/link';
import { useFetchTwelveProductsQuery } from '@/saleor/api';

export const ProductCollection = () => {
  const { loading, error, data } = useFetchTwelveProductsQuery();

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error</p>;

  if (data) {
    const products = data.products?.edges || [];

    return (
      <div>
        <ul role="list" className="grid gap-4 grid-cols-4">
          {products?.length > 0 &&
            products.map(
              ({ node: { id, name } }) => (
                <li key={id} className="relative bg-white border">
                  <a>
                    <div className="px-4 py-2 border-gray-100 bg-gray-50 border-t">
                      <p className="block text-lg text-gray-900 truncate">{name}</p>
                    </div>
                  </a>
                </li>
              ),
            )}
        </ul>
      </div>
    );
  }

  return null;
}

```

The GraphQL Code Generator alongside the Apollo plugins automatically generate the `useFetchTwelveProductsQuery` hook. 

Once the `data` is available, we display the product names as a list. It's important to notice that the product collection is available as a GraphQL edge as it stores information that goes beyond what's in each object, e.g. the count, and then each collection element is a node.

## Displaying product thumbnails

Right now our product collection contains only the product names. Let's improve it by adding the product thumbnails. We need to slightly modify our GraphQL query to also include the thumbnails in the response. While we are at it, we can also fetch the product category.

Here's the modified query:

```graphql
query FetchTwelveProducts {
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
```

Let's use this query to replace the previous one inside the `FetchTwelveProducts.graphql` that is located in `graphql/`.

Modify the `ProductCollection.tsx` component to display the product thumbnails along with their categories:

```tsx
import React from 'react';
import Link from 'next/link';
import { useFetchTwelveProductsQuery } from '@/saleor/api';

export const ProductCollection: React.VFC = () => {
  const { loading, error, data } = useFetchTwelveProductsQuery();

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error</p>;

  if (data) {
    const products = data.products?.edges || [];

    return (
      <div>
        <ul role="list" className="grid gap-4 grid-cols-4">
          {products?.length > 0 &&
            products.map(
              ({ node: { id, name, thumbnail, category } }) => (
                <li key={id} className="relative bg-white border">
                  <a>
                    <div className="aspect-h-1 aspect-w-1">
                      <img src={thumbnail?.url} alt="" className="object-center object-cover" />
                    </div>
                    <div className="px-4 py-2 border-gray-100 bg-gray-50 border-t">
                      <p className="block text-lg text-gray-900 truncate">{name}</p>
                      <p className="block text-sm font-medium text-gray-500">{category?.name}</p>
                    </div>
                  </a>
                </li>
              ),
            )}
        </ul>
      </div>
    );
  }

  return null;
}
```

## The `ProductElement` component

Before we dive into other Saleor features, let's take a moment to slightly refactor the `ProductCollection` component. We can take each product element in the list and put it into a separate component. 

Call this component `ProductElement` and put it in `components/`

```tsx
export const ProductElement: React.VFC = ({ id, name, thumbnail, category }) => {
  return (
    <li key={id} className="relative bg-white border">
      <a>
        <div className="aspect-h-1 aspect-w-1">
          <img src={thumbnail?.url} alt="" className="object-center object-cover" />
        </div>
        <div className="px-4 py-2 border-gray-100 bg-gray-50 border-t">
          <p className="block text-lg text-gray-900 truncate">{name}</p>
          <p className="block text-sm font-medium text-gray-500">{category?.name}</p>
        </div>
      </a>
    </li>
  )
}
```

Now we can reorganize the `ProductCollection` component in the following way:

```tsx
import React from 'react';
import Link from 'next/link';
import { useFetchTwelveProductsQuery } from '@/saleor/api';
import { ProductElement } from '@/components';

export const ProductCollection: React.VFC = () => {
  const { loading, error, data } = useFetchTwelveProductsQuery();

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error</p>;

  if (data) {
    const products = data.products?.edges || [];

    return (
      <div>
        <ul role="list" className="grid gap-4 grid-cols-4">
          {products?.length > 0 &&
            products.map(({ node }) => <ProductElement {...node} />)
          }
        </ul>
      </div>
    );
  }

  return null;
}
```