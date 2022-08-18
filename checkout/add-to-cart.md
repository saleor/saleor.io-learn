---
pos: 4
title: Adding to a Cart
description:
prev:
  path: /checkout/checkout-page/
next:
  path: /checkout/displaying-cart-content/
---

In the previous step we added the page that displays the content of a cart. For now it is just a dummy list of products. Let's make the cart page display the actual products that are added to that cart. Saleor API provides the `checkoutLinesAdd` mutation that takes the checkout `token` and a list of product variants along with their quantities (`lines`).

```graphql
checkoutLinesAdd(
  token: <token>
  lines: [{ quantity: 1, variantId: <product variant id >}]
) {
  checkout {
    id
  }
  error {
    messages
  }
}
```

<Notice>
When adding a product to a cart, we must use the product variant ID and not the product ID. One of the reasons is that a variant of the same product may have a different price.
</Notice>

The `checkoutLinesAdd` mutation can provide not only the `id` of the checkout, but also the entire checkout object with its content available as `lines`. Let's slightly change the mutation response so that we can use `lines` for verifying that the mutation was successful. In this context, we will get the `id` of each line in the cart along with the name of product, its variant and quantity. Call this mutation `ProductAddVariantToCart` and put it in `graphql/mutations/ProductAddVariantToCart.graphql` file:

```graphql
# graphql/mutations/ProductAddVariantToCart.graphql
mutation ProductAddVariantToCart($checkoutToken: UUID!, $variantId: ID!) {
  checkoutLinesAdd(
    token: $checkoutToken
    lines: [{ quantity: 1, variantId: $variantId }]
  ) {
    checkout {
      id
      lines {
        id
        quantity
        variant {
          name
          product {
            name
          }
        }
      }
    }
    errors {
      message
    }
  }
}
```

We can now incorporate this mutation into our React application.

<Notice>
In our application we automatically generate React Hooks for GraphQL operations using the GraphQL Code Generator toolset working in the `watch` mode.
</Notice>

Since we named our mutation `ProductAddVariantToCart`, this will generate the `useProductAddVariantToCart` hook.

Open the component for displaying the content of a single product, located at `components/ProductDetails.tsx` and modify it as shown below:

```tsx{4,6,31-32,40-45,73-75}
// components/ProductDetails.tsx
import React from 'react';
import { useRouter } from "next/router";
import { useLocalStorage } from "react-use";

import {
  useProductAddVariantToCartMutation,
  Product
} from "@/saleor/api";

import {
  VariantSelector
} from '@/components';

import { formatAsMoney } from '@/lib';

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
  product: Pick<Product, 'id' | 'name' | 'description' | 'thumbnail' | 'category' | 'media' | 'variants'>;
}

export const ProductDetails = ({ product }: Props) => {
  const router = useRouter();
  const [token] = useLocalStorage('token');
  const [addProductToCart] = useProductAddVariantToCartMutation();

  const queryVariant = process.browser
    ? router.query.variant?.toString()
    : undefined;
  const selectedVariantID = queryVariant || product?.variants![0]!.id!;
  const selectedVariant = product?.variants!.find((variant) => variant?.id === selectedVariantID);

  const onAddToCart = async () => {
    await addProductToCart({
      variables: { checkoutToken: token, variantId: selectedVariantID },
    });
    router.push("/cart");
  };

  return (
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

        <VariantSelector variants={product?.variants || []} id={product.id} selectedVariantID={selectedVariantID} />

        <div className="text-2xl font-bold">
          {formatAsMoney(selectedVariant?.pricing?.price?.gross.amount)}
        </div>

        <button
          onClick={onAddToCart}
          type="submit"
          className="primary-button"
        >
          Add to cart
        </button>
      </div>
    </div>
  );
}
```

At this stage the important part is the use of the `useAddProductToCheckoutMutation` hook to prepare (not execute) the mutation and the `onAddToCart` handler that executes the mutation once the _Add to cart_ button is clicked. We also get the token from the local storage to identify the current checkout session. Finally, once the mutation is executed and the selected product variant is added to the cart, we redirect to the cart page to display the current cart content. There is, however, a problem: we still display the dummy data for the cart. Let's change that in the next section.
