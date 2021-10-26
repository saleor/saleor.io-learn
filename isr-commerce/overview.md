---
pos: 1
title: Overview 
description: 
next:
  path: /isr-commerce/adding-isr-product-page/
---

Incremental Static Regeneration (ISR) enables to statically generate Next.js application on a per-page basis, without the need to re-generate it entirely from scratch. In other words, with ISR the content of a page in Next.js can be generated on-demand at the application runtime, instead of generating it when the application is being statically built.

This Next.js feature is an excellent fit for e-commerce. Let's imagine that our storefront must show thousands of products. If we assume that **50ms** is needed to statically generate a product page, with a store showcasing **100,000** products, it will take almost **2 hours** to build the entire storefront. ISR provides us with flexibility regarding the build time of our storefront.

We can for decide generate a small amount of products ahead-of-time. For example, we may select a subset of **1,000** most popular products in our store. This will greatly reduce the time needed for the build. In this particular case from almost **2 hours** to just **1 minute**. And the products that are not included in that list will be generated on-demand and then cached.
