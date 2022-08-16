---
pos: 1
title: Overview
description:
prev:
  path: /product/variants/
next:
  path: /prices/pricing-range/
---

In order to sell a product in a store we need to communicate its price. As products may come in different shapes and sizes, each product variant may be priced differently. In this section we will describe the fundamentals of pricing products and how we can get most of the Saleor API.

Now we know how to fetch products from GraphQL API and how to add some of those products to a cart to initiate the checkout process. The next step is to understand the pricing. Products have prices, but it is also possible a product depending on its size or shape, has some different prices. In that case, we talk about the price range of a given product.

The price for each product contains the tax (which may also be different for different countries). Saleor distinguishes between the net price, i.e., the prices without the tax, and the gross price, i.e., the price with the tax, and it can also return just the tax amount for each product.

Here's the final result for this section:

<Codesandbox slug="github/saleor/tutorial-walkthrough/tree/prices-pricing-range" />

Move the vertical sidebar to see the code.
