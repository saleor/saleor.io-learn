---
pos: 4
title: Taxes
description:
---

The product pricing is not only about the range of prices of its variants, but it also contains the information about the possible discounts, sales and taxes.

The prices in Saleor are not represented as the number values, but as structures containing the amount along with the currency and in some cases the tax information so that you can access the gross, net and just the tax value for a given product, as shown in the query below:

```tsx{5-14}
{
  productVariant(id: "UHJvZHVjdFZhcmlhbnQ6Mjgz", channel: "default-channel") {
    pricing(address: { country: PL }) {
      price {
        net {
          amount
        }
        gross {
          amount
        }
        tax {
          amount
        }
      }
    }
  }
}
```

The basic type for keeping prices with taxes for a single product variant is [`TaxedMoney`](https://docs.saleor.io/docs/3.x/developer/api-reference/objects/taxed-money) type:

```tsx
type TaxedMoney {
  currency: String!
  gross: Money!
  net: Money!
  tax: Money!
}
```

For specifying a range of prices with taxes you can use the [`TaxedMoneyRange`](https://docs.saleor.io/docs/3.x/developer/api-conventions/prices#taxedmoneyrange) type.

If you wish to set tax rates for a specific country, Saleor supports the following external tax providers: Avalara (USA) and Vatlayer (European Union). They are already integrated in the form of [plugins](https://docs.saleor.io/docs/3.x/dashboard/configuration/plugins/introduction).

For further reading about prices and taxes please head over to the [docs](https://docs.saleor.io/docs/3.x/developer/api-conventions/prices).
