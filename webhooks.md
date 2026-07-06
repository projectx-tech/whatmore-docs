# Webhooks

**Whatmore hosts these endpoints. Your backend posts to them.** Webhooks keep Whatmore in
step with your store without polling — for product changes, completed orders (attribution),
and optionally cart events.

{% hint style="warning" %}
Endpoints and payloads are **samples for example purposes only** — production may differ
and is confirmed jointly.
{% endhint %}

## Product Details Webhook

A webhook Whatmore **subscribes to** for product-detail changes — primarily **price and
stock-quantity updates** — so tagged products stay accurate without polling.

```jsonc
// POST → https://api.whatmore.ai/webhooks/{brand}/product-updates
{
  "event": "product.updated",
  "productId": "9268",
  "changes": { "price": { "amount": 11.000, "currency": "KWD" }, "inStock": false }
}
```

## Order Tracking Webhook

A webhook Whatmore **subscribes to** for the **order-completion** event, used to attribute
purchases to video-watched sessions.

- Attribution IDs (`whatmore_user_id`, `whatmore_session_id`) are carried at the
  **order-item level, not the whole order** — different items in one order may come from
  different videos/sessions (or none).
- Items are tagged **only where a video was watched**; the IDs originate from the
  [App SDK](app-sdk.md#session-identity--attribution).

```jsonc
// POST → https://api.whatmore.ai/webhooks/{brand}/orders
{
  "event": "order.completed",
  "orderId": "ORD-10293",
  "items": [
    {
      "productId": "9268",
      "quantity": 1,
      "price": 12.500,
      "whatmore_user_id": "<whatmore_user_id>",
      "whatmore_session_id": "<whatmore_session_id>",
      "merchant_session_id": "<merchant_session_id>"
    },
    {
      "productId": "5521",
      "quantity": 2,
      "price": 8.000
    }
  ],
  "total": { "amount": 28.500, "currency": "KWD" }
}
```

Note the second item carries no Whatmore IDs — it was not discovered through a video.

## Cart Tracking Webhook *(optional)*

A webhook for cart events (add / remove / update) to power funnel analytics between video
view and purchase. Same per-item attribution convention as order tracking.
**Nice-to-have, not a blocker.**

## Delivery semantics

> The following are **open points** to agree jointly — see
> [FAQ → Webhooks](faq.md#4-webhooks).

- **Retry policy** — retries with backoff on non-2xx / timeout
- **Timeout** — expected receiver response time before a delivery is considered failed
- **Idempotency** — a stable event id so retried deliveries are de-duplicated by the
  receiver
- **Failure recovery** — how missed events are backfilled after an outage
