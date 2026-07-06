# Salesforce Commerce Cloud (SFCC)

How an SFCC (B2C Commerce) store — SFRA or headless PWA Kit — fulfils the three
[integration tracks](README.md#the-three-integration-tracks). The APIs are identical to any
non-Shopify store; this page maps them onto SFCC specifics.

## 1. App SDK — embed the surfaces

- **SFRA:** include the [Web SDK](app-sdk.md#web-sdk-headless--non-mobile) via an ISML
  template (PDP, homepage), optionally packaged as a small **cartridge**.
- **PWA Kit / headless:** mount the SDK component in your React storefront.

```html
<div id="whatmore-carousel" data-brand-id="YOUR_BRAND_ID"></div>
<script src="https://cdn.whatmore.ai/sdk.js" async></script>
```

The SDK emits the video-view / add-to-cart signals your order code forwards at checkout.

## 2. Authentication

Fetch a bearer token from `GET /auth/access-token?store_id=<store_id>` server-side (store
credentials in SFCC service config). See [Authentication](authentication.md).

## 3a. Catalog sync (push products to Whatmore)

Map SFCC product data onto the [Catalog API](catalog-api.md):

- From a catalog job or product hook → `POST /product` with `product_link`,
  `client_product_id` (your SFCC product id or URL), `price`, `compare_price`, `currency`,
  `title`, `thumbnail_image`, and `product_metadata` (`sku`, `variant_id`).
- On price/inventory change → `PUT /v1/product` with the new `price` / `inventory`.

## 3b. Order tracking

On order confirmation, call
[`POST /external-shop-order-tracking/private`](order-tracking.md) with the order items and
the `whatmore_video_view` / `whatmore_add_to_cart` signals carried from the SDK.

## Verify

- Surfaces render in SFRA/PWA and SDK signals reach your order code
- Products appear in Whatmore (`GET /events/product/{client_product_id}`)
- Order tracking fires on confirmation and attribution shows in the dashboard
