---
pos: 4 
title: Adding to a Cart 
description: 
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

The `checkoutLinesAdd` mutation can provide not only the `id` of the checkout, but also the entire checkout object with its content available as `lines`. Let's slightly change the mutation response so that we can use `lines` for verifying that the mutation was successful. In this context, we will get the `id` of each line in the cart along with the name of product, its variant and quantity. Call this mutation `AddProductVariantToCart` and put it in `graphql/AddProductVariantToCart.graphql`:

```graphql{1,8-17}
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

```tsx{1-3,5,14-15,17,25-30,53-55}
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

## Removing from the cart

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

Let's name this mutation as `RemoveProductFromCheckout` and put it in `graphql/queries/RemoveProductFromCheckout.graphql`. As before, we are using the `CheckoutDetailsFragment` fragment to standarize the shape of a checkout object that we are getting in response to this mutation.

```graphql
mutation RemoveProductFromCheckout($checkoutToken: UUID!, $lineId: ID!) {
  checkoutLinesAdd(
    token: $checkoutToken
    lineId: $lineId
  ) {
    checkout {
      ...CheckoutDetailsFragment
    }
    errors {
      message
    }
  }
}
```

Now we can incorporate this mutation into our application as a React.js hook. Since the named our mutation `RemoveProductFromCheckout`, this will automatically generate the `useRemoveProductFromCheckout` hook.

The ability to remove from the cart will be added to the cart page available at `/cart`. We will add the remove link to each line in the cart.

```tsx
import {
  useCheckoutByTokenQuery,
  useRemoveProductFromCheckoutMutation,
} from "@/saleor/api";

const Cart: React.VFC = ({}) => {
  const [token] = useLocalStorage("token");
  const { data, loading, error } = useCheckoutByTokenQuery({
    fetchPolicy: "network-only",
    variables: { checkoutToken: token },
    skip: !token,
  });
  const [removeProductFromCheckout] = useRemoveProductFromCheckoutMutation();

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

                              <button
                                type="button"
                                onClick={() =>
                                  removeProductFromCheckout({
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
                              {price}
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

```