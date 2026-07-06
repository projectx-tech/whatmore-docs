# Magento / Adobe Commerce

How a Magento 2 / Adobe Commerce store fulfils the three
[integration tracks](README.md#the-three-integration-tracks). The APIs are the same for any
non-Shopify store — this page maps them onto Magento specifics.

## 1. App SDK — embed the surfaces

Add the [Web SDK](app-sdk.md#web-sdk-headless--non-mobile) via a custom `.phtml` template or
CMS block where you want each surface (PDP, homepage, category). The SDK emits the
video-view / add-to-cart signals your order code will forward at checkout.

```html
<div id="whatmore-carousel" data-brand-id="YOUR_BRAND_ID"></div>
<script src="https://cdn.whatmore.ai/sdk.js" async></script>
```

## 2. Authentication

Fetch a bearer token from `GET /auth/access-token?store_id=<store_id>` server-side (store
`store_id`/token in Magento secure config). See [Authentication](authentication.md).

## 3a. Catalog sync (push products to Whatmore)

Map Magento catalog data onto the [Catalog API](catalog-api.md):

- On product save/import → `POST /product` with `product_link`, `client_product_id` (your
  Magento product id or URL), `price`, `compare_price`, `currency`, `title`,
  `thumbnail_image`, and `product_metadata` (`sku`, `variant_id`).
- On price/stock change (Magento events or indexer hooks) → `PUT /v1/product` with the new
  `price` / `inventory`.

## 3b. Order tracking

On order placement (e.g. `checkout_submit_all_after` / order success), call
[`POST /external-shop-order-tracking/private`](order-tracking.md) with the order items and
the `whatmore_video_view` / `whatmore_add_to_cart` signals carried from the SDK through
checkout.

## Verify

- Surfaces render and the SDK signals reach your order code
- Products appear in Whatmore (`GET /events/product/{client_product_id}`)
- Order tracking fires on the success page and attribution shows in the dashboard
