---
pos: 4
title: Filtering with variables
description: 
stackblitz: saleor-tutorial-filtering-variables
prev:
  path: /product/basic-filtering/
next:
  path: /product/sorting/
---

Often the filtering criteria come from the users. Those criteria cannot be put as-is into the GraphQL query. We must dynamically set them using the query variables.

Let's rewrite the previous query that fetches T-shirts, so that the `filter` argument is set via a GraphQL query variable:

```graphql{1,2}
query FilterProducts($filter: ProductFilterInput!) {
  products(first: 12, channel: "default-channel", filter: $filter) {
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

In the example above, we are setting the `filter` argument using the `$filter` variable with a precisely defined type of `ProductFilterInput`. The exclamation mark signifies that query variable definition is *required*. 

Notice also that we change the name of the query to `FilterProducts`. This will generate the `useFilterProductsQuery` React Hook.

Such defined GraphQL query accepts its input via the `variables` field. This transformation is done automatically by the Apollo library along with the code generation and available as input in its React Hook.

In `components/ProductCollection.tsx`, we can replace the `useFetchTwelveProductsQuery` with the newly generated `useFilterProductsQuery` as the shape of the elements in the product collection doesn't change.

```tsx{4,12-16}
// components/ProductCollection.tsx
import React from 'react';

import { Product, useFilterProductsQuery } from '@/saleor/api';
import { ProductElement } from '@/components';

const styles = {
  grid: 'grid gap-4 grid-cols-4',
}

export const ProductCollection = () => {
  const { loading, error, data } = useFilterProductsQuery({
    variables: {
      filter: { search: 'T-Shirt' }
    }
  });

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

Thanks to GraphQL variables we can parametrize our queries directly from the React components. 

Here's the result for filtering with `search` set to `T-Shirt`:

**IMAGE**