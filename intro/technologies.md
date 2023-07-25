---
pos: 3
title: Technologies
description:
---

Our storefront will be built in React.js using the Next.js framework. Those two solutions will be complemented with TypeScript, GraphQL & Apollo Client and Tailwind CSS. We aim at Vercel and Netlify for easy deployment.

## React.js

[React](https://reactjs.org/) is a JavaScript library for building user interfaces. It makes creating interactive UIs simple and streamlined. React is specifically designed for single-page applications. Its goal is to be fast, scalable, and simple.

## Next.js

[Next.js](https://nextjs.org/) is a React.js framework built on top of Node.js. It enables functionalities such as server-side rendering and generating static websites. The Next.js framework utilizes JAMstack architecture, which distinguishes between front-end and back-end and allows for efficient front-end development that is independent of any back-end APIs.

## Vercel

[Vercel](https://vercel.com/docs) is a deployment platform known for its great developer experience. It takes care of the hard things: deploying instantly, scaling automatically, and serving personalized content around the globe. As Vercel team also created Next.js, Vercel provides production-grade hosting for Next.js websites with zero configuration. What's more, it has a rich library of templates developers can start with and enables continuous deployment setup with a Github repository within a few clicks.

## Netlify

[Netlify](https://www.netlify.com/) is a platform that offers comprehensive automation of modern web projects. It simplifies the process for developers to deploy and host a website with the focus on website performance. Netlify supports continuous delivery and continuous integration. It integrates with Github repository to automatically build the site, run plugins, and deploy. It also provides serverless functions (AWS Lambda Functions) to add additional back-end functionality. It is cost-effective, yet delivering rich features. It is considered the main alternative to Vercel.

## TypeScript

[TypeScript](https://www.typescriptlang.org/) is a strongly typed programming language that builds on JavaScript. It is a superset of JavaScript that provides optional static typing, classes, and interfaces. One of the biggest benefits of using TypeScript is to make IDEs provide a richer environment so that you can see common errors as you type the code easier and faster.

## GraphQL

[GraphQL](https://graphql.org/) is a query language and server-side runtime for APIs that prioritizes giving clients exactly the data they request and no more. GraphQL is designed to make APIs fast, flexible, and developer-friendly. As an alternative to REST, GraphQL lets developers construct requests that pull data from multiple data sources in a single API call. Additionally, GraphQL gives API maintainers the flexibility to add or deprecate fields without impacting existing queries. Developers can build APIs with whatever methods they prefer, and the GraphQL specification will ensure they function in predictable ways to clients.

## Tailwind CSS

[Tailwind CSS](https://tailwindcss.com/) is a utility-first CSS framework that allows you to build complex components from a constrained set of primitive utilities like `flex`, `pt-4`, `text-center`, and `rotate-90`, directly in your markup. The framework provides a wide variety of predefined classes where each has its own focused use. This approach makes it easier to quickly test and check how CSS styles are applied on web pages.

## Typesense

[Typesense](https://typesense.org/) is an open source search engine that is optimised for instant sub-50ms searches, while providing an intuitive developer experience. It is equipped with a number of useful features, such as typo tolerance, faceting and filtering and geo search by default which makes it a tempting alternative to Algolia or ElasticSearch.

## React InstantSearch

With [InstantSearch.js](https://www.algolia.com/doc/guides/building-search-ui/what-is-instantsearch/js/) UI library from Algolia, we can quickly build a search interface in our front-end app. It equips you with a complete production-ready search ecosystem based on widgets that are fully customizable. Moreover, InstantSearch.js comes with many flavours, one of them being React Instant Search, which we will use in this guide.

## PNPM

[PNPM](https://pnpm.io/) is a modern, community-made package manager. It handles the installation, upgrading, and removal of computer software packages. It is considered an alternative to [NPM](https://www.npmjs.com/). It was built on top of it to simplify the installation process of packages in Node applications. PNPM follows the same principles as NPM but it has some additional features that makes it more powerful than its predecessor:

1. PNPM is faster and more efficient. The algorithm of PNPM does not use a flatten dependency tree. It groups all dependencies by symlinks (or junctions in Windows), but retains all the dependencies. This approach makes it easier to implement, maintain, and requires less computation. Files inside node_modules are linked from a single content-addressable storage.
2. It is more secure. PNPM uses checksums and verifies the integrity of its code before executing it.
3. PNPM has built-in support for monorepos: multiple packages in a repository.

You can easily install PNPM in your Terminal with `npm install -g pnpm`.
