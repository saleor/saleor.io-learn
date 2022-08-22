---
pos: 6
title: Deploy your Saleor App to Netlify with Github
description:
prev:
  path: /cli/deploy-to-vercel/
next:
  path: /cli/checkout-integration/
---

MINIMUM SALEOR VERSION
3.5.10<br/>
MINIMUM SALEOR CLI VERSION
1.13

This tutorial will guide you through the process of deploying your Saleor App to Netlify, another popular platform, which provides various tools and features for automating web projects. You can read more about Netlify in the [Technologies](/intro/technologies#Netlify) section.

## Prerequisites

1. Install `saleor CLI` with `npm i -g saleor`.
2. Establish your [organisation and environment](/cli/getting-started/).
3. Create a Saleor App template. More on that [here](/cli/deploy-to-vercel/).
4. Set up a remote Git repository for your App at GitHub.com. More on that [here](/cli/deploy-to-vercel/).

## Step 1. Registering to Netlify.

Registering or logging into Netlify is possible through one of the providers listed at [their site](https://app.netlify.com/). For the purpose of this tutorial, let's use GitHub.

![Register or log in to Netlify.](/images/login-netlify.png)

## Step 2. Creating a project.

Once logged in, you will be redirected to your dashboard. Here, click **Add new site** and select **Import an existing project**.

![Import existing project.](/images/netlify1.png)

## Step 3. Connecting to Git provider.

1. Connect to Git provider.

![Connect to Git provider.](/images/netlify2.png)

2. Search and select your app repository from the list. If you don't see the list, click on **Configure the Netlify app on GitHub link** below and allow all / chosen repositories to be managed by Netlify.

![Pick the repository.](/images/netlify3.png)

## Step 4. Preparing your App for deployment.

1. On the next page, change the build command to:
   `npx pnpm i --store=node_modules/.pnpm-store && npx pnpm run build`

![Update build command.](/images/netlify4.png)

Netlify's build environment does not support `pnpm` out of the box. So, with this command we will kindly ask Netlify to use `npx` to install `pnpm` in the `node_modules` folder and run the `build` script with it.

2. In the Advanced settings, add two new variables:

- `NEXT_PUBLIC_SALEOR_HOST_URL`, take the value from `.env` file of your app
- `NPM_FLAGS`, set the value to `--legacy-peer-deps`.

![Add new variables.](/images/netlify5.png)

3. Hit Deploy site.

After a while, you will have your site live, which you can check by clicking **Open published deploy** button in the dashboard.

![Open published dashboard.](/images/netlify6.png)

You can use the `url` of your deployed App in the process of i.e. App installation as described in [Creating Apps with Saleor CLI](/cli/creating-apps/).
