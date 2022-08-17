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

Often the filtering criteria come from the users. We cannot put these criteria as-is into the GraphQL query. We must dynamically set them using the query variables.

1. Go to the `graphql/queries` folder and create a `ProductFilterByName.graphql` file. Copy/paste the query below:

```graphql{1,2}
# graphql/queries/ProductFilterByName.graphql
query ProductFilterByName($filter: ProductFilterInput!) {
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

In the example above, we are setting the `filter` argument using the `$filter` variable with a precisely defined type of `ProductFilterInput`. The exclamation mark signifies that the query variable definition is _required_.

2. In the Terminal, run:

```
npm run generate
```

or

```
pnpm run generate
```

It will generate the `useProductFilterByNameQuery` React Hook.

Such defined GraphQL query accepts its input via the `variables` field. This transformation is done automatically by the Apollo library along with the code generation and is available as input in its React Hook.

3. In `components/ProductCollection.tsx`, replace the `useProductFetchTwelveElementsQuery` with the newly generated `useProductFilterByNameQuery` as the shape of the elements in the product collection doesn't change.

```tsx{4,12-16}
// components/ProductCollection.tsx
import React from 'react';

import { Product, useProductFilterByNameQuery } from '@/saleor/api';
import { ProductElement } from '@/components';

const styles = {
  grid: 'grid gap-4 grid-cols-4',
}

export const ProductCollection = () => {
  const { loading, error, data } = useProductFilterByNameQuery({
    variables: {
      filter: { search: 'juice' }
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

Here's the result for filtering with `search` set to `juice`:

![Search filter in action.](/images/filter-product.png)
