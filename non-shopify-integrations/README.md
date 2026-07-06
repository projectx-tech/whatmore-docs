# Whatmore Integrations

Technical integration documentation for connecting a storefront to Whatmore's
shoppable-video and live-streaming platform **without** the native Shopify app — i.e.
Magento / Adobe Commerce, Salesforce Commerce Cloud, and custom / headless apps built on
the Whatmore App SDKs.

## The three integration tracks

Every non-Shopify integration is made up of the same three tracks. They can be built in
parallel.

| Track | What it covers |
| ----- | -------------- |
| **[Part 1 — App SDK](app-sdk.md)** | Rendering Whatmore's shoppable-video surfaces (carousel, PDP carousel, PDP floating card, celebrity pages) in your app/site |
| **[Part 2 — Authentication](authentication.md)** | Getting a bearer access token to call the Whatmore APIs |
| **[Part 3 — API Integrations](api-integrations.md)** | The [Catalog API](catalog-api.md) (sync your products to Whatmore) and [Order Tracking](order-tracking.md) (report purchases for attribution) |

## How the data flows

Your storefront integrates with Whatmore in one direction: **you push data to Whatmore.**
There is no Whatmore-hosted service you need to expose an API for.

```
  Your systems                                   Whatmore
  ────────────                                   ────────
  catalog        ── POST/PUT /product ─────────► Product catalog
  (products)                                     (price, stock, media)

  App SDK        ── renders surfaces ──────────► shoppable video
  (in your app)  ◄─ emits view/atc signals ───

  order backend  ── POST order-tracking ───────► Attribution
  (on purchase)     (with SDK signals)           (video → sale)
```

1. **Catalog** — you sync products **to** Whatmore with the [Catalog API](catalog-api.md),
   and keep price/stock current with `PUT /v1/product`.
2. **App SDK** — Whatmore's [SDK](app-sdk.md) renders video surfaces in your app and emits
   *video-view* and *add-to-cart* signals per product.
3. **Order tracking** — on order completion your backend calls the
   [Order Tracking](order-tracking.md) endpoint with the order items **and** the SDK
   signals, so Whatmore can attribute the purchase to the video that drove it.

## Key concepts

- **`store_id`** — your store's identifier on Whatmore; used to get an access token and on
  every API call.
- **`client_product_id`** — *your* product identifier (commonly the product URL). It is how
  you reference a product on every call, so Whatmore's records stay aligned with yours
  without a separate mapping layer.
- **Attribution is per order item** — the video-view / add-to-cart signals are matched to
  individual line items, so a single order can attribute different items to different
  videos (or none).

## Start here

1. **[Getting Started](getting-started.md)** — access token, `store_id`, checklist
2. **[Part 1 — App SDK](app-sdk.md)**
3. **[Part 2 — Authentication](authentication.md)**
4. **[Part 3 — API Integrations](api-integrations.md)** → [Catalog API](catalog-api.md) · [Order Tracking](order-tracking.md)
5. Your platform: **[Magento](magento.md)** · **[Salesforce Commerce Cloud](salesforce-commerce-cloud.md)**
