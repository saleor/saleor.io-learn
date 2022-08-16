---
pos: 3
title: Basic filtering
description:
stackblitz: saleor-tutorial-basic-filtering
prev:
  path: /product/fetching-products/
next:
  path: /product/filtering-with-variables/
---

When fetching products using the `products` query, we can also filter them using various criteria provided via the `filter` argument.

The most basic form of filtering can be done using the `search` field. It takes a string value as input that is matched against the product title and description. In the following example, we want to find all the t-shirts available in our store.

```graphql
# graphql/queries/ProductSearchTShirt.graphql
query ProductSearchTShirt {
  products(
    first: 12
    channel: "default-channel"
    filter: { search: "t-shirt" }
  ) {
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

Let's put this query as `ProductSearchTShirt.graphql` in the `graphql/queries` directory.

If the code generation is running in the watch mode, the corresponding React.js hook will be created automatically. Otherwise, run the `generate` script:

```
npm run generate
```

or

```
pnpm generate
```

In `components/ProductCollection.tsx`, we can replace the `useProductGetTwelveElementsQuery` with the newly generated `useProductSearchTShirtQuery` as the shape of the elements in the product collection doesn't change.

```tsx{4,12}
// components/ProductCollection.tsx
import React from 'react';

import { Product, useProductSearchTShirtQuery } from '@/saleor/api';
import { ProductElement } from '@/components';

const styles = {
  grid: 'grid gap-4 grid-cols-4',
}

export const ProductCollection = () => {
  const { loading, error, data } = useProductSearchTShirtQuery();

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error</p>;

  if (data) {
    const products = data.products?.edges || [];

    return (
      <ul role="list" className={styles.grid}>
        {products?.length > 0 &&
          products.map(
            ({ node }) => <ProductElement key={node.id} {...node as Product} />,
          )}
      </ul>
    );
  }

  return null;
}
```

As a result of the search, we will render the T-shirts only.
![The results of product search.](/images/product-search.png)
