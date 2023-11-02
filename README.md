# Hydrogen Bug Replication for Storefront API carts with both `errors` and `cart` in a single response

When a buyer has been associated to a Storefront API cart a specific `buyerIdentity` fragment can cause a Storefront API cart query or mutation response to have both a valid cart and errors attributes. This happens when asking for the `customer.lastIncompleteCheckout.availableShippingRates` of the `buyerIdentity` on the cart query or mutation response. Here's an example of a `BuyerIdentity` graphql fragment asking for the info that causes a `Shipping Address Can't Be Blank` error unless a shipping address has been previously provided at checkout for the cart.

```graphql
fragment buyer on BuyerIdentity {
  customer {
    lastIncompletCheckout {
      availableShippingRates {
        shippingRates {
          handle
        }
      }
    }
  }
}
```

At the moment, Hydrogen `2023.10.0` (and earlier) throw an error for any presence of a Storefront API mutaion or query response that contains `errors` in the response. With the cart query happening at the root, this error causes the app to crash.

## Summary of Replication Project

This repo uses Hydrogen `2023.10.0` to provide a reproducilbe flow based on the demo starter template with tweaks to make reproducing the issue easier.

1. Added a custom cart handler in `server.js` that facilitates handling both line item add and buyer identity update of a Storefront API request in a single post.
2. Added a new cart action to `cart.jsx` route to invoke the custom cart handler function.
3. Added two new cart buttons to the PDP route. First, `Add To Cart with Buyer Identity` adds an item and buyer identity to the cart without using a query that will cause a cart error. Second `Add To Cart with Buyer Identity Without Crash Protection` that will switch the root cart query to one that will cause an error. These two buttons act like toggles setting a cookie `crashprotection` that signals to `server.js` to use a cart query that causes an error. `crashprotection=off` will cause the error, while the `crashprotection` cookie not being present, or any other value than off will force the use of a safe cart query.
