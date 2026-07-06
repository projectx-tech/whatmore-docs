---
title: "WooCommerce"
---

How a WooCommerce (WordPress) store integrates Whatmore. You connect your catalog via
WooCommerce's REST API, embed the widget with a dashboard-generated snippet, and report
orders for attribution.

## 1. Connect your catalog

Whatmore reads your products through the WooCommerce REST API and you map the fields in the
dashboard — no export needed.

**a. Generate WooCommerce API keys**

- WordPress admin → **WooCommerce → Settings → Advanced → REST API → Add key**
- Description: `Whatmore Integration`; Permissions: **Read/Write**
- Copy the **Consumer key** (`ck_…`) and **Consumer secret** (`cs_…`)

Your product endpoint looks like:

```bash
curl https://yourstore.com/wp-json/wc/v3/products/PRODUCT_ID \
  -u ck_your_consumer_key:cs_your_consumer_secret
```

**b. Map fields in the dashboard**

In [dashboard.whatmore.live](https://dashboard.whatmore.live) select **WooCommerce**, enter
your endpoint + keys, and map the fields Whatmore fetches from a sample response (see
[Catalog API → Connect in the dashboard](/integrations/catalog-api#connect-in-the-dashboard-recommended)):

| Whatmore field | WooCommerce field |
| -------------- | ----------------- |
| Product title | `name` |
| Product ID / SKU | `id` |
| Price | `price` |
| Compare-at / MRP | `regular_price` |
| Product URL | `permalink` |
| Product image | `images[0].src` |
| Currency | set manually |

## 2. Embed the widget

In the dashboard, set up a surface (e.g. Homepage Video Carousel), choose a template, and
**copy the generated snippet**. Paste it where you want the surface — a block, a **Custom
HTML** widget, or a theme template (e.g. `single-product.php`). No hardcoded script URL; the
snippet is generated for your store.

## 3. Authentication

For order tracking you need a `store_id` and a bearer token from
`GET /auth/access-token?store_id=<store_id>`. See [Authentication](/integrations/authentication).

## 4. Order tracking

On order completion (`woocommerce_thankyou` or the `woocommerce_order_status_completed`
hook), call [Order Tracking](/integrations/order-tracking) with the order items. Use the
[ready-to-use web snippet](/integrations/order-tracking#ready-to-use-snippet-web) — the widget already
stores video-view / add-to-cart signals in `localStorage`, so the snippet picks them up
automatically.

## Verify

- Products resolve in Whatmore (`GET /events/product/{client_product_id}`)
- Widget renders from the pasted snippet
- Order tracking fires on the thank-you page and attribution shows in the dashboard
