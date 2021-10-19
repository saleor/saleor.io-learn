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


Such defined GraphQL query accepts its input via the `variables` field. This transformation is done automatically by the Apollo library along with the code generation and available as input in its React.js hook.

```js
const { loading, error, data } = useFilterProductsQuery({
  variables: {
    filter: { search: 'T-Shirt' }
  }
});
```

Thanks to GraphQL variables we can parametrize our queries directly from the React components. 

Here's the result for filtering with `search` set to `T-Shirt`:

**IMAGE**