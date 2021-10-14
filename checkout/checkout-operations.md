---
pos: 999 
title: Checkout Operations
description: 
---

Since a `Checkout` represents a `Cart` in Saleor, there are `Checkout` operations for adding, removing and updating the cart content.

## Adding to a cart

Let's start by adding something to a cart. Saleor provides the `checkoutLinesAdd` mutation that takes the checkout `token` and an array of product variants with quantities as `lines`.

```graphql
checkoutLinesAdd(
  token: <token> 
  lines: [{ quantity: 1, variantId: <product variant id >}]
) {
  checkout {
    token
  }
  error {
    messages
  }
}
```

It is important to note that when adding a product to a cart, we must use the product variant ID and not the product ID. One of the reasons is that variants of the same product may have different prices.

Let's name this mutation as `AddProductToCheckout` and put it in `graphql/queries/AddProductToCheckout.graphql`

```graphql
mutation AddProductToCheckout($checkoutToken: UUID!, $variantId: ID!) {
  checkoutLinesAdd(
    token: $checkoutToken
    lines: [{ quantity: 1, variantId: $variantId }]
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

Finally we need to incorporate this mutation into React.js as a hook.

<Notice>
Reminder. In our application we automatically generate React.js hooks for GraphQL operations using the GraphQL Code Generator toolset.
</Notice>

Since the named our mutation `AddProductToCheckout`, this will generate the `useAddProductToCheckout` hook.

```tsx
const ProductPage: React.VFC<InferGetStaticPropsType<typeof getStaticProps>> =
  ({ productSlug, token }) => {

    const [addProductToCheckout] = useAddProductToCheckoutMutation();

    const onAddToCart = async () => {
      await addProductToCheckout({
        variables: { checkoutToken: token, variantId: selectedVariantId },
      });
      router.push("/cart");
    };

    return (
      <div className="min-h-screen bg-gray-100">
        ...
        <button
          onClick={onAddToCart}
          type="submit"
          className="max-w-xs w-full bg-blue-500 border border-transparent rounded-md py-3 px-8 flex items-center justify-center text-white hover:bg-blue-600 focus:outline-none"
        >
          Add to cart
        </button>
      </div>
    )
  }
```

The `ProductPage` component is not shown in full as it's a bit complex. At this stage the important part is the use of the `useAddProductToCheckoutMutation` hook to prepare (not execute) the mutation and the `onAddToCart` handler that executes the mutation once the *Add to cart* button is clicked.

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