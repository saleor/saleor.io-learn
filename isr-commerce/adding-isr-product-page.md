---
pos: 1
title: Adding ISR to Product Page 
description: 
prev:
  path: /isr-commerce/overview/
---

Let's use [Incremental Static Renegartion](https://vercel.com/docs/concepts/next.js/incremental-static-regeneration) for the product page in our storefront. In order to enable ISR, we need to make two adjustments in the product page located at `pages/product/[id].tsx`:

1. define a per-page revalidation time
2. define paths for pages to generate ahead-of-time

## Per-page Revalidation 

Open `pages/product/[id].tsx` and add the `revalidate` field to the `return` statement of `getStaticProps` as shown below:

```tsx{6}
export const getStaticProps: GetStaticProps = async ({ params }) => {
  return {
    props: {
      id: params?.id,
    },
    revalidate: 15,
  };
};
```

In this example, we define the revalidation time to be **15** seconds. This means that whenever a page changes, for example in response to a remote call, Next.js will trigger a regeneration of that page in background after 15 seconds. Once the page has been successfully generated, Next.js will invalidate the cache and show the updated product page. At the same time, if the background regeneration fails, the old page will be kept unchanged.

## Pages to generate at build-time

Our Saloer API provides a total of 12 products. Let's generate 4 product pages ahead-of-time, at build-time and the rest 8 product pages will be generated on-demand whenever there is an incoming request for one of these 8 product pages. 

We are still in `pages/product/[id].tsx` and we need to change the `getStaticPath` function as shown below:

```tsx{5,14}
export async function getStaticPaths() {
  const { data } = await apolloClient.query<ProductCollectionQuery>({
    query: ProductCollectionDocument,
    variables: {
      first: 4,
    }
  });
  const paths = data.products?.edges.map(({ node: { id } }) => ({
    params: { id },
  }));

  return {
    paths,
    fallback: 'blocking',
  };
}
```

In this scenerio we pre-fetch first 4 products to generate their pages ahead-of-time at build-time. And then, whenever the product page than these 4 pre-generated, we will block the request till the product page is generated using `fallback: 'blocking'`.

---

ISR doesn't change anything in the way our storefront is presented to the user, but now we have a control over which pages are generated at build-time and how long it takes to revalidate a product page when it changes.