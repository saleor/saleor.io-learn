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
