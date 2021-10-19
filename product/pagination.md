---
pos: 6
title: Pagination 
description: 
stackblitz: saleor-tutorial-pagination 
prev:
  path: /product/sorting/
next:
  path: /product/single-product/
---

The `products` query returns a paginated collection. This means that once we receive the query result, we also have data to ask for the next elements in that collection. In order to keep track of the position in the collection we need a pointer, usually known as a cursor. The query result must provide such cursor in the response once it returns so that we can ask for another subset of data. 

It’s not a good idea to keep this information in the entity itself as it’s not related to the business model. For that we introduce a layer of indirection. In GraphQL this layer is usually known as `edges`.

Let's adapt the query for fetching products so that it can be paginated.

```graphql{2,4}
query Products($before: String, $after: String) {
  products(first: 4, channel: "default-channel", after: $after, before: $before) {
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
      pageInfo {
        hasNextPage
        hasPreviousPage
        startCursor
        endCursor
      }
    }
  }
}
```

We are not only returing the product `node`, but also the `pageInfo` with `hasNextPage` / `hasPreviousPage` helpers to see if there are elements after and before the current collection subset along with `startCursor` / `endCursor` that uniqely identify the current subset of product collection. 

Additionally, the `products` query has the `after` and `before` arguments that take the value of `startCursor` and `endCursor` as input. 

```tsx
import React from "react";

import { ProductFilterInput, useProductCollectionQuery } from "@/saleor/api";

import { Pagination } from "./Pagination";
import { ProductElement } from "./ProductElement";

export const ProductCollection: React.VFC = ({}) => {
  const { loading, error, data, fetchMore } = useProductCollectionQuery();

  const onLoadMore = () => {
    fetchMore({
      variables: {
        after: data?.products?.pageInfo.endCursor,
      },
    });
  };

  if (loading) return <p>Loading...</p> 
  if (error) return <p>Error</p>;

  const products = data?.products?.edges.map((edge) => edge.node) || [];

  if (products.length === 0) {
    return <p>No products.</p>;
  }

  return (
    <div>
      <ul
        role="list"
        className="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4"
      >
        {products.map((product) => (
          <ProductElement key={product.id} product={product} />
        ))}
      </ul>

      <Pagination
        onLoadMore={onLoadMore}
        pageInfo={data?.products?.pageInfo}
        itemCount={data?.products?.edges.length}
        totalCount={data?.products?.totalCount || undefined}
      />
    </div>
  );
};

export default ProductCollection;
```

```tsx
import { PageInfo } from "@/saleor/api";

export interface PaginationProps {
  pageInfo?: PageInfo;
  onLoadMore: () => void;
  totalCount?: number;
  itemCount?: number;
}

export const Pagination: React.VFC<PaginationProps> = ({
  pageInfo,
  onLoadMore,
  itemCount,
  totalCount,
}) => {

  if (!pageInfo || !pageInfo?.hasNextPage) {
    return <></>;
  }

  return (
    <nav className="mt-8 p-4 ">
      <div className="flex justify-center flex-col items-center">
        <a
          onClick={onLoadMore}
          className="relative inline-flex  items-center px-4 py-2 border text-sm font-medium rounded-md text-gray-700 bg-gray-50 hover:border-blue-300 cursor-pointer"
        >
          Load More
        </a>
        {itemCount && totalCount && (
          <div className="text-sm text-gray-500 mt-2">
            {itemCount} out of {totalCount}
          </div>
        )}
      </div>
    </nav>
  );
};

```

---

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