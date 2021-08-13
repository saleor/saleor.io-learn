---
pos: 1
title: Overview 
description: 
---

# Pricing Overview 

Since products have variants, the product price may be different dependening on the variant. For that reason, it is not possible to ask for a product price and the product exposes a range of prices as the `pricing` field on the `Product` type. The range is represented as a tuple with the lowest and the highest price in that range.  If all the variants have the same price, the both elements of the tuple are equal.

Now we know how to fetch products from GraphQL API and how to add some of those products to a cart in order to initiate the checkout process. The next step is to understand the pricing. Products have prices, but it is also possible a product depending on its size or shape has some different prices. In that case we talk about the price range of a given product. 

The price for each product contains the tax (which may be also different for different countries). Saleor distinguishes between the net price, i.e. the prices without the tax, the gross price i.e. the price with the tax, and it can also the return just the tax amount for each product. 