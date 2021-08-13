---
pos: 4
title: Filtering with variables
description: 
stackblitz: saleor-tutorial-filtering-variables
---

Often the filtering criteria come from the users. They cannot be put as-is into the GraphQL query, but they must be dynamically set via the query arguments.

Let's rewrite the previous query that fetches T-shirts, so that the `filter` field is set via the GraphQL query argument:

```graphql
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

Such GraphQL query accepts its input via the `variables` field. This transformation is done automatically by the Apollo library along with the code generation and available as input in its React.js hook.

```js
const { loading, error, data } = useFilterProductsQuery({
  variables: {
    filter: { search: 'T-Shirt' }
  }
});
```