---
title: "Order Tracking"
---

When an order completes, your backend reports it to Whatmore so purchases can be attributed
to the videos that drove them. This is a single authenticated call.

## Endpoint

```http
POST https://api.whatmore.live/external-shop-order-tracking/private?store_id=<store_id>
Authorization: Bearer <access_token>
Content-Type: application/json
```

## Payload

```json
{
  "order_id": "ORD-10293",
  "order_items": [
    { "product_id": "9268", "item_id": "LI-1", "sku": "RHM-75", "price": "12.500", "quantity": 1, "currency": "KWD" },
    { "product_id": "5521", "item_id": "LI-2", "sku": "LIP-02", "price": "8.000",  "quantity": 2, "currency": "KWD" }
  ],
  "whatmore_video_view":  "[{\"product_id\": \"9268\", \"widget_info\": { … }}]",
  "whatmore_add_to_cart": "[{\"product_id\": \"9268\", \"widget_info\": { … }}]"
}
```

| Field | Notes |
| ----- | ----- |
| `order_id` | Your order identifier. Used for idempotency (see below). |
| `order_items[]` | One entry per line item: `product_id`, `item_id`, `sku`, `price` (string), `quantity` (int), `currency`. |
| `whatmore_video_view` | JSON-encoded **string** — a list of `{ product_id, widget_info }` for products watched in a video. On web, the widget stores this in `localStorage._whatmore_viewed_products`. Defaults to `"[]"`. |
| `whatmore_add_to_cart` | JSON-encoded **string** — same shape, for products added to cart from a video. On web, stored in `localStorage._whatmore_add_to_cart_products`. Defaults to `"[]"`. |

<Warning>
The `product_id` you send in `order_items[]` **must be the same identifier your catalog uses
for that product** (your `client_product_id`) — otherwise the item can't be matched to the
video signal.
</Warning>

## Ready-to-use snippet (web)

On a web storefront the video widget records viewed / added-to-cart products into
`localStorage`. Call this once on your order-confirmation page — it reads those signals and
reports the order:

```javascript
async function sendOrderTrackingRequest({ orderId, orderItems, storeId, token }) {
  const url = `https://api.whatmore.live/external-shop-order-tracking/private?store_id=${encodeURIComponent(storeId)}`;
  const payload = {
    order_id: orderId,
    order_items: orderItems,
    whatmore_video_view: localStorage._whatmore_viewed_products || "[]",
    whatmore_add_to_cart: localStorage._whatmore_add_to_cart_products || "[]",
  };
  const res = await fetch(url, {
    method: "POST",
    headers: { "Content-Type": "application/json", Authorization: `Bearer ${token}` },
    body: JSON.stringify(payload),
  });
  if (!res.ok) throw new Error(`Order tracking failed: ${res.status} ${res.statusText}`);
  return res;
}
```

Build `orderItems` from your order (`product_id`, `item_id`, `sku`, `price`, `quantity`,
`currency` per line), then call `sendOrderTrackingRequest({ orderId, orderItems, storeId, token })`.

## How attribution works

- Whatmore matches each **order item** to the SDK signals by `product_id` (your
  `client_product_id`).
- Attribution is therefore **per line item** — items driven by a video are attributed;
  items bought independently are not. A single order can mix both.
- `whatmore_video_view` and `whatmore_add_to_cart` carry the `widget_info` that identifies
  which surface/video is credited.

## Idempotency & retries

- Orders are de-duplicated by `order_id`. If an order is submitted twice, the duplicate is
  rejected with **HTTP 404 (`Order Id already exists`)** rather than double-counted.
- This makes retries safe: re-sending the same `order_id` after a network failure cannot
  create a duplicate. Use a stable `order_id` and treat the duplicate response as success.

<Info>
Send one tracking call per completed order. The `whatmore_video_view` / `whatmore_add_to_cart`
strings are produced by the App SDK and passed through your checkout — see
[App SDK → attribution](/integrations/app-sdk#the-integration-model).
</Info>

## Cart tracking *(optional)*

Cart events can additionally be reported to power funnel analytics between video view and
purchase. Nice-to-have, not required for attribution — scope confirmed during onboarding.
