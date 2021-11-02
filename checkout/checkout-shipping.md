---
pos: 999
title: Checkout Shipping
description: 
prev:
  path: /checkout/remove-from-cart/
next:
  path: /checkout/checkout-payment/
---

Products can be divided into physical and digital ones. For the physical products, we need an additional step in our checkout process to specify the proper shipping method.

We can use the `checkoutShippingMethodUpdate` mutation to attach the checkout instance with the shipping method selected by the user.