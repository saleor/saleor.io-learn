---
pos: 999
title: Checkout Shipping
description:
prev:
  path: /checkout/remove-from-cart/
next:
  path: /checkout/checkout-payment/
---

Products can be divided into physical and digital ones. For the physical products, we need an additional step in our checkout process to specify the proper shipping method. Saleor API provides queries and mutations which enable managing the shipping process efficiently.

1. The `availableShippingMethods` query will list all possible shipping methods for your shop.

```graphql
query ShippingMethodsFetchAll {
  shop {
    availableShippingMethods(channel: "default-channel") {
      id
      name
    }
  }
}
```

2. You can list the shipping methods for the current checkout by updating the `CheckoutFragment` query with `shippingMethods` object.

```graphql{39-46}
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
  shippingMethods{
    id
    name
    price{
      amount
      currency
    }
  }
}
```

3. If you want to update the shipping information for the checkout, set the shipping address with `checkoutShippingAddressUpdate` mutation, which will be necessary to execute the `checkoutDeliveryMethodUpdate`.

```graphql
mutation CheckoutUpdateShippingAddress {
  checkoutShippingAddressUpdate(
    id: "Q2hlY2tvdXQ6ODAxMWFkOWQtZTg3YS00ZjQyLWE3YzUtYmRiYTIyZGU1ZTky"
    shippingAddress: {
      firstName: "John"
      streetAddress1: "Some Street"
      city: "Warsaw"
      country: PL
      postalCode: "22-100"
    }
  ) {
    checkout {
      id
    }
    errors {
      field
      message
      code
    }
  }
}
```

```graphql
mutation CheckoutUpdateDeliveryMethod {
  checkoutDeliveryMethodUpdate(
    deliveryMethodId: "U2hpcHBpbmdNZXRob2Q6Mg=="
    id: "Q2hlY2tvdXQ6ODAxMWFkOWQtZTg3YS00ZjQyLWE3YzUtYmRiYTIyZGU1ZTky" //id of the checkout
  ) {
    checkout {
      id
    }
    errors {
      field
      message
      code
    }
  }
}
```

You can play along with these in your GraphQL Playground. However, having gone through the previous parts in this section, we encourage you to build the shipping methods form and update the checkout with the delivery methods the user chooses. Good luck!
