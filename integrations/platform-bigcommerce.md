# BigCommerce

How a BigCommerce store integrates Whatmore — Stencil (hosted) or headless. The building
blocks are the same as any non-Shopify site; this page maps them onto BigCommerce specifics.

{% hint style="info" %}
Snippets are illustrative; final shapes are confirmed at onboarding.
{% endhint %}

## 1. Embed the widget

- **Stencil:** add the Whatmore web widget via **Script Manager** (Storefront → Script
  Manager) or a Stencil template (e.g. `product.html`).
- **Headless:** mount the widget in your storefront app.

```html
<div id="whatmore-carousel" data-brand-id="YOUR_BRAND_ID"></div>
<script src="https://cdn.whatmore.ai/sdk.js" async></script>
```

The widget emits video-view / add-to-cart signals your order code forwards at checkout.

## 2. Authentication

Fetch a bearer token from `GET /auth/access-token?store_id=<store_id>` server-side (store the
`store_id` / token in your app's secure config). See [Authentication](authentication.md).

## 3. Catalog sync

Map BigCommerce catalog data onto the [Catalog API](catalog-api.md):

- From a catalog sync job or a **products/*** webhook → `POST /product` with `product_link`,
  `client_product_id` (your BigCommerce product id or URL), `price`, `compare_price` (sale
  price vs RRP), `currency`, `title`, `thumbnail_image`, and `product_metadata` (`sku`,
  `variant_id`).
- For an initial load, use bulk-by-URL:
  [`POST /product/upload/bulk`](catalog-api.md#bulk-import-large-catalogs).
- On price/inventory change → `PUT /v1/product`.

## 4. Order tracking

On order completion (order-confirmation page, or a BigCommerce `store/order/*` webhook),
call [`POST /external-shop-order-tracking/private`](order-tracking.md) with the order items
and the `whatmore_video_view` / `whatmore_add_to_cart` signals carried from the widget.

## Verify

- Widget renders and signals reach your order code
- Products appear in Whatmore (`GET /events/product/{client_product_id}`)
- Order tracking fires on confirmation and attribution shows in the dashboard
