---
pos: 1
title: Overview 
description: 
next:
  path: /checkout/checkout-creation/
---

Let's go over the checkout process in Saleor. It is important that you finished the previous steps of this tutorial before starting this section. 

**Important**: In Saleor the checkout is synonymous to the cart in other e-commerce solutions. Thus, whenever we say *Checkout*, it can be substituted by *Cart*. In other words, both Checkout and Cart refer to the same entity that represent the currently selected produtcs for purchase.

Checkout is a state kept server-side by Saleor and we will be referring to it using an unique identifier. Whenever we want to add something to the cart, we notify Saleor about that change using the unique identifier of our checkout state.

First thing we need to do is to create the checkout and we will cover that in the next section.


