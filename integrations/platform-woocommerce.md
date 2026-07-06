# WooCommerce

How a WooCommerce (WordPress) store integrates Whatmore. The building blocks are the same as
any non-Shopify site — this page maps them onto WooCommerce specifics.

{% hint style="info" %}
Snippets are illustrative; final shapes are confirmed at onboarding.
{% endhint %}

## 1. Embed the widget

Add the Whatmore web widget where you want each surface (product page, homepage, category).
Common options:

- A block / shortcode in the page or a **Custom HTML** widget.
- A snippet in your theme template (e.g. `single-product.php`) or via a code-snippets
  plugin.

```html
<div id="whatmore-carousel" data-brand-id="YOUR_BRAND_ID"></div>
<script src="https://cdn.whatmore.ai/sdk.js" async></script>
```

The widget emits video-view / add-to-cart signals your order code forwards at checkout.

## 2. Authentication

Fetch a bearer token from `GET /auth/access-token?store_id=<store_id>` server-side (store the
`store_id` / token in WordPress options / secrets, e.g. via `wp_options` or environment).
See [Authentication](authentication.md).

## 3. Catalog sync

Map WooCommerce products onto the [Catalog API](catalog-api.md):

- On product save (`save_post_product` / `woocommerce_update_product` hooks) → `POST /product`
  with `product_link`, `client_product_id` (your WooCommerce product id or URL), `price`,
  `compare_price` (regular vs sale price), `currency`, `title`, `thumbnail_image`, and
  `product_metadata` (`sku`, `variant_id`).
- For an initial load, use bulk-by-URL:
  [`POST /product/upload/bulk`](catalog-api.md#bulk-import-large-catalogs).
- On price/stock change → `PUT /v1/product` with the new `price` / `inventory`.

## 4. Order tracking

On order completion (`woocommerce_thankyou` or the `woocommerce_order_status_completed`
hook), call [`POST /external-shop-order-tracking/private`](order-tracking.md) with the order
items and the `whatmore_video_view` / `whatmore_add_to_cart` signals carried from the widget.

## Verify

- Widget renders and signals reach your order code
- Products appear in Whatmore (`GET /events/product/{client_product_id}`)
- Order tracking fires on the thank-you page and attribution shows in the dashboard
