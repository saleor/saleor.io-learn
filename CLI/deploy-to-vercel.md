---
pos: 5
title: Deploy your Saleor App to Vercel with Github
description:
prev:
  path: /cli/deploy-to-vercel/
next:
  path: /cli/checkout-integration/
---

# Deploy your Saleor App to Vercel with Github

Vercel is a robust platform that enables hosting of static sites and frontend frameworks. It is built to integrate with a headless content, commerce, or database. It provides a smooth integration with Next.js and thus in this guide we'll try and set up the Saleor template App to be hosted on this platform.
The most convenient way (although not [the only one](https://vercel.com/docs/concepts/deployments/overview#making-deployments)) of deploying an App to Vercel is by using Git.

## Prerequisites

In order to proceed with the deployment, please make sure you have:

- Registered a Github account.
- Registered a Vercel account (it is best to authenticate with Github).
- Haved Saleor CLI installed.

## Step 1. Creating a template App.

In your Terminal, go to a directory on your computer you'd like to install the App in and type: `saleor app create [name]`. After a few moments, you'll have your developer environment set up and ready.

## Step 2. Initialize git.

1. `cd` to your App's folder in the Terminal.
2. Run `git init`.

This command will reinitialise a local git repository with your project.

## Step 3. Creating a new remote repository at Github and pushing to remote.

1. In Github.com create a new remote repository and copy/paste the instructions to push an existing local repository to remote.

```
git add .
git commit -m"First commit."
git remote add origin <url-to-your-remote-repo>
git branch -M main
git push -u origin main
```

## Step 4. Deploying to Vercel.

1. In Vercel, click New Project.
2. Choose to Import a Github repository with the name of your App.
3. At the configuration page, override the install command to `pnpm install`.
4. Hit Deploy button.

Your App should be deployed, and you can visit it.
![deployed app](/images/deploy-to-vercel.png)
You can also use the url of your deployed App in the process of App installation as described in [Creating Apps with Saleor CLI](#).
