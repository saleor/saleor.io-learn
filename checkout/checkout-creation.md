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

You can create a checkout in Saleor using the `checkoutCreate` mutation. We need to specify the `email` address of the owner of this checkout, its initial content provided via `lines` (this is a list of product variants along with their quantities) and the `channel`.

1. In your code editor, direct to the `graphql` folder and create a `mutations` folder inside it.
2. There, create a `CheckoutCreate.graphql` file and copy/paste the mutation below.

```graphql
# graphql/mutations/CheckoutCreate.graphql
mutation CheckoutCreate {
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

When you execute this mutation, an empty checkout will be created. In response to this checkout creation, we want to return the `token` that will be used to refer to this checkout from the client. We also specify that the mutation should return errors if there are any.

3. Run the `generate` script in the Terminal or make sure it's running in the `watch` mode.

4. Open the GraphQL Playground and execute this mutation. You will see the token in the query results.

![Checkout created in the Playground.](/images/checkout-create-playground.png)

## Initializing the checkout in React.js

In our React.js application we need to trigger either creating a new checkout or reusing the existing one. In order to distinguish between new and returning user, we will keep the checkout `token` client-side in the local storage. If the local storage is empty, it means there is no checkout yet and it must be created. Otherwise, we will use the `token` that is kept in the local storage to ask the Saleor server to fetch an existing checkout.

For local storage management we will adopt `useLocalStorage` hook from the `react-use` which is a collection of essential React Hooks.

1. In your Terminal, in the root of your project, run:

```
npm install react-use
```

or

```
pnpm add react-use
```

Let's use `pages/_app.tsx` as a place to initialize the checkout in our React application. Since the checkout creation is performed via a GraphQL mutation, we need the GraphQL client to be already available within the React component tree. For that we will create a _pass-through_ component called `Root` that wraps the built-in `Component` provided by Next.js. With such a setup in place, we will be able to use GraphQL queries and mutations at the top level.

2. In your code editor, navigate to `pages/_app.tsx` file and update its contents:

```tsx
// pages/_app.tsx
import type { AppProps } from "next/app";
import { ApolloProvider } from "@apollo/client";

import { useEffect } from "react";
import { useLocalStorage } from "react-use";

import "../styles/main.css";

import { apolloClient } from "@/lib";
import { useCheckoutCreateMutation } from "@/saleor/api";

const Root = ({ Component, pageProps }: AppProps) => {
  const [token, setToken] = useLocalStorage("token");
  const [checkoutCreate, { data, loading }] = useCheckoutCreateMutation();

  useEffect(() => {
    async function doCheckout() {
      const { data } = await checkoutCreate();
      const token = data?.checkoutCreate?.checkout?.token;

      setToken(token);
    }

    doCheckout();
  }, [checkoutCreate, setToken]);

  return <Component {...pageProps} token={token} />;
};

const MyApp = (props: AppProps) => (
  <ApolloProvider client={apolloClient}>
    <Root {...props} />
  </ApolloProvider>
);

export default MyApp;
```

`Root` reads the local storage to get the token of a checkout. If the token exists, it fetches an existing checkout, otherwise it creates a new one and sets its token in the local storage.

At this stage, the checkout session exists in the application state as long as you don't refresh the page. We will make this session persist between requests in the following sections.
