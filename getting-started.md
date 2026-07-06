# Getting Started

Prerequisites and a checklist for integrating a non-Shopify storefront with Whatmore.

{% hint style="warning" %}
URLs and shapes here are **illustrative** — confirmed jointly during integration.
{% endhint %}

## 1. Get access

Your Whatmore contact provisions a **brand** and issues:

- A **Brand ID** — your store's identifier on Whatmore (used by the App SDK).
- A **Whatmore API token** — used by **your** backend to post
  [webhooks](webhooks.md) to Whatmore.
- Access to the Whatmore **dashboard** for uploading and tagging videos.

In the other direction, **you** issue Whatmore a **merchant API token** so Whatmore can
call your [Product Details API](product-details-api.md). See
[Authentication](authentication.md).

## 2. Environments

Test and production use **separate keys and endpoints**.

| Environment | Whatmore webhooks base | Purpose |
| ----------- | ---------------------- | ------- |
| Production  | `https://api.whatmore.ai/webhooks/{brand}/...` | Live traffic |
| Staging     | *confirmed jointly* | Integration testing |

## 3. Integration checklist

- [ ] Brand provisioned; both API tokens exchanged ([Auth](authentication.md))
- [ ] [App SDK](app-sdk.md) surfaces embedded; session IDs flow to checkout
- [ ] [Product Details API](product-details-api.md) live (`GET /api/products/{productId}`)
- [ ] `product.updated` [webhook](webhooks.md#product-details-webhook) posting on price/stock change
- [ ] `order.completed` [webhook](webhooks.md#order-tracking-webhook) posting with **per-item** attribution
- [ ] *(optional)* cart webhook
- [ ] Attribution verified in dashboard

## Mental model

- **No bulk catalog upload** — Whatmore pulls product detail on demand by `productId`
  (a substring of the product URL). See [README](README.md#no-bulk-catalog-upload-required).
- **Traffic is bidirectional** — you host an API Whatmore calls, and you post webhooks
  Whatmore receives.
- **Attribution is per order item** — carry `whatmore_user_id` / `whatmore_session_id`
  from the SDK through to the order webhook.
