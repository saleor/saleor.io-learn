---
pos: 7
title: Fetching single product 
description: 
prev:
  path: /product/pagination/
next:
  path: /product/variants/
---

Sometimes you may need to fetch a specific product. Usually, you display the most important product information on the collection page, and then when the user navigates to a specific product you provide details about it.  We will use the `product` query (singular) from the Saleor API for fetching those details.

## GraphQL Query for Single Product

Let's call our query `ProductByID` to make it explicit that it is about fetching product details by ID:

```graphql
# graphql/queries/ProductByID.graphql
query ProductByID($id: ID!) {
  product(id: $id, channel: "default-channel") {
    id
    name
    description
    media {
      url
    }
    category {
      name
    }
  }
}
```

The `product` query requires a value for the `id` argument, which is the product we want to fetch. We can then use the generated `useProductByIdQuery` hook in React components with the product `id` specified via the `variables` as shown below:

```js
const { loading, error, data } = useProductByIdQuery({ 
  variables: { 
    id: 'UHJvZHVjdDoxMTE=' 
  } 
});
```

## Display the Single Product Page

Let's start with creating a dedicated page for a single product. This page will be displayed when accessing the `/product/[id]` route with `[id]` being an actual product identifier. 

Since Next.js provides a file-based routing we can simply create the `product/` directory within `pages` along with the `[id].tsx` file inside that directory. The `[id].tsx` is a special format in Next.js for handling dynamic routes. Once the page is generated the dynamic section i.e. `[id]` will be replaced with an ID of an actual element. In our case, this will be a product ID.

Let's focus on the React component first. Let's name it `ProductPage`:

```tsx
// pages/product/[id].tsx
import {
  useProductByIdQuery,
} from "@/saleor/api";
import {
  Layout
} from '@/components';

const styles = {
  columns: 'grid grid-cols-2 gap-x-10 items-start',
  image: {
    aspect: 'aspect-w-1 aspect-h-1 bg-white rounded',
    content: 'object-center object-cover'
  },
  details: {
    title: 'text-4xl font-bold tracking-tight text-gray-800',
    category: 'text-lg mt-2 font-medium text-gray-500',
    description: 'prose lg:prose-s'
  }
}

interface Props {
  id: string;
}

const ProductPage = ({ id }: Props) => {
  const { loading, error, data } = useProductByIdQuery({ variables: { id } });

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error</p>;

  if (data) {
    const { product } = data;

    return (
      <Layout>
        <div className={styles.columns}>
          <div className={styles.image.aspect}>
            <img
              src={product?.media![0]?.url}
              className={styles.image.content}
            />
          </div>

          <div className="space-y-8">
            <div>
              <h1 className={styles.details.title}>
                {product?.name}
              </h1>
              <p className={styles.details.category}>
                {product?.category?.name}
              </p>
            </div>

            <article className={styles.details.description}>
              {product?.description}
            </article>
          </div>
        </div>
      </Layout>
    );
  }

  return null;
}

export default ProductPage;

```

`ProductPage` has the product identifier passed in and then it uses that to fetch details for a particular product using the auto-generated `useProductByIdQuery`. Then, we display the product data we received. The flow is similar to the page that displays the collection of products except that here we need an `id` of a product to display as input for this React component.

Next.js provides two special functions for React components that are used as pages (i.e. the ones located in `pages/` directory) with dynamic routes: `getStaticPaths` and `getStaticProps`.

`getStaticPaths` generates all the possible values for parameters in dynamic routes ahead of time when the website is being built. In our case, this function will return a collection of identifiers for all products provided by our Saleor API. Let's write it down:

```tsx{2,3,20-32}
// pages/product/[id].tsx
import {
  useProductByIdQuery,
  ProductCollectionDocument,
  ProductCollectionQuery
} from "@/saleor/api";
import { apolloClient } from "@/lib";
import {
  Layout
} from '@/components';

const styles = {
  // ... as before
}

interface Props {
  id: string;
}

const ProductPage = ({ id }: Props) => {
  // ... as before
}

export default ProductPage;

export async function getStaticPaths() {
  const { data } = await apolloClient.query<ProductCollectionQuery>({
    query: ProductCollectionDocument
  });
  const paths = data.products?.edges.map(({ node: { id } }) => ({
    params: { id },
  }));

  return {
    paths,
    fallback: false,
  };
}
```

First, we are executing a GraphQL to fetch a collection of products. We are re-using the same query we defined in the section about fetching products. Both, `ProductCollectionDocument` and `ProductCollectionQuery` are auto-generated from the query name (`ProductCollection`). The first is the actual query while the second is the type describing the shape of the data returned in response to that query.

The response data is a collection of products. We extract just the identifier of each product using the built-in `map` function, and return the result as the collection of paths.

Since we are re-using the instance of the Apollo client, let's move the initialization of that client to a separate file: `lib/graphql.ts`

```ts
import { ApolloClient, InMemoryCache } from '@apollo/client';
import { relayStylePagination } from "@apollo/client/utilities";

export const apolloClient = new ApolloClient({
  uri: "https://vercel.saleor.cloud/graphql/",
  cache: new InMemoryCache({
    typePolicies: {
      Query: {
        fields: {
          products: relayStylePagination(),
        },
      },
    }
  }),
});
```

For convenience, let's define the export statement in `lib/index.ts` so that we can import directly from `@/lib` anywhere in our codebase.

```ts
export { apolloClient } from './graphql';
```

The second function provided by Next.js for pages is `getStaticProps` - it returns the props that will be passed to the React component during the build. In our context, we return an identifier of a product to display. `getStaticProps` is invoked for each element of the collection returned by `getStaticPaths`. Let's add it to the single product page at `pages/product/[id].tsx`:

```tsx{2,26-32}
// pages/product/[id].tsx
import { GetStaticProps } from "next";

import {
  useProductByIdQuery,
  ProductCollectionDocument,
  ProductCollectionQuery
} from "@/saleor/api";
import { apolloClient } from "@/lib";
import {
  Layout
} from '@/components';

const styles = {
  // as before
}

interface Props {
  id: string;
}

const ProductPage = ({ id }: Props) => {
  // as before
}

export default ProductPage;

export async function getStaticPaths() {
  // as before
}

export const getStaticProps: GetStaticProps = async ({ params }) => {
  return {
    props: {
      id: params?.id,
    },
  };
};
```

## Link to Single Product Page

We have the page for displaying a single product, but it is not yet connected with the collection. We would like to be able to click on each element on the collection page to go to that particular product page. Let's adjust the `ProductElement` component that's responsible for displaying each element in the product collection.

```tsx{2,15,25}
import React from 'react';
import Link from 'next/link';

const styles = {
  // as before
}

import { Product } from '@/saleor/api'

type Props = Pick<Product, 'id' | 'name' | 'thumbnail' | 'category'>;

export const ProductElement = ({ id, name, thumbnail, category }: Props) => {
  return (
    <li key={id} className={styles.card}>
      <Link href={`/product/${id}`}>
        <a>
          <div className={styles.image.aspect}>
            <img src={thumbnail?.url} alt="" className={styles.image.content} />
          </div>
          <div className={styles.summary}>
            <p className={styles.title}>{name}</p>
            <p className={styles.category}>{category?.name}</p>
          </div>
        </a>
      </Link>
    </li>
  );
}
```

We use `Link` from the official `next/link` package to define the route to go when click on that element in the product collection. You should be now able to move from the collection page to the single product page by clicking around.

## Refactor the App Component

Since we moved the Apollo client initialization to `lib/graphql.ts`, the root component at `pages/_app.tsx` can be simplified:

```tsx{2,10}
import type { AppProps } from 'next/app'
import { ApolloProvider } from '@apollo/client';

import '../styles/main.css';

import { apolloClient } from '@/lib';

export default function MyApp({ Component, pageProps }: AppProps) {
  return (
    <ApolloProvider client={apolloClient}>
      <Component {...pageProps} />
    </ApolloProvider>
  )
}
```

## Define the Navbar

Let's finish off by defining a navbar with a link that gets us back to the collection page whenever we click on it. This way we can navigate to a product and then back to the collection without using the back button in the browser.

```tsx
import React from "react";
import Link from "next/link";

const styles = {
  background: 'bg-white shadow-sm',
  container: 'max-w-7xl mx-auto px-8',
  menu: 'flex justify-between h-16',
  menuSection: 'flex space-x-8 h-full',
  menuLink: 'font-bold text-gray-700 hover:text-blue-400 z-10 flex items-center text-sm'
}

export const Navbar = () => {
  return (
    <div className={styles.background}>
      <div className={styles.container}>
        <div className={styles.menu}>
          <div className={styles.menuSection}>
            <Link href="/">
              <a className={styles.menuLink} aria-expanded="false">
                All Products 
              </a>
            </Link>
          </div>

          <div className={styles.menuSection}>
          </div>
        </div>
      </div>
    </div>
  );
};
```

The product collection is visible on the home route at `/`. We use the same path for the `Link` component in `Navbar`. Let's not forget to add it to `components/index.ts` so that we can always export it from `@/components`:

```tsx{5}
export { ProductCollection } from './ProductCollection';
export { Layout } from './Layout';
export { ProductElement } from './ProductElement';
export { Pagination } from './Pagination';
export { Navbar } from './Navbar';
```

Finally, let's add `Navbar` to `Layout` so it's always visible:

```tsx{17}
import React from 'react';

import { Navbar } from '@/components';

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
      <Navbar />
      <div className={styles.container}>
        {children}
      </div>
    </div>
  );
}

```