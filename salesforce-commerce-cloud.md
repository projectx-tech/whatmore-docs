# Salesforce Commerce Cloud (SFCC)

How an SFCC (B2C Commerce) store — SFRA or headless PWA Kit — fulfils the three
[integration tracks](README.md#the-three-integration-tracks). The contracts are identical
to any non-Shopify store; this page maps them onto SFCC specifics.

{% hint style="warning" %}
Snippets are **illustrative**. Confirm final shapes jointly.
{% endhint %}

## 1. App SDK — embed the surfaces

- **SFRA (server-rendered):** include the [Web SDK](app-sdk.md#web-sdk-headless--non-mobile)
  via an ISML template (PDP, homepage). Consider packaging it as a small **cartridge** so
  it drops into the cartridge path.
- **PWA Kit / headless:** mount the SDK component in your React storefront.

```html
<div id="whatmore-carousel" data-brand-id="YOUR_BRAND_ID"></div>
<script src="https://cdn.whatmore.ai/sdk.js" async></script>
```

Wire `addToCart` to the SFCC **Basket** APIs (OCAPI `POST /baskets/{id}/items` or SCAPI
Shopper Baskets), carrying the
[Whatmore session IDs](app-sdk.md#session-identity--attribution) onto the basket line.

## 2. Authentication

Issue keys per [Authentication](authentication.md). Store the merchant API token protecting
your Product Details API in SFCC secure config (e.g. service credentials).

## 3a. Product Details API (you host)

Expose [`GET /api/products/{productId}`](product-details-api.md), backed by SFCC product
data (Shopper Products / OCAPI), where `productId` maps to the identifier in your SFCC
product URL. Return live price, availability (`inventory`), variants, and images.

## 3b. Webhooks (you post to Whatmore)

- **Product updates:** on price/inventory change (job step or hook), POST
  [`product.updated`](webhooks.md#product-details-webhook).
- **Orders:** on order confirmation, POST
  [`order.completed`](webhooks.md#order-tracking-webhook) with **per-item** Whatmore
  attribution IDs.

## Verify

- Surfaces render in SFRA/PWA and add-to-cart writes to the right basket
- Product Details API returns live price/availability
- Order webhook fires on confirmation with per-item attribution
