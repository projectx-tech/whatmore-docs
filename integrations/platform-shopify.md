# Shopify

Shopify merchants use the **native Whatmore app** — there is no manual API or SDK
integration to build.

{% hint style="success" %}
**Install the app → you're done.**
[**Whatmore on the Shopify App Store**](https://apps.shopify.com/whatmore-live)
{% endhint %}

## What the app handles for you

Once installed, the Whatmore Shopify app wires up everything automatically:

- **Widget embedding** — shoppable-video surfaces render on your storefront via the app's
  theme integration (no manual script).
- **Catalog** — your Shopify products sync automatically; no [Catalog API](catalog-api.md)
  calls needed.
- **Order tracking & attribution** — handled through Shopify's checkout / web-pixel
  integration; no manual [Order Tracking](order-tracking.md) call.

You manage videos, tagging, and campaigns in the **Whatmore dashboard**, exactly as with any
other platform.

## When to use the APIs instead

The [Core APIs](authentication.md) and [App SDK](app-sdk.md) in this section are for
**non-Shopify** storefronts (Magento, SFCC, WooCommerce, BigCommerce, custom / headless, and
mobile apps). If you're on Shopify, you don't need them — install the app above.
