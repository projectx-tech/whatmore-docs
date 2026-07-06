# Magento / Adobe Commerce

How a Magento 2 / Adobe Commerce store fulfils the three
[integration tracks](README.md#the-three-integration-tracks). The contracts are the same as
any non-Shopify store — this page maps them onto Magento specifics.

{% hint style="warning" %}
Snippets are **illustrative**. Confirm final shapes jointly.
{% endhint %}

## 1. App SDK — embed the surfaces

Add the [Web SDK](app-sdk.md#web-sdk-headless--non-mobile) via a custom `.phtml` template
or CMS block where you want each surface (PDP, homepage, category):

```html
<div id="whatmore-carousel" data-brand-id="YOUR_BRAND_ID"></div>
<script src="https://cdn.whatmore.ai/sdk.js" async></script>
```

Wire the SDK's `addToCart` callback to Magento's cart (`/rest/V1/carts/mine/items` or the
guest-cart endpoint), and carry the
[Whatmore session IDs](app-sdk.md#session-identity--attribution) onto the cart line so they
survive to checkout.

## 2. Authentication

Issue Magento → Whatmore and Whatmore → Magento keys per
[Authentication](authentication.md). Store the merchant API token used to protect your
Product Details API in Magento's config/secure storage.

## 3a. Product Details API (you host)

Expose [`GET /api/products/{productId}`](product-details-api.md) backed by Magento product
data. `productId` should map to the identifier embedded in your Magento product URL.

- Source fields from the catalog: price (with store-view currency), `is_in_stock` and
  `qty` from stock, media gallery, configurable/variant data.
- Implement as a custom REST endpoint (custom module) or a thin service in front of
  Magento's Catalog APIs.

## 3b. Webhooks (you post to Whatmore)

- **Product updates:** observe price/stock changes (e.g. via Magento events / indexer
  hooks or a scheduled diff) and POST
  [`product.updated`](webhooks.md#product-details-webhook).
- **Orders:** on order completion, POST [`order.completed`](webhooks.md#order-tracking-webhook)
  with **per-item** Whatmore attribution IDs for items that carried them.

## Verify

- SDK surfaces render and add-to-cart writes to the correct Magento cart
- Product Details API returns live price/stock
- Order webhook fires on the confirmation step with per-item attribution
