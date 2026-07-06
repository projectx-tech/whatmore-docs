# BigCommerce

How a BigCommerce store integrates Whatmore — Stencil (hosted) or headless. The building
blocks are the same as any non-Shopify store; this page maps them onto BigCommerce.

## 1. Connect your catalog

Whatmore reads your products through the BigCommerce Catalog API and you map the fields in
the dashboard. Your product endpoint is typically:

```bash
curl "https://api.bigcommerce.com/stores/STORE_HASH/v3/catalog/products/PRODUCT_ID" \
  -H "X-Auth-Token: <bigcommerce_api_token>"
```

In [dashboard.whatmore.live](https://dashboard.whatmore.live) select **BigCommerce**, enter
the endpoint + token, and map fields to Whatmore's (title, `client_product_id`, price,
compare-at, product URL, image) — see
[Catalog API → Connect in the dashboard](catalog-api.md#connect-in-the-dashboard-recommended).
*(Prefer to push? Use the [Catalog API](catalog-api.md#catalog-api-automation).)*

## 2. Embed the widget

In the dashboard, set up a surface, choose a template, and **copy the generated snippet**:

- **Stencil:** paste it via **Script Manager** (Storefront → Script Manager) or a Stencil
  template (e.g. `product.html`).
- **Headless:** mount it in your storefront app.

The snippet is generated for your store — no hardcoded script URL.

## 3. Authentication

For order tracking, fetch a bearer token from `GET /auth/access-token?store_id=<store_id>`
server-side (store the `store_id` / token in your app's secure config). See
[Authentication](authentication.md).

## 4. Order tracking

On order completion (order-confirmation page, or a BigCommerce `store/order/*` webhook), call
[Order Tracking](order-tracking.md) with the order items. The web widget stores video-view /
add-to-cart signals in `localStorage`, so the
[ready-to-use snippet](order-tracking.md#ready-to-use-snippet-web) picks them up.

## Verify

- Products resolve in Whatmore (`GET /events/product/{client_product_id}`)
- Widget renders from the pasted snippet
- Order tracking fires on confirmation and attribution shows in the dashboard
