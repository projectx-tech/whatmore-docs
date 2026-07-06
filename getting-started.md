# Getting Started

Prerequisites and a checklist for integrating a non-Shopify storefront with Whatmore.

## 1. Get access

Your Whatmore contact provisions your store and issues:

- A **`store_id`** — identifies your store; used to get an access token and on every API call.
- A **Brand ID** — used by the [App SDK](app-sdk.md) to render your surfaces.
- Access to the Whatmore **dashboard** for managing videos and tagging products.

You obtain a **bearer access token** yourself from `GET /auth/access-token?store_id=<store_id>`
and send it on every API call. See [Authentication](authentication.md).

## 2. Environments

Production and staging issue **separate `store_id`s and tokens**, so integration testing
never touches live data. Base URLs are provided at onboarding.

## 3. Integration checklist

- [ ] `store_id` + Brand ID received; access token obtained ([Auth](authentication.md))
- [ ] Products synced via [`POST /product`](catalog-api.md#add-a-product)
- [ ] Price/stock updates wired via [`PUT /v1/product`](catalog-api.md#update-price--stock)
- [ ] [App SDK](app-sdk.md) surfaces embedded; view / add-to-cart signals collected
- [ ] [`POST /external-shop-order-tracking/private`](order-tracking.md) called on order completion, with SDK signals
- [ ] Attribution verified in the dashboard

## Mental model

- **You push to Whatmore.** You sync your catalog and report orders; there is no
  Whatmore-hosted API you must expose an endpoint for.
- **Reference products by your own `client_product_id`** (commonly the product URL) — no
  separate id mapping.
- **Attribution is per order item** — the SDK's video-view / add-to-cart signals are
  matched to line items on the order-tracking call.
