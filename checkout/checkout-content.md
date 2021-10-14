---
pos: 4
title: Checkout Content
description: 
---

So far the checkout page that displays the content of the cart is static. We hardcoded all the elements on the page to focus on its structure. In this section, we will connect the checkout object provided by Saleor with what's displayed on the screen when visiting the `/cart` route. This way the cart page will be able to dynamically display what's currently in the cart.

## Getting the checkout content from Saleor 

The client uses the `token` to reference the checkout object associated with the current session. At any point in time, we can use this token to fetch the checkout content. Let's start with defining a query for that. 

Let's name this mutation as `CheckoutByToken` and put it in `graphql/queries/CheckoutByToken.graphql`. 

```graphql
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

Now we can incorporate this mutation into our application as a React.js hook. Since the named our mutation `CheckoutByToken`, this will automatically generate the `useCheckoutByToken` hook

## Simplifying the checkout query using fragments

The checkout object is returned not only from the checkout query, but also in response to various operations like adding or removing that operate on the checkout. In order to avoid repeating the same query part in all those situations, we can use GraphQL Fragment. Fragments as the name suggests describe a part of the query and are identified by a name.

When creating a fragment we start with a custom name followed by a existing GraphQL type. In this particular case we will create a fragment on `Checkout`.

```graphql
fragment CheckoutDetailsFragment on Checkout {
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
query CheckoutByToken($checkoutToken: UUID!) {
  checkout(token: $checkoutToken) {
    ...CheckoutDetailsFragment
  }
}
```

## Displaying the checkout content

Let's create the `pages/cart.tsx` page that will be available at `/cart`. Whenever a user navigates to this page, we will check for the session, and if a session is available, we will ask the Saleor API for the checkout content.

```tsx
import React from "react";
import Link from "next/link";
import Image from "next/image";

import { useLocalStorage } from "react-use";

import { Navbar, CartSummary } from "@/components";
import { useCheckoutByTokenQuery } from "@/saleor/api";
import { CHECKOUT_TOKEN } from "@/lib/const";
import BaseTemplate from "@/components/BaseTemplate";

const Cart: React.VFC = ({}) => {
  const [token] = useLocalStorage(CHECKOUT_TOKEN);
  const { data, loading, error } = useCheckoutByTokenQuery({
    fetchPolicy: "network-only",
    variables: { checkoutToken: token },
    skip: !token,
  });

  if (loading) return <BaseTemplate isLoading={true} />;
  if (error) return <p>Error</p>;
  if (!data || !data.checkout) return null;
  
  const products = data.checkout?.lines || [];

  return (
    <BaseTemplate>
      <div className="py-10">
        <header className="mb-4">
          <div className="max-w-7xl mx-auto px-8">
            <div className="flex justify-between">
              <h1 className="text-3xl font-extrabold tracking-tight text-gray-900">
                Your Cart
              </h1>
              <div>
                <Link href="/">
                  <a className="text-sm ">Continue Shopping</a>
                </Link>
              </div>
            </div>
          </div>
        </header>
        <main>
          <div className="grid grid-cols-1 md:grid-cols-3 gap-8 max-w-7xl mx-auto px-8">
            <section className="col-span-2">
              <ul role="list" className="divide-y divide-gray-200">
                {products.map((line) => {
                  const lineID = line?.id || "";
                  const variant = line?.variant;
                  const product = line?.variant.product;
                  const price = line?.totalPrice?.gross;
                  return (
                    <li key={line?.id} className="flex py-6">
                      <div className="flex-shrink-0 bg-white w-48 h-48 border object-center object-cover relative">
                        <Image
                          src={product?.thumbnail?.url || ""}
                          alt={product?.thumbnail?.alt || ""}
                          layout="fill"
                        />
                      </div>

                      <div className="ml-8 flex-1 flex flex-col justify-center">
                        <div>
                          <div className="flex justify-between">
                            <div className="pr-6">
                              <h3 className="text-xl font-bold">
                                <Link href={`/products/${product?.slug}`}>
                                  <a className="font-medium text-gray-700 hover:text-gray-800">
                                    {product?.name}
                                  </a>
                                </Link>
                              </h3>
                              <h4 className="text-m font-regular">
                                <a className="text-gray-700 hover:text-gray-800">
                                  {variant?.name}
                                </a>
                              </h4>
                            </div>

                            <p className="text-xl text-gray-900 text-right">
                              {price?.localizedAmount}
                            </p>
                          </div>
                        </div>
                      </div>
                    </li>
                  );
                })}
              </ul>
            </section>

            <div>
              <CartSummary checkout={data.checkout} />
              <div className="mt-12">
                <Link href="/checkout">
                  <a className="block w-full bg-blue-500 border border-transparent rounded-md shadow-sm py-3 px-4 text-center font-medium text-white hover:bg-blue-700">
                    Checkout
                  </a>
                </Link>
              </div>
            </div>
          </div>
        </main>
      </div>
    </BaseTemplate>
  );
};

export default Cart;

```
