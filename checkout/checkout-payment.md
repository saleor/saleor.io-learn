---
pos: 999
title: Checkout Payment
description:
prev:
  path: /checkout/checkout-shipping/
---

While creating the checkout, you may also specify the shipping and the billing address at this stage. The following mutation shows how this could be achieved:

```graphql
mutation {
  checkoutCreate(
    input: {
      channel: "default-channel"
      email: "customer@example.com"
      lines: [{ quantity: 2, variantId: "UHJvZHVjdFZhcmlhbnQ6MjU2" }]
      shippingAddress: {
        firstName: "John"
        lastName: "Doe"
        streetAddress1: "1470  Pinewood Avenue"
        city: "Michigan"
        postalCode: "49855"
        country: US
        countryArea: "MI"
      }
      billingAddress: {
        firstName: "John"
        lastName: "Doe"
        streetAddress1: "1470  Pinewood Avenue"
        city: "Michigan"
        postalCode: "49855"
        country: US
        countryArea: "MI"
      }
    }
  ) {
    checkout {
      id
      totalPrice {
        gross {
          amount
          currency
        }
      }
      isShippingRequired
    }
    errors {
      field
      code
    }
  }
}
```

You may need to know what the available shipping methods are. The `checkoutCreate` mutation can also return this information as shown in the example below:

```graphql
mutation {
  checkoutCreate(
    input: {
      channel: "default-channel"
      email: "customer@example.com"
      lines: [{ quantity: 1, variantId: "UHJvZHVjdFZhcmlhbnQ6Mjk3" }]
      shippingAddress: {
        firstName: "John"
        lastName: "Doe"
        streetAddress1: "1470  Pinewood Avenue"
        city: "Michigan"
        postalCode: "49855"
        country: US
        countryArea: "MI"
      }
      billingAddress: {
        firstName: "John"
        lastName: "Doe"
        streetAddress1: "1470  Pinewood Avenue"
        city: "Michigan"
        postalCode: "49855"
        country: US
        countryArea: "MI"
      }
    }
  ) {
    checkout {
      id
      totalPrice {
        gross {
          amount
          currency
        }
      }
      isShippingRequired
      shippingMethods {
        id
        name
      }
    }
    errors {
      field
      code
    }
  }
}
```
