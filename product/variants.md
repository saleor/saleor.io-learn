---
pos: 8
title: Using Variants 
description: 
prev:
  path: /product/single-product/
next:
  path: /prices/overview/
---

A product may be available in different variants, e.g.  a particular t-shirt may be available in different sizes, and each size is a variant of that t-shirt. The notion of variants is available in Saleor out of the box. You can access it on the product via the `variants` field.

## Fetching variants of a product

In the previous step, we used the `ProductByID` query to fetch details for a particular product in our store. Let's now extend this query to also include the information about the possible variants of each product. In `graphql/queries/ProductByID.graphql` add the following `variants` snippet:

```graphql{13-16}
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
    variants {
      id
      name
    }
  }
}
```

If you run this query for a product from the T-shirt category, you will notice that the variant names are T-shirt sizes, i.e. S, M, L or XL.

## Displaying variants on the product page

Let's display all possible variants of a product on the page that displays the product details. This is being handled by the `ProductDetails` component.

```tsx{6,22,46}
// components/ProductDetails.tsx
import React from 'react';

import {
  Product
} from "@/saleor/api";

import {
  VariantSelector
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
  product: Pick<Product, 'id' | 'name' | 'description' | 'thumbnail' | 'category' | 'media' | 'variants'>;
}

export const ProductDetails = ({ product }: Props) => {
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

        <VariantSelector variants={product?.variants || []} id={product.id} />
      </div>
    </div>
  );
}
```

Let's add the export statement in `components/index.ts` for `VariantSelector`, so that we can directly import it from `@/components`.


```tsx{8}
// components/index.ts
export { ProductCollection } from './ProductCollection';
export { Layout } from './Layout';
export { ProductElement } from './ProductElement';
export { Pagination } from './Pagination';
export { Navbar } from './Navbar';
export { ProductDetails } from './ProductDetails';
export { VariantSelector } from './VariantSelector';
```


With the `ProductByID` query adjusted by `variants` we pass them to a newly created `VariantSelector` component. This components gets a collection of variants and displays their names:

```tsx
// components/VariantSelector.tsx
import React from 'react';
import { ProductVariant } from '@/saleor/api'

type Variant = Pick<ProductVariant, "id" | "name"> | null | undefined;

const styles = {
  grid: 'grid grid-cols-8 gap-2',
  variant: 'flex justify-center border rounded-md p-3 font-semibold hover:border-blue-400',
}

interface Props {
  variants: Variant[];
}

export const VariantSelector = ({ variants }: Props) => {
  
  return (
    <div className={styles.grid}>
      {variants.map((variant) => {
        return (
          <a className={styles.variant}>
            {variant?.name}
          </a>
        );
      })}
    </div>    
  );
}
```

Next step is to make those variant clickable so that the user can select a particular variant of a product they want to add to their cart.

For constructing className strings conditionally let's use a tiny utility - `clsx`.

```
npm install clsx
```

```
pnpm add clsx
```

Now the `VariantSelector` component can be modified:


```tsx
// components/VariantSelector.tsx
import React from 'react';
import Link from 'next/link';
import clsx from 'clsx';

import { ProductVariant } from '@/saleor/api'

type Variant = Pick<ProductVariant, "id" | "name"> | null | undefined;

const styles = {
  grid: 'grid grid-cols-8 gap-2',
  variant: {
    default: 'flex justify-center border rounded-md p-3 font-semibold hover:border-blue-400',
    selected: 'border-2 border-blue-300 bg-blue-300',
  }
}

interface Props {
  id: string;
  selectedVariantID: string;
  variants: Variant[];
}

export const VariantSelector = ({ variants, id, selectedVariantID }: Props) => {
  return (
    <div className={styles.grid}>
      {variants.map((variant) => {
        const isSelected = variant?.id === selectedVariantID;
        return (
          <Link
            key={variant?.name}
            href={{
              pathname: "/product/[id]",
              query: { variant: variant?.id, id },
            }}
            replace
            shallow
          >
            <a className={clsx(styles.variant.default, isSelected && styles.variant.selected)}>
              {variant?.name}
            </a>
          </Link>
        );
      })}
    </div>    
  );
}
```

The id of a selected variant is being passed-in to the `VariantSelector` component as the parent component must also know its value so that we can add the correct product variant to the cart. Let's slightly modify the parent component, i.e. `ProductDetails`:

```tsx{3,7,23,27,29-32,54}
// components/ProductDetails.tsx
import React from 'react';
import { useRouter } from "next/router";

import {
  Product
} from "@/saleor/api";

import {
  VariantSelector
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
  product: Pick<Product, 'id' | 'name' | 'description' | 'thumbnail' | 'category' | 'media' | 'variants'>;
}

export const ProductDetails = ({ product }: Props) => {
  const router = useRouter();

  const queryVariant = process.browser
    ? router.query.variant?.toString()
    : undefined;
  const selectedVariantID = queryVariant || product?.variants![0]!.id!;

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
      </div>
    </div>
  );
}

```