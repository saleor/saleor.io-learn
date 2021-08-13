---
pos: 5
title: Sorting Products
description: 
stackblitz: saleor-tutorial-sorting
---

The order in the returned product collection can modified. The `products` query takes `sortBy` as input and it allows to specify the field that will be used for sorting along with the sorting order, i.e. *ascending* or *descending*.

Let's rework our previous query so that it also specifies the sorting:

```graphql
query FilterProducts($filter: ProductFilterInput!, $sortBy: ProductOrder) {
  products(first: 12, channel: "default-channel", filter: $filter, sortBy: $sortBy) {
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

As with filtering, we use the `$sortBy` argument on our `FilterProducts` query so that the sorting can be specified dynamically in React.js as input to its hook.

```js
const { loading, error, data } = useFilterProductsQuery({
  variables: {
    filter: { search: 'T-Shirt' },
    sortBy: {
      field: ProductOrderField.Name,
      direction: OrderDirection.Desc
    }
  }
});
```

In this example, we are using the product name as the field for sorting along with the descending direction. As a result, the order of products will be reversed.