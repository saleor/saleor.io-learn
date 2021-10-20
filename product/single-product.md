---
pos: 7
title: Fetching single product 
description: 
prev:
  path: /product/pagination/
next:
  path: /product/variants/
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

## Next.js Page for Single Product


```tsx
import { GetStaticProps } from "next";
import { useRouter } from "next/router";
import Blocks from "editorjs-blocks-react-renderer";

import { Navbar } from "@/components";
import {
  useAddProductToCheckoutMutation,
  useProductByIdQuery,
  Product,
} from "@/saleor/api";

import { ProductPaths } from "../../components/config";
import apolloClient from "../../lib/graphql";

export default function ProductPage({ product, token }: { product: Product; token: string }) {
  const router = useRouter();

  const { loading, error, data } = useProductByIdQuery({ variables: product });

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error</p>;

  if (data) {
    const { product } = data;
    const price = product?.pricing?.priceRange?.start?.gross.amount || 0;
    const variantId = product?.variants![0]!.id!;

    return (
      <div className="min-h-screen bg-gray-100">
        <Navbar />

        <main className="max-w-7xl mx-auto pt-8 px-8">
          <div className="grid grid-cols-2 gap-x-10 items-start">
            <div className="w-full aspect-w-1 aspect-h-1 bg-white rounded">
              <img
                src={product?.media![0]?.url}
                className="w-full h-full object-center object-cover"
              />
            </div>

            <div className="space-y-8">
              <div>
                <h1 className="text-4xl font-bold tracking-tight text-gray-800">
                  {product?.name}
                </h1>
                <p className="text-lg mt-2 font-medium text-gray-500">
                  {product?.category?.name}
                </p>
              </div>

              <p className="text-2xl text-gray-900">
                {new Intl.NumberFormat("en-US", {
                  minimumFractionDigits: 2,
                  style: "currency",
                  currency: "USD",
                }).format(price)}
              </p>

              <div className="text-base text-gray-700 space-y-6">
                <article className="prose lg:prose-s">
                  <Blocks data={JSON.parse(product?.description)} />
                </article>
              </div>

              <div className="grid grid-cols-8 gap-2">
                {product?.variants?.map((variant) => {
                  return (
                    <a
                      key={variant?.name}
                      className="flex justify-center border border-gray-300 rounded-md p-3 font-semibold hover:border-blue-300"
                    >
                      {variant?.name}
                    </a>
                  );
                })}
              </div>
            </div>
          </div>
        </main>
      </div>
    );
  }

  return null;
}

export async function getStaticPaths() {
  const { data } = await apolloClient.query({ query: ProductPaths });
  const paths = data.products.edges.map(({ node }) => ({
    params: { slug: node.id },
  }));

  return {
    paths,
    fallback: false,
  };
}

export const getStaticProps: GetStaticProps<any> = async ({ params }) => {
  const product = { id: params?.slug };

  return {
    props: {
      product,
    },
  };
};

```