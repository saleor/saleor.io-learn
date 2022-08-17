---
pos: 3
title: Checkout Page
description:
prev:
  path: /checkout/checkout-creation/
next:
  path: /checkout/add-to-cart/
---

In a typical storefront, there is a page that displays the current content of a cart. The application from this tutorial will be no different. The cart page will be available under the `/cart` route.

## Setting up the cart page

Let's start by creating the `cart.tsx` file inside the `pages` with just the visual structure of the cart. It will consist of the navbar, the cart header, and the list of product variants currently in the cart.

```tsx
// pages/cart.tsx
import React from "react";
import Link from "next/link";

import { Layout } from "@/components";

const styles = {
  header: "flex justify-between",
  title: "text-3xl font-extrabold tracking-tight text-gray-900",
  grid: "grid grid-cols-3 gap-8",
};

const Cart = () => {
  return (
    <Layout>
      <header className={styles.header}>
        <h1 className={styles.title}>Your Cart</h1>
        <div>
          <Link href="/">
            <a className="link">Continue Shopping</a>
          </Link>
        </div>
      </header>

      <div className={styles.grid}>
        <div className="col-span-2">
          <ul role="list" className="divide-y divide-gray-200">
            <li className="py-6">Product 1</li>
            <li className="py-6">Product 2</li>
          </ul>
        </div>
        <div>Cart Summary</div>
      </div>
    </Layout>
  );
};

export default Cart;
```

## Splitting the cart page into components

Before adding new features, let's refactor the cart page a little. We will split the cart component into a few smaller components so that it's easier to manage.

We can identify the following sections to extract as separate components:

1. The cart header section that consists of the content between the `<header>...</header>` tags;
1. The list of products inside the `<ul>...</ul>` tags;
1. The cart summary that will eventually contain the cart value, i.e. the products total along with shipping costs, taxes and possible discounts;

As for the cart header, let's put it into `components/CartHeader.tsx`

```tsx
// components/CartHeader.tsx
import React from "react";
import Link from "next/link";

const styles = {
  header: "flex justify-between",
  title: "text-3xl font-extrabold tracking-tight text-gray-900",
};

export const CartHeader = () => {
  return (
    <header className={styles.header}>
      <h1 className={styles.title}>Your Cart</h1>
      <div>
        <Link href="/">
          <a className="link">Continue Shopping</a>
        </Link>
      </div>
    </header>
  );
};
```

For the cart list, let's put it into `components/CartList.tsx`

```tsx
// components/CartList.tsx
import React from "react";

export const CartList = () => {
  return (
    <ul role="list" className="divide-y divide-gray-200">
      <li className="py-6">Product 1</li>
      <li className="py-6">Product 2</li>
    </ul>
  );
};
```

And finally, for the cart summary, let's put it into the `components/CartSummary.tsx`

```tsx
// components/CartSummary.tsx
import React from "react";

export const CartSummary = () => {
  return <div>Cart Summary</div>;
};
```

Let's not forget to export added components from `components/index.ts`:

```ts{9-11}
// components/index.ts
export { ProductCollection } from './ProductCollection';
export { Layout } from './Layout';
export { ProductElement } from './ProductElement';
export { Pagination } from './Pagination';
export { Navbar } from './Navbar';
export { ProductDetails } from './ProductDetails';
export { VariantSelector } from './VariantSelector';
export { CartHeader } from './CartHeader';
export { CartList } from './CartList';
export { CartSummary } from './CartSummary';
```

Now, we can put it all together and rewrite the cart page (located in `pages/cart.tsx`):

```tsx
// pages/cart.tsx
import React from "react";

import { Layout, CartHeader, CartList, CartSummary } from "@/components";

const styles = {
  grid: "grid grid-cols-3 gap-8",
};

const Cart = () => {
  return (
    <Layout>
      <CartHeader />

      <div className={styles.grid}>
        <div className="col-span-2">
          <CartList />
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

Open your browser and navigate to `/cart`. You should see the following page:

![Visual structure of the cart.](/images/checkout-page.png)
