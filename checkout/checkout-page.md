---
pos: 3
title: Checkout Page
description: 
prev:
  path: /checkout/checkout-creation/
next:
  path: /checkout/checkout-content/
---

In a typical storefront, there is a page that displays the current content of a cart. The application from this tutorial will be no different. The cart page will be available under the `/cart` route.

## Setting up the cart page

Let's start by creating `pages/cart.tsx` with just the visual structure of the cart. There will be the website navbar along with the cart header and a list of product variants currently in the cart.

```tsx
import React from "react";
import Link from "next/link";

import {
  Navbar,
} from "@/components";

const Cart: React.VFC = ({}) => {
  return (
    <div className="min-h-screen bg-gray-100">
      <Navbar />

      <div className="py-10">
        <header className="mb-4">
          <div className="max-w-7xl mx-auto px-8">
            <div className="flex justify-between">
              <h1 className="text-3xl font-extrabold tracking-tight text-gray-900">
                Your Cart
              </h1>
              <div>
                <Link href="/">
                  <a className="text-sm text-blue-600 hover:text-blue-500">
                    Continue Shopping
                  </a>
                </Link>
              </div>
            </div>
          </div>
        </header>

        <main>
          <div className="max-w-7xl mx-auto px-8">
            <div className="grid grid-cols-3 gap-8">
              <div className="col-span-2">
                <ul role="list" className="divide-y divide-gray-200">
                  <li className="flex py-6">Product 1</li>
                  <li className="flex py-6">Product 2</li>
                </ul>
              </div>
              <div>
                Cart Summary
              </div>
            </div>
          </div>
        </main>
      </div>
    </div>
  );
};

export default Cart;
```

## Splitting the cart page into components

Let's refactor the cart page by splitting it into a few components so it's easier to manage. We can identify sections as components to extract: 
1. the cart header section that consists of the content between the `<header>...</header>` tags;
1. the list of products described by the `<ul>...</ul>`;
1. the cart summary that will eventually contain the cart value, i.e. the products total along with shipping costs, taxes and possible discounts

For the cart header, let's put it into `components/CartHeader.tsx`

```tsx
import React from "react";
import Link from "next/link";

export const CartHeader: React.VFC = ({}) => {
  return (
    <header className="mb-4">
      <div className="max-w-7xl mx-auto px-8">
        <div className="flex justify-between">
          <h1 className="text-3xl font-extrabold tracking-tight text-gray-900">
            Your Cart
          </h1>
          <div>
            <Link href="/">
              <a className="text-sm text-blue-600 hover:text-blue-500">
                Continue Shopping
              </a>
            </Link>
          </div>
        </div>
      </div>
    </header>
  );
```

For the cart list, let's put it into `components/CartList.tsx`

```tsx
import React from "react";
import Link from "next/link";

export const CartList: React.VFC = ({}) => {
  return (
    <ul role="list" className="divide-y divide-gray-200">
      <li className="flex py-6">Product 1</li>
      <li className="flex py-6">Product 2</li>
    </ul>
  )
}
```

And finally, for the cart summary, let's put it into the `components/CartSummary.tsx`

```tsx
import React from "react";
import Link from "next/link";

export const CartSummary: React.VFC = ({}) => {
  return (
    <div>Cart Summary</div>
  )
}
```

Now we can put it all together and rewrite the cart page (that's located in `pages/cart.tsx`):

```tsx
import React from "react";
import Link from "next/link";

import {
  Navbar,
  CartHeader,
  CartList,
  CartSummary
} from "@/components";

const Cart: React.VFC = ({}) => {
  return (
    <div className="min-h-screen bg-gray-100">
      <Navbar />

      <div className="py-10">
        <CartHeader />

        <main>
          <div className="max-w-7xl mx-auto px-8">
            <div className="grid grid-cols-3 gap-8">
              <div className="col-span-2">
                <CartList />
              </div>
              <CartSummary />
            </div>
          </div>
        </main>
      </div>
    </div>
  );
};

export default Cart;
```