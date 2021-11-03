---
pos: 5 
title: Displaying the Cart Content
description: 
prev:
  path: /checkout/add-to-cart/
next:
  path: /checkout/remove-from-cart/
---

So far the checkout page that displays the content of the cart is static. We hardcoded all the elements on the page to focus on its structure. In this section, we will connect the checkout object provided by Saleor with what's displayed on the screen when visiting the `/cart` route. This way the cart page will be able to dynamically display what's currently in the cart.

## Getting the cart content from Saleor 

The client uses the `token` to reference the checkout object associated with the current session. At any point in time, we can use this token to fetch the checkout content. Let's start with defining a query for that. 

Let's name this mutation `CheckoutByToken` and put it in `graphql/queries/CheckoutByToken.graphql`:

```graphql
# graphql/queries/CheckoutByToken.graphql
query CheckoutByToken($checkoutToken: UUID!) {
  checkout(token: $checkoutToken) {
    id
    email
    lines {
      id
      totalPrice {
        gross {
          amount
          currency
        }
      }
      variant {
        product {
          id
          name
          slug
          thumbnail {
            url
            alt
          }
        }
        pricing {
          price {
            gross {
              amount
              currency
            }
          }
        }
        name
      }
    }
    totalPrice {
      gross {
        amount
        currency
      }
    }
  }
}
```

Now we can incorporate this mutation into our application as a React Hook. Since the named our mutation `CheckoutByToken`, this will automatically generate the `useCheckoutByToken` hook

## Simplifying the checkout query using fragments

The checkout object is returned not only from the checkout query, but also in response to various operations like adding or removing that operate on the checkout. In order to avoid repeating the same query part in all those situations, we can use GraphQL Fragment. Fragments as the name suggests describe a part of the query and are identified by a name.

When creating a fragment we start with a custom name followed by a existing GraphQL type. In this particular case we will create a fragment on `Checkout`. Put this GraphQL statement in `graphql/fragments/CheckoutFragment.graphql`:

```graphql
# graphql/fragments/CheckoutFragment.graphql
fragment CheckoutFragment on Checkout {
  id
  email
  lines {
    id
    totalPrice {
      gross {
        amount
        currency
      }
    }
    variant {
      product {
        id
        name
        slug
        thumbnail {
          url
          alt
        }
      }
      pricing {
        price {
          gross {
            amount
            currency
          }
        }
      }
      name
    }
  }
  totalPrice {
    gross {
      amount
      currency
    }
  }
}
```

With this newly created fragment, we can now simplify and rewrite the `CheckoutByToken` as follows:

```graphql
# graphql/queries/CheckoutByToken.graphql
query CheckoutByToken($checkoutToken: UUID!) {
  checkout(token: $checkoutToken) {
    ...CheckoutFragment
  }
}
```

## Displaying the cart 

Let's update the `pages/cart.tsx` page. Whenever a user navigates to this page, we will check for the session, and if a session is available, we will ask the Saleor API for the checkout content.

```tsx{3,6,13-24,31}
// pages/cart.tsx
import React from "react";
import { useLocalStorage } from 'react-use';

import {
  Layout,
  CartHeader,
  CartList,
  CartSummary
} from "@/components";
import { useCheckoutByTokenQuery } from "@/saleor/api";

const styles = {
  grid: 'grid grid-cols-3 gap-8',
}

const Cart = () => {
  const [token] = useLocalStorage('token');
  const { data, loading, error } = useCheckoutByTokenQuery({
    variables: { checkoutToken: token },
    skip: !token,
  });

  if (loading) return <div>Loading...</div>; 
  if (error) return <div>Error</div>;
  if (!data || !data.checkout) return null;

  const products = data.checkout?.lines || [];

  return (
    <Layout>
      <CartHeader />

      <div className={styles.grid}>
        <div className="col-span-2">
          <CartList products={products} />
        </div>
        <div>
          <CartSummary />
        </div>
      </div>
    </Layout>
  );
};

export default Cart;
```

As we pass the `products` list to `CartList`, we also need to adjust this component:

```tsx
// components/CartList.tsx
import React from 'react';
import Link from 'next/link';

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
                      <a>
                        {product?.name}
                      </a>
                    </Link>
                  </h3>
                  <h4>
                    {variant?.name}
                  </h4>
                </div>

                <p className="text-xl text-gray-900 text-right">
                  {price?.amount} {price?.currency}
                </p>
              </div>
            </div>
          </li>
        );
      })}
    </ul >
  );
}
```

## Adding Cart Icon in the Navbar