---
pos: 4 
title: Adding to a Cart 
description: 
prev:
  path: /checkout/checkout-page/
next:
  path: /checkout/displaying-cart-content/
---

In the previous step we added the page that displays the content of a cart. For now it is just a dummy list of products. Let's make the cart page display the actual products that are added to that cart.  Saleor API provides the `checkoutLinesAdd` mutation that takes the checkout `token` and a list of product variants along with their quantities (`lines`).

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

It is important to note that when adding a product to a cart, we must use the product variant ID and not the product ID. One of the reasons is that a variant of the same product may have a different price.

The `checkoutLinesAdd` mutation can provide not only the `id` of the checkout, but also the entire checkout object with its content available as `lines`. Let's slightly change the mutation response so that we can use `lines` for verifying that the mutation was successful. In this context, we will get the `id` of each line in the cart along with the name of product, its variant and quantity. Call this mutation `AddProductVariantToCart` and put it in `graphql/mutations/AddProductVariantToCart.graphql`:

```graphql{2,9-18}
# graphql/mutations/AddProductVariantToCart.graphql
mutation AddProductVariantToCart($checkoutToken: UUID!, $variantId: ID!) {
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
Reminder. In our application we automatically generate React Hooks for GraphQL operations using the GraphQL Code Generator toolset.
</Notice>

Since we named our mutation `AddProductVariantToCart`, this will generate the `useAddProductVariantToCart` hook.  

Open the page for displaying the content of a single product, located at `pages/product/[id].tsx` and modify it as shown below:

```tsx{2-4,6,14-16,18,26-31,54-56}
// pages/product/[id].tsx
import { GetStaticProps, InferGetStaticPropsType } from "next";
import { useRouter } from "next/router";
import { useLocalStorage } from "react-use";

import {
  useProductByIdQuery,
  useAddProductVariantToCartMutation,
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

const ProductPage = ({ id }: InferGetStaticPropsType<typeof getStaticProps>) => {
  const router = useRouter();
  const [token] = useLocalStorage('token');
  const { loading, error, data } = useProductByIdQuery({ variables: { id } });
  const [addProductToCart] = useAddProductVariantToCartMutation();

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error</p>;

  if (data) {
    const { product } = data;

    const onAddToCart = async () => {
      await addProductToCart({
        variables: { checkoutToken: token, variantId: product?.variants![0]?.id! },
      });
      router.push("/cart");
    };

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

            <button
              onClick={onAddToCart}
              type="submit"
              className="primary-button"
            >
              Add to cart
            </button>
          </div>
        </div>
      </Layout>
    );
  }

  return null;
}

export default ProductPage;

export async function getStaticPaths() {
  // as before
}

export const getStaticProps: GetStaticProps = async ({ params }) => {
  // as before
};

```

The `ProductPage` component is not shown in full as it's a bit complex. At this stage the important part is the use of the `useAddProductToCheckoutMutation` hook to prepare (not execute) the mutation and the `onAddToCart` handler that executes the mutation once the *Add to cart* button is clicked. We also get the token from the local storage to identify the current checkout session. Finally, once the mutation is executed and the selected product variant is added to the cart, we redirect to the cart page to display the current cart content. There is, however, a problem: we still display the dummy data for the cart. Let's change that in the next section.
