---
pos: 2
title: Checkout Creation 
description: 
prev:
  path: /checkout/overview/
next:
  path: /checkout/checkout-page/
---

## Creating an empty checkout 

You can create a checkout in Saleor using the `createCheckout` mutation. We need to specify the `email` address of the owner of this checkout, its initial content provided via `lines` (this is a list of product variants along with their quantities) and the `channel`.

In the following example, we are creating an empty cart 

```graphql
# graphql/CreateCheckout.graphql
mutation CreateCheckout {
  checkoutCreate(
    input: {
      channel: "default-channel"
      email: "customer@example.com"
      lines: []
    }
  ) {
    checkout {
      token
    }
    errors {
      field
      code
    }
  }
}
```

In response to this checkout creation, we want to return the `token` that will be used to refer to this checkout from the client.  We also specify that the mutation should return errors if there are any.

Open the GraphQL Playground and try to execute this mutation.

**IMAGE**

Let's put this mutation in `graphql/CreateCheckout.graphql`. Don't forget to run your `generate` script or make sure it's running in the `watch` mode.

## Initializing the checkout in React.js

In our React.js application we need to trigger either create a new checkout or reuse the existing one. In order to distinguish between new and returning user, we will keep the checkout `token` client-side in the local storage. If the local storage is empty, it means there is no checkout yet and it must be created. Otherwise, we will use the `token` that found in the local storage to ask the Saleor server to fetch an existing checkout.

For local storage management we will adopt `useLocalStorage` hook from the `react-use` which is a collection of essential React Hooks.

```
npm install react-use
```

```
pnpm add react-use
```


Let's use `pages/_app.tsx` as a place to initialize the checkout in our React application. Since the checkout creation is performed via a GraphQL mutation, we need that the GraphQL client is already available within the React component tree. For that, we will create a *pass-through* component called `Root` that wraps the built-in `Component` provided by Next.js. With such setup in place we will be able to use GraphQL queries and mutations at the top level. 

```tsx
// pages/_app.tsx
import type { AppProps } from 'next/app'

import { useEffect } from "react";
import { useLocalStorage } from "react-use";
import { ApolloProvider } from '@apollo/client';

import '../styles/main.css';

import { apolloClient } from '@/lib';
import { useCreateCheckoutMutation } from '@/saleor/api';

const Root = ({ Component, pageProps }: AppProps) => {
  const [token, setToken] = useLocalStorage("token");
  const [createCheckout, { data, loading }] = useCreateCheckoutMutation();

  useEffect(() => {
    async function doCheckout() {
      const { data } = await createCheckout();
      const token = data?.checkoutCreate?.checkout?.token;

      setToken(token);
    }

    doCheckout();
  }, []);

  return <Component {...pageProps} token={token} />;
};

const MyApp = (props: AppProps) => 
  <ApolloProvider client={apolloClient}>
    <Root {...props} />
  </ApolloProvider>

export default MyApp;
```

`Root` reads the local storage to get the token of a checkout. If the token exists, it fetches an existing checkout, otherwise it creates a new one and sets its token in the local storage.

At this stage the checkout session exists in the application state as long as you don't refresh the page. We will make this session persisted between requests in the next sections.