---
pos: 6
title: Removing from a Cart
description:
prev:
  path: /checkout/displaying-cart-content/
next:
  path: /checkout/checkout-shipping/
---

Removing from a cart is available as the `checkoutLineDelete` mutation that takes the checkout `token` and the ID of a line in checkout to remove.

```graphql
checkoutLineDelete(token: $checkoutToken, lineId: $lineId) {
  checkout {
    token
  }
  errors {
    field
    message
  }
}
```

Let's name this mutation as `CheckoutRemoveProduct` and put it in `graphql/mutations/CheckoutRemoveProduct.graphql`. As before, we are using the `CheckoutFragment` fragment to standardize the shape of a checkout object we are getting in response to this mutation.

```graphql
# graphql/mutations/CheckoutRemoveProduct.graphql
mutation CheckoutRemoveProduct($checkoutToken: UUID!, $lineId: ID!) {
  checkoutLineDelete(token: $checkoutToken, lineId: $lineId) {
    checkout {
      ...CheckoutFragment
    }
    errors {
      message
    }
  }
}
```

Now we can incorporate this mutation into our application as a React.js hook. Since we named our mutation `CheckoutRemoveProduct`, this will automatically generate the `useCheckoutRemoveProduct` hook.

We will add the possibility to remove a product from the cart to the `CartList` component located at `components/CartList`. We will attach the remove link to each line in the cart.

```tsx{5,7,22,23,53-55}
// components/CartList.tsx
import React from 'react';
import Link from 'next/link';

import {
  useCheckoutRemoveProductMutation,
} from "@/saleor/api";

import { useLocalStorage } from 'react-use';

interface Props {
  products: any[];
}

const styles = {
  product: {
    image: 'flex-shrink-0 bg-white w-48 h-48 border object-center object-cover',
    container: 'ml-8 flex-1 flex flex-col justify-center',
    name: 'text-xl font-bold',
  }
}

export const CartList = ({ products }: Props) => {
  const [token] = useLocalStorage("token");
  const [CheckoutremoveProduct] = useCheckoutRemoveProductMutation();

  return (
    <ul role="list" className="divide-y divide-gray-200">
      {products.map((line) => {
        const lineID = line?.id || "";
        const variant = line?.variant;
        const product = line?.variant.product;
        const price = line?.totalPrice?.gross;
        const productID = product?.id

        return (
          <li key={line?.id} className="py-6 flex">
            <div className={styles.product.image}>
              <img
                src={product?.thumbnail?.url || ""}
                alt={product?.thumbnail?.alt || ""}
              />
            </div>

            <div className={styles.product.container}>
              <div className="flex justify-between">
                <div className="pr-6">
                  <h3 className={styles.product.name}>
                    <Link href={`/product/${productID}`}>
                        {product?.name}
                    </Link>
                  </h3>
                  <h4>
                    {variant?.name}
                  </h4>
                  <button
                    type="button"
                    onClick={() =>
                      CheckoutremoveProduct({
                        variables: {
                          checkoutToken: token,
                          lineId: lineID,
                        },
                      })
                    }
                    className="ml-4 text-sm font-medium text-indigo-600 hover:text-indigo-500 sm:ml-0 sm:mt-3"
                  >
                    <span>Remove</span>
                  </button>
                </div>

                <p className="text-xl text-gray-900 text-right">
                  {price?.amount} {price?.currency}
                </p>
              </div>
            </div>
          </li>
        );
      })}
    </ul>
  );
}
```
