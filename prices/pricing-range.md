---
pos: 2
title: Pricing Range 
description: 
prev:
  path: /prices/overview/
next:
  path: /prices/pricing-variants/
---

Since products have variants, the product price may be different dependening on the particular variant. For that reason, it is not possible to ask for a product price. Instead, the product exposes a **range of prices** as the `pricing` field on the `Product` type. The range is represented as a tuple with the lowest and the highest price in that range.  If all the variants have the same price, the both elements of the tuple are equal.

## Getting the price range of a product

Let's use the `products` query again, but this time we will only list the lowest and the highst price for each product in the collection. 

```graphql{6-18}
{
  products(first: 100, channel: "default-channel") {
    edges {
      node {
        name
        pricing {
          priceRange {
            start {
              gross {
                amount
              }
            }
            stop {
              gross {
                amount
              }
            }
          }
        }
      }
    }
  }
}
```

The `pricing` field gives a range descriptor (`priceRange`) where we can request the lowest (`start`) and the highest (`stop`) price among each variants of a product. Additionally, the price of a product is always available as a gross, net or just the tax value. In the example above we are interested in displaying the final price to the customer, thus we select the gross value.

Try this query in the GraphQL Playground. In response you will get a collection of product with names and their price ranges. Most elements will have the same value for the `start` and `stop` fields, meaning that the product variants don't differ in price.

```json
{
  "data": {
    "products": {
      "edges": [
        {
          "node": {
            "id": "UHJvZHVjdDoxMDc=",
            "name": "Saleor Polo Shirt",
            "pricing": {
              "priceRange": {
                "start": {
                  "gross": {
                    "amount": 5
                  }
                },
                "stop": {
                  "gross": {
                    "amount": 5
                  }
                }
              }
            }
          }
        },
        {
          "node": {
            "id": "UHJvZHVjdDoxMDg=",
            "name": "Saleor T-Shirt",
            "pricing": {
              "priceRange": {
                "start": {
                  "gross": {
                    "amount": 4.5
                  }
                },
                "stop": {
                  "gross": {
                    "amount": 4.5
                  }
                }
              }
            }
          }
        },
        // ...
      ],
    }
  }
}
```

## Displaying prices in the collection page

Let's update the product collection pages so that there is also the price information displayed along with each product. For simplicity, we will display a single price if both values in the price range are the same, otherwise we will display two values: the lowest and the highest price for each product.

Open the `FilterProducts` query located in `graphql/queries` and adjust it as shown below:

```graphql{14-27}
# graphql/queries/FilterProducts.graphql
query ProductCollection($first: Int = 4, $after: String) {
  products(first: $first, channel: "default-channel", after: $after) {
    edges {
      node {
        id
        name
        thumbnail {
          url
        }
        category {
          name
        }
        pricing {
          priceRange {
            start {
              gross {
                amount
              }
            }
            stop {
              gross {
                amount
              }
            }
          }
        }
      }
    }
    pageInfo {
      hasNextPage
      hasPreviousPage
      startCursor
      endCursor
    }
    totalCount
  }
}
```

Now, let's modify the `ProductElement` component to display the price range of a product along with the existing product information.

```tsx{8,19,21-23,33-39}
// components/ProductElement.tsx
import React from 'react';
import Link from 'next/link';

const styles = {
  card: 'bg-white border',
  summary: 'px-4 py-2 border-gray-100 bg-gray-50 border-t',
  description: 'flex justify-between items-center',
  title: 'block text-lg text-gray-900 truncate',
  category: 'block text-sm font-medium text-gray-500',
  image: {
    aspect: 'aspect-h-1 aspect-w-1',
    content: 'object-center object-cover'
  }
}

import { Product } from '@/saleor/api'

type Props = Pick<Product, 'id' | 'name' | 'thumbnail' | 'category' | 'pricing'>;

export const ProductElement = ({ id, name, thumbnail, category, pricing }: Props) => {
  const lowestPrice = pricing?.priceRange?.start?.gross.amount ?? 0;
  const highestPrice = pricing?.priceRange?.stop?.gross.amount ?? 0;

  return (
    <li key={id} className={styles.card}>
      <Link href={`/product/${id}`}>
        <a>
          <div className={styles.image.aspect}>
            <img src={thumbnail?.url} alt="" className={styles.image.content} />
          </div>
          <div className={styles.summary}>
            <div className={styles.description}>
              <div>
                <p className={styles.title}>{name}</p>
                <p className={styles.category}>{category?.name}</p>
              </div>
              <div>{lowestPrice == highestPrice ? highestPrice : `${lowestPrice} - ${highestPrice}`}</div>
            </div>
          </div>
        </a>
      </Link>
    </li>
  );
}
```

As a result, you should see the price displayed on the right side of each product element in the collection, as shown in the image below:

![Product Collection with Prices](/images/products-price-range.png)