---
title: "Magento / Adobe Commerce"
timestamp: false
---

How a Magento 2 / Adobe Commerce store integrates Whatmore — connect your catalog, embed the
widget, and report orders. The building blocks are the same as any non-Shopify store; this
page maps them onto Magento.

## 1. Connect your catalog

Whatmore reads your products through Magento's REST API and you map the fields in the
dashboard. Your product endpoint is typically:

```bash
curl https://yourstore.com/rest/V1/products/SKU \
  -H "Authorization: Bearer <magento_integration_token>"
```

In [dashboard.whatmore.live](https://dashboard.whatmore.live/) select **Magento**, enter the
endpoint + token, and map fields to Whatmore's (title, `client_product_id`, price,
compare-at, product URL, image) — see
[Catalog API → Connect in the dashboard](/integrations/catalog-api#pull-initial-load-and-refresh).
*(Also push price/stock/image updates via the [Catalog API](/integrations/catalog-api#push-real-time-updates).)*

## 2. Embed the widget

In the dashboard, set up a surface, choose a template, and **copy the generated snippet**.
Paste it into a custom `.phtml` template or a CMS block where you want the surface (PDP,
homepage, category). The snippet is generated for your store — no hardcoded script URL.

## 3. Authentication

For order tracking, fetch a bearer token from `GET /auth/access-token?store_id=<store_id>`
server-side (store `store_id` / token in Magento secure config). See
[Authentication](/integrations/authentication).

## 4. Order tracking

On order placement (e.g. `checkout_submit_all_after` / the order-success page), call
[Order Tracking](/integrations/order-tracking) with the order items. The web widget stores video-view /
add-to-cart signals in `localStorage`, so the
[ready-to-use snippet](/integrations/order-tracking#ready-to-use-snippet-web) picks them up.

## Verify

- Products appear in your Whatmore dashboard catalog
- Widget renders from the pasted snippet
- Order tracking fires on the success page and attribution shows in the dashboard
