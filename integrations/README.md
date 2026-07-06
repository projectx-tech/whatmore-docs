# Overview

Integrate Whatmore's shoppable-video and live-shopping platform into **any** storefront —
native mobile apps, custom / headless sites, and the major commerce platforms. This section
is the technical reference for that integration.

Whatmore's dashboard does the heavy lifting — video hosting, product tagging, campaigns, and
analytics all live there. Your integration is deliberately small, so you go live fast.

## Platform support

| Platform | How you integrate |
| -------- | ----------------- |
| **[Shopify](platform-shopify.md)** | Install the native app — no code |
| **[Magento / Adobe Commerce](platform-magento.md)** | Web widget + Core APIs |
| **[Salesforce Commerce Cloud](platform-sfcc.md)** | Web widget + Core APIs |
| **[WooCommerce](platform-woocommerce.md)** | Web widget + Core APIs |
| **[BigCommerce](platform-bigcommerce.md)** | Web widget + Core APIs |
| **Custom / headless** | [App SDK](app-sdk.md) or web widget + Core APIs |
| **Mobile apps** | [iOS](sdk-ios.md) · [React Native](sdk-react-native.md) · [Android](sdk-android.md) |

## The integration surface

Regardless of platform, an integration is made of three building blocks — you can build them
in parallel:

| Building block | What it does |
| -------------- | ------------ |
| **[App SDK / web widget](app-sdk.md)** | Renders the shoppable-video surfaces in your app or site |
| **[Catalog](catalog-api.md)** | Makes your products available to Whatmore (single, update, or bulk-by-URL) |
| **[Order Tracking](order-tracking.md)** | Reports purchases so Whatmore can attribute them to videos |

Everything else — uploading videos, tagging products to them, building campaigns, viewing
analytics — happens in the **Whatmore dashboard**, not in your code.

## How data flows

You push data to Whatmore; there is no Whatmore-hosted service you must expose an endpoint
for. All calls are authenticated with a [bearer token](authentication.md).

```
  Your systems                                   Whatmore
  ────────────                                   ────────
  catalog        ── POST/PUT /product ─────────► Product catalog
  (products)        POST /product/upload/bulk    (price, stock, media)

  SDK / widget   ── renders surfaces ──────────► shoppable video
  (app or site)  ◄─ emits view / atc signals ──

  order backend  ── POST order-tracking ───────► Attribution
  (on purchase)     (with SDK signals)           (video → sale)
```

## Key concepts

- **`store_id`** — your store's identifier on Whatmore; used to get an access token and on
  every API call.
- **`client_product_id`** — *your* product identifier (commonly the product URL). You
  reference products by it on every call, so Whatmore stays aligned with your catalog
  without a separate mapping layer.
- **Attribution is per order item** — video-view / add-to-cart signals are matched to
  individual line items, so one order can attribute different items to different videos (or
  none).

## Start here

1. **[Getting Started](getting-started.md)** — access token, `store_id`, checklist
2. **[App SDK](app-sdk.md)** — mobile ([iOS](sdk-ios.md) · [React Native](sdk-react-native.md) · [Android](sdk-android.md)) or web widget
3. **Core APIs** — [Authentication](authentication.md) · [Catalog API](catalog-api.md) · [Order Tracking](order-tracking.md)
4. **Your platform** — [Shopify](platform-shopify.md) · [Magento](platform-magento.md) · [SFCC](platform-sfcc.md) · [WooCommerce](platform-woocommerce.md) · [BigCommerce](platform-bigcommerce.md)
