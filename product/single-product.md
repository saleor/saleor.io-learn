---
pos: 7
title: Fetching single product 
description: 
---

Sometimes you may need to fetch a specific product. Usually, you display the most important product information on the collection page, and then when the user navigates to a specific product you provide details about it.  We can use the `product` query for fetching those details.

## GraphQL Query for Single Product

Let's call our query `ProductByID` to make it explicit:

```graphql
query ProductByID($id: ID!) {
  product(id: $id, channel: "default-channel") {
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
```

The `product` query requires a value for the `id` argument, which is the product we want to fetch. We can then use the generated `useProductByIdQuery` hook in React.js components with the product `id` specified via the `variables` as shown below:

```js
const { loading, error, data } = useProductByIdQuery({ 
  variables: { 
    id: 'UHJvZHVjdDoxMTE=' 
  } 
});
```

## React Component for Single Product

## Page per Product
