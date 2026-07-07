---
title: "Salesforce Commerce Cloud (SFCC)"
timestamp: false
---

How an SFCC (B2C Commerce) store — SFRA or headless PWA Kit — integrates Whatmore. The
building blocks are the same as any non-Shopify store; this page maps them onto SFCC.

## 1. Connect your catalog

Whatmore reads your products through the SFCC Shopper Products API (SCAPI / OCAPI) and you
map the fields in the dashboard. Your product endpoint is typically:

```bash
curl "https://yourinstance.commercecloud.salesforce.com/.../products/PRODUCT_ID" \
  -H "Authorization: Bearer <sfcc_access_token>"
```

In [dashboard.whatmore.live](https://dashboard.whatmore.live/) select **Salesforce Commerce
Cloud**, enter the endpoint + token, and map fields to Whatmore's (title,
`client_product_id`, price, compare-at, product URL, image) — see
[Catalog API → Connect in the dashboard](/integrations/catalog-api#pull-initial-load-and-refresh).
*(Also push price/stock/image updates via the [Catalog API](/integrations/catalog-api#push-event-based-updates).)*

## 2. Embed the widget

In the dashboard, set up a surface, choose a template, and **copy the generated snippet**:

- **SFRA:** paste it into an ISML template (PDP, homepage), optionally packaged as a small
  **cartridge**.
- **PWA Kit / headless:** mount it in your React storefront.

The snippet is generated for your store — no hardcoded script URL.

## 3. Authentication

For order tracking, fetch a bearer token from `GET /auth/access-token?store_id=<store_id>`
server-side (store credentials in SFCC service config). See
[Authentication](/integrations/authentication).

## 4. Order tracking

On order confirmation, call [Order Tracking](/integrations/order-tracking) with the order items. The web
widget stores video-view / add-to-cart signals in `localStorage`, so the
[ready-to-use snippet](/integrations/order-tracking#ready-to-use-snippet-web) picks them up.

## Verify

- Products resolve in Whatmore (`GET /events/product/{client_product_id}`)
- Widget renders from the pasted snippet
- Order tracking fires on confirmation and attribution shows in the dashboard
