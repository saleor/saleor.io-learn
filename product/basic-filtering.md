---
pos: 3
title: Basic filtering 
description: 
stackblitz: saleor-tutorial-basic-filtering
---

When fetching products using the `products` query, we can also filter them using various criteria provided via the `filter` argument. 

The most basic form of filtering can be done using the `search` field. It takes a string value as input that is matched against the product title and description. In the following example we want to find all the t-shirts available in our store. 

```graphql
query TShirtProducts {
  products(first: 12, channel: "default-channel", filter: { search: "t-shirt" }) {
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

Let's put this query as `TShirtProducts.graphql` in the `graphql/` directory.

If the code generation is running in the watch mode, the corresponding React.js hook will be created automatically. Otherwise, run the `generate` script:

```
npm run generate
```

In `components/ProductCollection.tsx`, we can replace the `useFetchTwelveProductsQuery` with the newly generated `useTShirtProductsQuery` as the shape of the elements in the product collection doesn't change.

```tsx{3,5}
import React from 'react';
import Link from 'next/link';
import { useTShirtProductsQuery } from '@/saleor/api';

export const ProductCollection = () => {
  const { loading, error, data } = useTShirtProductsQuery();

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