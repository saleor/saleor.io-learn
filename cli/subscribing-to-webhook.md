---
pos: 4
title: Subscribing To a HTTPS Webhook
description:
prev:
  path: /cli/abandoned-checkouts/
next:
  path: /cli/deploy-to-vercel/
quizQuestions:
  - question: Which command generates the webhook handler?
    answerType: radio
    answerOptions:
      - answer: saleor app generate webhook
        isCorrect: true
      - answer: saleor webhook create
    feedback: With saleor webhook create you create and active the webhook at Saleor.
---

SALEOR VERSION
3.4.2

After you have installed your first App and learned how to developed its new features, it is time to extend its capabilities. Saleor platform allows developers to use webhooks, which can be set to notify the App about certain events, i.e. creating a new order, updating a category, sending an invoice and many more. The architectural details of webhooks and the complete list of webhook events are available in the [docs](https://docs.saleor.io/docs/3.x/developer/extending/apps/asynchronous-webhooks).

## What will I learn?

After finishing this guide, you'll have accomplished the following:

1. Generated a webhook handler using CLI.
2. Created a webhook for the installed App.
3. Tested and verified the webhook endpoint of type HTTPS.

## What will I build?

You will extend the Saleor template App with the use of a webhook that will send notifications over the HTTPS every time a new order is created.

## Prerequisites

1. You know the concept of webhooks.
2. You're familiar with how webhooks work with Saleor API.
3. You're authenticated with the Saleor Cloud.
4. You've created and installed an App for testing webhook integration.

## Step 1. Creating a webhook handler.

In this step you're going to generate a boilerplate code to handle incoming webhook notifications in your App. The data is going to be displayed in the console.

1. In your Terminal type in: `saleor app generate webhook`.
2. Using arrow keys navigate to `ORDER_CREATED` event and press Enter.

You can now inspect the newly created handler in `api/webhooks/order-created.ts`file. As you can see for now, the handler's job is simply to log out the request body in the console.

## Step 2. Creating a webhook in Saleor.

Let's hook up our handler to the webhook instance created in Saleor.

1. In the Terminal type in: `saleor webhook create`.
2. Provide a name for the webhook, i.e. "order-created".
3. From the web browser copy the url of your App's live instance and add the path to your webhook handler:

`https://[app-in-your-environment.saleor.live]/api/webhooks/order-created`

4. Paste the link to Target URL prompt.
5. Optionally, provide a Secret to secure the operation.
6. Using arrow keys navigate to ORDER_CREATED event and press Space to select it. Also, check the DRAFT_ORDER_CREATED event as well.
7. Press Enter to confirm selections.
8. Skip the synchronous events selection by pressing Enter.
9. Confirm webhook activation by pressing Enter.
10. Optionally, provide a string with the subscription query.

After a successful install, you will be provided with the webhook id and are ready to test and verify its functionality.

## Step 3. Testing the webhook.

For your webhook to send a notification to the handler, you need to trigger an action of creating a new order. You can use your Saleor Dashboard to initiate it.

1. Go to your Saleor Dashboard and click the Orders tab.
2. On the right side, click Create Order.
3. Choose the Channel and Confirm.
4. Go to your code editor and check up the local server console.

You should be provided with an object describing your newly triggered action:

```ts
{
    type: 'Order',
    id: 'T3JkZXI6YTZiYTg3MmQtMjU1OS00YWFlLWFjZDItZmMzZWE1MTJkY2Rm',
    channel: {
      type: 'Channel',
      id: 'Q2hhbm5lbDox',
      slug: 'default-channel',
      currency_code: 'USD'
    },
    shipping_method: null,
    shipping_address: null,
    billing_address: null,
    discounts: null,
    token: 'a6ba872d-2559-4aae-acd2-fc3ea512dcdf',
    user_email: '',
    created: '2022-06-01T08:37:33.319Z',
    original: 'T3JkZXI6Tm9uZQ==',
    lines: [],
    fulfillments: [],
    collection_point: null,
    payments: [],
    meta: {
      issued_at: '2022-06-01T08:37:33.373027+00:00',
      version: '3.3.12',
      issuing_principal: [Object]
    },
    private_metadata: {},
    metadata: {},
    status: 'draft',
    language_code: 'en',
    origin: 'draft',
    shipping_method_name: null,
    collection_point_name: null,
    shipping_price_net_amount: '0.00',
    shipping_price_gross_amount: '0.00',
    shipping_tax_rate: '0.0',
    total_net_amount: '0.00',
    undiscounted_total_net_amount: '0.00',
    total_gross_amount: '0.00',
    undiscounted_total_gross_amount: '0.00',
    weight: '0.0:kg'
  }
```
