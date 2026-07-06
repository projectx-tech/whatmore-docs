# Magento / Adobe Commerce

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

In [dashboard.whatmore.live](https://dashboard.whatmore.live) select **Magento**, enter the
endpoint + token, and map fields to Whatmore's (title, `client_product_id`, price,
compare-at, product URL, image) — see
[Catalog API → Connect in the dashboard](catalog-api.md#connect-in-the-dashboard-recommended).
*(Prefer to push? Use the [Catalog API](catalog-api.md#catalog-api-automation).)*

## 2. Embed the widget

In the dashboard, set up a surface, choose a template, and **copy the generated snippet**.
Paste it into a custom `.phtml` template or a CMS block where you want the surface (PDP,
homepage, category). The snippet is generated for your store — no hardcoded script URL.

## 3. Authentication

For order tracking, fetch a bearer token from `GET /auth/access-token?store_id=<store_id>`
server-side (store `store_id` / token in Magento secure config). See
[Authentication](authentication.md).

## 4. Order tracking

On order placement (e.g. `checkout_submit_all_after` / the order-success page), call
[Order Tracking](order-tracking.md) with the order items. The web widget stores video-view /
add-to-cart signals in `localStorage`, so the
[ready-to-use snippet](order-tracking.md#ready-to-use-snippet-web) picks them up.

## Verify

- Products resolve in Whatmore (`GET /events/product/{client_product_id}`)
- Widget renders from the pasted snippet
- Order tracking fires on the success page and attribution shows in the dashboard
