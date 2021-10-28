---
pos: 2
title: Fetching Products 
description: 
stackblitz: saleor-tutorial-start 
github:
prev:
  path: /product/overview/
next:
  path: /product/basic-filtering/
---

In Saleor, you can fetch:
- a collection of products by using the `products` query, or
- a single product by using the `product` query that requires the product `ID` as input. 

The first operation also allows to filter and sort a collection of products by specifing either the filter criteria, or the sorting parameters.

## Fetching a collection 

Let's start with a simple collection of 12 elements that returns a product `id` and `name`. Put the following GraphQL query inside the GraphQL Playground of your Saleor endpoint and execute it using the play button in the middle. 

```graphql
# graphql/queries/FetchTwelveProducts.graphql
query FetchTwelveProducts {
  products(first: 12, channel: "default-channel") {
    edges {
      node {
        id
        name
      }
    }
  }
}
```

<Notice>
It's a good practice to always name your GraphQL queries even if it's not required by the GraphQL engine as it helps with caching and debugging. 
</Notice>

The `first` parameter specifies how many elements we want to fetch from the Saleor API. This parameter is required to avoid fetching too much data in case we have many products in our backend. 

Here's how a typical response to that query may look like:

```json
{
  "data": {
    "products": {
      "edges": [
        {
          "node": {
            "id": "UHJvZHVjdDo3Mg==",
            "name": "Apple Juice"
          }
        },
        {
          "node": {
            "id": "UHJvZHVjdDo3NA==",
            "name": "Banana Juice"
          }
        },
        // ... 10 more
      ]
    }
  }
}
```

Another way of controlling the number of elements to fetch is through pagination. We will discuss it at the end of this section.

## Autogenerate a React Hook for Products

Now we can put this query in our application under the `graphql/queries` directory as `FetchTwelveProducts.graphql`.

Run `generate` to generate the corresponding React.js hooks:

```
npm run generate
```

<Notice>
The `generate` script we configured in the setup section uses the `-w` option that instructs the code generation to keep running and watch for changes in GraphQL files. You can leave it running in a terminal for convenience.
</Notice>

## React Component for Product Collection

Let's create our first React component for displaying available products as a grid. Name this component `ProductCollection.tsx` and place the file in `components/`.

```tsx{3}
// components/ProductCollection.tsx
import React from 'react';
import { useFetchTwelveProductsQuery } from '@/saleor/api';

const styles = {
  grid: 'grid gap-4 grid-cols-4',
  product: {
    card: 'bg-white border',
    summary: 'px-4 py-2 border-gray-100 bg-gray-50 border-t',
    title: 'block text-lg text-gray-900 truncate',
  }
}

export const ProductCollection = () => {
  const { loading, error, data } = useFetchTwelveProductsQuery();

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error</p>;

  if (data) {
    const products = data.products?.edges || [];

    return (
      <ul role="list" className={styles.grid}>
        {products?.length > 0 &&
          products.map(
            ({ node: { id, name } }) => (
              <li key={id} className={styles.product.card}>
                <a>
                  <div className={styles.product.summary}>
                    <p className={styles.product.title}>{name}</p>
                  </div>
                </a>
              </li>
            ),
          )}
      </ul>
    );
  }

  return null;
}

```

The GraphQL Code Generator alongside the Apollo React plugin automatically generates the `useFetchTwelveProductsQuery` React hook. 

Once the `data` is available, we display the product names as a list. It's important to notice that the product collection is available as a GraphQL edge as it stores information that goes beyond what's in each object, e.g. the count, and then each collection element is a node.

Let's define the export statement in `components/index.ts` for this component, so that we can directly import it from `@/components` later on.

```tsx
// components/index.ts
export { ProductCollection } from './ProductCollection';
```

## Display the Home Page

In Next.js the routing is generated from the file system. All the pages are located in `pages` and `pages/index.tsx` is the page that will be displed for the `/` path. Right now, it's just a 4-element grid with links to Next.js resources. Let's replace it with the `ProductCollection` component:

```tsx
// pages/index.tsx
import React from 'react'

import { 
  ProductCollection 
} from '@/components';

const styles = {
  background: 'min-h-screen bg-gray-100',
  container: 'py-10 max-w-7xl mx-auto',
}

const Home = () => {
  return (
    <div className={styles.background}>
      <div className={styles.container}>
        <ProductCollection />
      </div>
    </div>
  )
}

export default Home
```

As a result when navigating to `localhost:3000/` you should see a grid of product names as shown below:

> **IMAGE**

Additionally, we can standarise the structure for each Next.js page with a fixed layout using the following component:

```tsx
// components/Layout.tsx
import React from 'react';

interface Props {
  children: React.ReactNode;
}

const styles = {
  background: 'min-h-screen bg-gray-100',
  container: 'py-10 max-w-7xl mx-auto',
}

export const Layout = ({ children }: Props) => {
  return (
    <div className={styles.background}>
      <div className={styles.container}>
        {children}
      </div>
    </div>
  );
}
```

Let's define the export statement in `components/index.ts` for this component, so that we can directly import it from `@/components` later on.

```tsx{3}
// components/index.ts
export { ProductCollection } from './ProductCollection';
export { Layout } from './Layout';
```

With `Layout` we can rewrite the `Home` page in the following way:

```tsx
// pages/index.tsx
import React from 'react'

import { 
  ProductCollection,
  Layout 
} from '@/components';

const Home = () => {
  return (
    <Layout>
      <ProductCollection />
    </Layout>
  )
}

export default Home
```

## Displaying product thumbnails

Right now our product collection contains only the product names. Let's improve it by including the product images. 

We need to slightly modify our GraphQL query to also include the product thumbnails in the response. While we are at it, we can also fetch the product category.

Here's the modified query:

```graphql{8-13}
# graphql/queries/FetchTwelveProducts.graphql
query FetchTwelveProducts {
  products(first: 12, channel: "default-channel") {
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

Let's use this query to replace the previous one inside the `FetchTwelveProducts.graphql` that is located in `graphql/`.

Modify the `ProductCollection.tsx` component to display the product thumbnails along with their categories:

```tsx
// components/ProductCollection.tsx
import React from 'react';
import { useFetchTwelveProductsQuery } from '@/saleor/api';

const styles = {
  grid: 'grid gap-4 grid-cols-4',
  product: {
    card: 'bg-white border',
    summary: 'px-4 py-2 border-gray-100 bg-gray-50 border-t',
    title: 'block text-lg text-gray-900 truncate',
    category: 'block text-sm font-medium text-gray-500',
    image: {
      aspect: 'aspect-h-1 aspect-w-1',
      content: 'object-center object-cover'
    }
  }
}

export const ProductCollection = () => {
  const { loading, error, data } = useFetchTwelveProductsQuery();

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error</p>;

  if (data) {
    const products = data.products?.edges || [];

    return (
      <ul role="list" className={styles.grid}>
        {products?.length > 0 &&
          products.map(
            ({ node: { id, name, thumbnail, category } }) => (
              <li key={id} className={styles.product.card}>
                <a>
                  <div className={styles.product.image.aspect}>
                    <img src={thumbnail?.url} alt="" className={styles.product.image.content} />
                  </div>
                  <div className={styles.product.summary}>
                    <p className={styles.product.title}>{name}</p>
                    <p className={styles.product.category}>{category?.name}</p>
                  </div>
                </a>
              </li>
            ),
          )}
      </ul>
    );
  }

  return null;
}
```

## The `ProductElement` component

Before we dive into other Saleor features, let's take a moment to slightly refactor the `ProductCollection` component. We can take each product element in the list and put it into a separate component. 

Call this component `ProductElement` and put it in `components/`

```tsx
// components/ProductElement.tsx
import React from 'react';

const styles = {
  card: 'bg-white border',
  summary: 'px-4 py-2 border-gray-100 bg-gray-50 border-t',
  title: 'block text-lg text-gray-900 truncate',
  category: 'block text-sm font-medium text-gray-500',
  image: {
    aspect: 'aspect-h-1 aspect-w-1',
    content: 'object-center object-cover'
  }
}

import { Product } from '@/saleor/api'

type Props = Pick<Product, 'id' | 'name' | 'thumbnail' | 'category'>;

export const ProductElement = ({ id, name, thumbnail, category }: Props) => {
  return (
    <li key={id} className={styles.card}>
      <a>
        <div className={styles.image.aspect}>
          <img src={thumbnail?.url} alt="" className={styles.image.content} />
        </div>
        <div className={styles.summary}>
          <p className={styles.title}>{name}</p>
          <p className={styles.category}>{category?.name}</p>
        </div>
      </a>
    </li>
  );
}
```

Let's also add the `export` statement for this component to `components/index.ts`:

```tsx{4}
// components/index.ts
export { ProductCollection } from './ProductCollection';
export { Layout } from './Layout';
export { ProductElement } from './ProductElement';
```

Now, we can reorganize the `ProductCollection` component in the following way:

```tsx
// components/ProductCollection.tsx
import React from 'react';

import { Product, useFetchTwelveProductsQuery } from '@/saleor/api';
import { ProductElement } from '@/components';

const styles = {
  grid: 'grid gap-4 grid-cols-4',
}

export const ProductCollection = () => {
  const { loading, error, data } = useFetchTwelveProductsQuery();

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

Refactoring `ProductElement` out of `ProductCollection` doesn't change anything visually in our application at this stage. It should look like on the following image:

![Refactored Product Collection](/images/product-overview.png)