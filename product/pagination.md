---
pos: 6
title: Pagination 
description: 
stackblitz: saleor-tutorial-pagination 
---

The `products` query returns a paginated collection.  This means that once we receive a result of a query, we can ask for the following elements. In order to keep track of the position in the collection we need a pointer, usually known as a cursor. This means that a query must return such cursor once it returns so that we know how to ask for another subset. It’s not a good idea to keep this information in the entity itself as it’s not related to the business model. For that we introduce a layer of indirection. In GraphQL this layer is usually known as `edges`.

Let's adapt the query for fetching products so that it can be paginated.

```graphql
query Products($before: String, $after: String) {
  products(first: 4, channel: "default-channel", after: $after, before: $before) {
    edges {
      cursor
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

Now, we are not only returing the product `node`, but also the `cursor` that uniqely identifies that product item. The `products` query takes the `after` and `before` arguments that takes the product cursor as value. 

The pagination process consists of passing the last product that is displayed as `after` when paginating forward, and as `before` when paginating backward. Here's a React.js snippet that implements that behavior.

```js
const [before, setBefore] = useState('');
const [after, setAfter] = useState('');
const { loading, error, data } = useProductsQuery({
  variables: { after, before }
});
```

In our React.js component, we must connect the pagination buttons accordingaly:

```tsx
<div>
  <a href="#" onClick={() => setBefore(latestProducts[0].cursor || '')}>Prev</a>
  <a href="#" onClick={() => setAfter(latestProducts[latestProducts.length - 1].cursor || '')}>Next</a>
</div>
```