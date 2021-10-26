---
pos: 3
title: Pricing for variants 
description: 
---

You can also query the prices for variants. In that case the `pricing` field no longer returns a range, but a single price.

## Query for Variant Price

Let's start with adjusting the `ProductByID` query by getting a price for each variant as shown below:

```graphql{13-23}
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
      pricing {
        price {
          gross {
            amount
          }
        }
      }
    }
  }
}
```

This is similar to getting a price range for a product, except that variants can only have one value for price, thus we use `price` instead of `priceRange`.

## Displaying the Price for Product Variant

With the price available for each variant, we can now adjust the `ProductDetails.tsx` component as shown below:

```tsx{9,37,68-70}
import React from 'react';
import { useRouter } from "next/router";
import { useLocalStorage } from "react-use";

import {
  useAddProductVariantToCartMutation,
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
  const [addProductToCart] = useAddProductVariantToCartMutation();

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

We are getting the currently selected variant from the route via the query params. The URL contains the identifier of a variant for the product on the current page. Using the identifier you search for the variant object within the response returned via React Hook that triggers the `ProductByID` query. Having the whole variant object of a product, we can not only display the price for a product variant, but the price will also dynamically change whenver we click to select a different one.

Finally, we define a helper function to format the decimal value of a price. Let's call this function `formatAsMoney` and put it in the `lib/util.ts`

```ts
export const formatAsMoney = (amount: number = 0, currency = "USD") =>
  new Intl.NumberFormat("en-US", {
    minimumFractionDigits: 2,
    style: "currency",
    currency,
  }).format(amount);
```