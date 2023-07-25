---
pos: 1
title: Installing the Storefront Project
description:
next:
  path: /setup/saleor-graphql-playground/
---

This tutorial provides two ways of installing the storefront:

1. Creating a Saleor React Storefront instance using Saleor CLI (as easy as can be).
2. Installing Next.js, Typescript and other libraries manually (medium difficulty).

## Using Saleor CLI

Building a storefront from ground up may seem like a drudgery. Thus, Saleor provides a ready-made [Saleor React Storefront](https://github.com/saleor/react-storefront) (SRS) which you can play with and extend further. Additionally, with the help of Saleor CLI, you can quickly create a running instance of SRS:

1. Make sure you have the [Node.js](https://nodejs.org/) version at least 18 installed on your machine.
2. In the terminal, run `npm i -g saleor` to install [the Saleor CLI tool](https://docs.saleor.io/docs/3.x/cli).
3. Login or register to Saleor and create the environment. You can do it in your Saleor account in the web browser or by using the CLI. For more details, please follow the guide in [Getting Started with Saleor CLI](/cli/getting-started/).
4. Navigate to the directory you want to install your storefront in.
5. Run `saleor storefront create my-storefront`

This command will install a Saleor React Storefront project called `my-storefront` on your machine. To run the storefront - start the local server - go to the my-storefront directory `cd my-storefront` and start the development server `pnpm dev`. The application will already be preinstalled with:

- [Typescript](https://www.typescriptlang.org/)
- [Tailwind CSS](https://tailwindcss.com/)
- [GraphQL Code Generator](https://www.graphql-code-generator.com/)

...and other goodies that will support you in extending the storefront further.

Now, you can skip the remaining steps in the setup and jump straight into the next sections of this tutorial. However, if you wish you can choose to go through the manual setup and configure all the nuts and bolts on your own, so that you have a better understanding of the base for building e-commerce applications with Next.js & Saleor using TypeScript.

## Doing the Manual Setup

If you prefer to set up your development environment manually, first you need to install Node.js and Next.js on your computer:

1. Install the [Node.js](https://nodejs.org/) runtime. Use version at least 18.
2. Head over to the directory you want to install your project in and initialize the application using the following command:

With `npm`:

```
npx create-next-app@latest my-storefront
```

or with `pnpm` (you need version `8.x` or higher):

```
pnpm create next-app my-storefront
```

This command will bootstrap the latest version of Next.js template app wired with Typescript. For more details on specifying different options when using Create Next App head over to the [docs](https://nextjs.org/docs/api-reference/create-next-app).

After the successful installation, you will be able to start the development server:

1. In your Terminal, run `npm dev` or `pnpm dev` .
2. Visit `http://localhost:3000` in your browser to see it live.

![Create Next App.](/images/create-next-app.png)

Please follow the next steps in this section to finish the manual setup.
