---
pos: 1
title: Prerequisites 
description: 
stackblitz: saleor-tutorial-start
next:
  path: /setup/saleor-graphql-playground/ 
---

In this tutorial, we will be using Stackblitz, a development environment in the browser. This way, you won't need to install anything locally on your computer. After each section, we will finish with a working example as described in that section. The example will contain all the code required to run it, and once opened, it will launch the working application side by side.

Click the **Launch Editor** button at the top to launch the initial step of this tutorial. This is a Next.js application that displays the Saleor image in the center.

The Stackblitz environemnt for this setup only works in Chrome, Edge and Brave; the browser support for Firefox and Safari is coming soon.

## Local Development

If you prefer to use your own computer for development, you need to setup Node.js and Next.js on your computer. For that you need:

* the Node.js runtime
* the `create-next-app` wizard - it will be automatically installed when running `npx create-next-app`

Initialize the application using the following command:

With `npm`:

```
npx create-next-app --typescript saleor-tutorial
```

With `pnpm` (you need version `6.13.0` or higher):

```
pnpm dlx create-next-app --typescript saleor-tutorial
```
