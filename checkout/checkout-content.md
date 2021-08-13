---
pos: 4
title: Checkout Content
description: 
---

So far the checkout page that displays the content of the cart is static. We hardcoded all the elements on the page to focus on its structure. In this section, we will connect the checkout object provided by Saleor with what's displayed on the screen when visiting the `/cart` route. This way the cart page will be able to dynamically display what's currently in the cart.

## Getting the checkout content from Saleor 

The client uses the `token` to reference the checkout object associated with the current session. At any point in time, we can use this token to fetch the checkout content. Let's start with defining a query for that. We will call it `CheckoutByToken`

```graphql
query CheckoutByToken($checkoutToken: UUID!) {
  checkout(token: $checkoutToken) {
    id
    email
    lines {
      id
      totalPrice {
        gross {
          amount
          currency
        }
      }
      variant {
        product {
          id
          name
          slug
          thumbnail {
            url
          }
        }
        pricing {
          price {
            gross {
              amount
              currency
            }
          }
        }
        name
      }
    }
    totalPrice {
      gross {
        amount
        currency
      }
    }
  }
}
```

