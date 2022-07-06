---
pos: 1
title: Installing Saleor Storefront
description:
next:
  path: /setup/saleor-graphql-playground/
---

This tutorial provides two ways of installing the storefront:

1. Creating Saleor Storefront using Saleor CLI (as easy as can be).
2. Installing Next.js, Typescript and other libraries manually (medium difficulty).

## Using Saleor CLI

Building a storefront from ground up may seem like a drudgery. Thus, Saleor provides an excellent tool to speed up the process - the Saleor CLI tool. With the help of CLI, creating a sample storefront is quick and painless:

1. Make sure you have the [Node.js](https://nodejs.org/) runtime installed on your machine.
2. In the terminal, run `npm i -g saleor-cli` to install [the Saleor CLI tool](https://docs.saleor.io/docs/3.x/cli).
3. Login or register to Saleor and create the environment. You can do it in your Saleor account in the web browser or by using the CLI. For more details, please follow the guide in [Getting Started with Saleor CLI](/cli/getting-started/).
4. Navigate to the directory you want to install your storefront in.
5. Run

`saleor storefront create my-storefront`

6. Choose the environment in which you want to install this storefront instance and hit Enter.

This command will install a Saleor Storefront project called `my-storefront` on your machine and start the local server. The application will already be preinstalled with:

- [Typescript](https://www.typescriptlang.org/)
- [Tailwind CSS](https://tailwindcss.com/)
- [GraphQL Code Generator](https://www.graphql-code-generator.com/)
- [Apollo GraphQL client](https://www.apollographql.com/docs/react/)

...and other goodies that will support you in extending the storefront further.

![saleor cli creating storefront](/images/saleor-cli-storefront.png)

Now, you can skip the remaining steps in the setup and jump straight into the next section. However, if you wish you can choose to go through the manual setup and configure all the nuts and bolts on your own, so that you have a better understanding of the base for building e-commerce applications with Next.js & Saleor using TypeScript.

## Doing the Manual Setup

If you prefer to set up your development environment manually, first you need to install Node.js and Next.js on your computer:

1. Install the [Node.js](https://nodejs.org/) runtime.
2. Head over to the directory you want to install your storefront in and initialize the application using the following command:

With `npm`:

```
npx create-next-app@latest --ts my-storefront
```

or with `pnpm` (you need version `6.13.0` or higher):

```
pnpm create next-app --ts my-storefront
```

This command will bootstrap the latest version of Next.js template app wired with Typescript. For more details on specifying different options when using Create Next App head over to the [docs](https://nextjs.org/docs/api-reference/create-next-app).

![create next app](/images/create-next-app.png)

Please follow the next steps in this section to finish the manual setup.
