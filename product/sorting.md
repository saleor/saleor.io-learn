---
pos: 5
title: Sorting Products
description:
stackblitz: saleor-tutorial-sorting
prev:
  path: /product/filtering-with-variables/
next:
  path: /product/pagination/
---

The `products` query also enables us to change the order of the returned product collection through sorting. The `products` query takes the `sortBy` argument as input, and it allows you to specify which field to use for sorting along with the sorting order, i.e., _ascending_ or _descending_.

Let's rework our previous query so that it also specifies the sorting:

1. Navigate to `graphql/queries/FilterProducts.graphql` file and update the query:

```graphql{2-7}
# graphql/queries/FilterProducts.graphql
query FilterProducts($filter: ProductFilterInput!, $sortBy: ProductOrder) {
  products(
    first: 12
    channel: "default-channel"
    filter: $filter
    sortBy: $sortBy
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

As with filtering, we use the `$sortBy` variable on our `FilterProducts` query so that the sorting can be specified dynamically in a React component as input to its React hook.

2. Run the `generate` script in your Terminal or have it run in the `watch` mode, as described in the previous sections.
3. Update the `ProductCollection.tsx` component with the changes below:

```tsx{3,8-11}
// components/ProductCollection.tsx
...
import {
  Product,
  useFilterProductsQuery,
  OrderDirection,
  ProductOrderField
} from '@/saleor/api';
...
  const { loading, error, data } = useFilterProductsQuery({
    variables: {
      filter: { search: 'juice' },
      sortBy: {
        field: ProductOrderField.Name,
        direction: OrderDirection.Desc
      }
    }
  });
...
```

In the example above, we have the product name as the field for sorting, along with the _descending_ direction. That means we will be using the lexicographic order, and as a result, the products will be reversed:

![Filtered products in the reversed order.](/images/sort-product.png)
