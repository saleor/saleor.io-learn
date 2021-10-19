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

The `products` query allows us to the order in the returned product collection with sorting. The `products` query takes the `sortBy` argument as input and it allows to specify the field that will be used for sorting along with the sorting order, i.e. *ascending* or *descending*.

Let's rework our previous query so that it also specifies the sorting:

```graphql{1,2}
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

As with filtering, we use the `$sortBy` variable on our `FilterProducts` query so that the sorting can be specified dynamically in a React component as input to its React hook.

```js{4-7}
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

In the example above, we are using the product name as the field for sorting along with the *descending* direction. That means we will be using the lexicographic order and as a result, the products will be reversed.