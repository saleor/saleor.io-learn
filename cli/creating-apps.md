---
pos: 2
title: Creating Apps with Saleor CLI
description:
prev:
  path: /cli/getting-started/
next:
  path: /cli/abandoned-checkouts/
quizQuestions:
  - question: Which command in Saleor CLI will install the App in the Dashboard?
    answerType: radio
    feedback: "The correct answer is saleor app install."
    answerOptions:
      - answer: saleor app create
      - answer: saleor app list
      - answer: saleor app install
        isCorrect: true
---

SALEOR VERSION
3.4.2

## Overview

Saleor platform gives developers a powerful way to extend its capabilities by building `Apps`, which can then be integrated with the Saleor store. By developing Apps you can add new features to the store, create a better user experience or pull Saleor store data in response to webhook notifications or user input. To read more about Apps architecture and integration with Saleor, [read the Saleor docs](https://docs.saleor.io/docs/3.x/developer/extending/apps/key-concepts).

## What will I learn?

After finishing this guide, you'll have accomplished the following:

1. Generated a template for your first Saleor App.
2. Prepared your local development environment for App installation.
3. Installed your App in the Admin Dashboard.

## Prerequisites

Before you create your first App using CLI, you need to make sure to:

1. Install `saleor-cli`
2. Register an account in Saleor Cloud and be logged in
3. Create an organisation, a project and an environment

You may find the instructions for every of these steps in the [Getting Started with Saleor CLI guide](#).

## What will I build?

You will create your first Saleor App. Also, you will embed this app to your Admin Dashboard in the Saleor Cloud and be able to see its view at a dedicated app page. Since you haven't built any Saleor apps yet, it is a good idea to start with a template.

## Step 1. Installing the App template to scaffold the Abandoned Checkouts app

Saleor CLI provides developers with the possibility to use a ready-made example App. In this guide we will utilize CLI to create such a template.
In your Terminal, `cd` to the path where you want to install the template and type in:

`saleor app create abandoned-checkouts`

This command will install a Next.js app called "abandoned-checkouts" equipped with many tools that'll come in handy during the process of integration. After the successful completion of the installation process, CLI will automatically start the development server.

## Step 2. Preparing the App for integration with Saleor Dashboard.

### Updating App manifest.

App manifest informs Saleor API about the details of your app such as name, url and permissions. You can read more about the manifest in [the docs](https://docs.saleor.io/docs/3.x/developer/extending/apps/manifest).

1. Go to `pages/api/manifest` folder in your app template and locate the `manifest` object.

```ts
const  manifest = {
id:  "saleor.app",
...
```

2. Alter the permissions for the App:

```ts
...
permissions: ["MANAGE_CHECKOUTS"],
...
```

3. Add the `extensions` field under `webhooks`.

```jsx
...
webhooks,
extensions: [
	{
		label:  "Abandoned Checkouts",
		mount:  "NAVIGATION_ORDERS",
		target:  "APP_PAGE",
		permissions: ["MANAGE_CHECKOUTS"],
		url:  "/abandoned-checkouts",
	},
],
],
...
```

Here, we're giving the extension a label under which it is going to be visible in the navbar in the Admin Dashboard, the mounting place for the App (which is the Orders tab in the navbar) and permissions for the App. We want the App's view to be the `abandoned-checkouts` page.

### Updating the App's name.

1. Go to `package.json` and change the name of the App:

```ts
...
{
"name": "abandoned-checkouts",
}
...
```

2. Restart your local development server by typing `pnpm run dev`

## Step 3. Installing the App in the Admin Dashboard.

In order to integrate your App with Saleor Cloud it needs to have a public endpoint. Saleor CLI uses a technique called _tunneling_, to expose the App to the world. Installing your App in the Dashboard using CLI is pretty straightforward but prior to installation you need to make sure your **local development server is running.**

### Using `saleor app tunnel`

Go to your App's root folder in another Terminal and type in:

1.  `saleor app tunnel`
2.  In the wizard, confirm you want to install the App in the Dashboard by pressing `y`.

This command will provide a live url to your App and install the App in the Dashboard.

### Or setting the tunnel manually

If for any reason you are unable to use `saleor app tunnel`, you can set a tunnel manually. In order to do that you can use a third-party app like [localtunnel](https://theboroer.github.io/localtunnel-www/) or [ngrok](https://ngrok.com/). Follow their installation guides linked.
After setting up your tunnelling software, go to the App's root folder in the Terminal and:

1. Type in `saleor app install`.
2. Provide the `name` for the app: `abandoned-checkouts`.
3. Provide the `url` to the app's manifest:

_[yourdomain-set-with-tunneling-app]/api/manifest_

After a successful install, you can go to your Admin Dashboard and inspect your new App in the Apps tab and inside the Orders tab.

![Admin Dashboard with App installed](./app-installed.png)
