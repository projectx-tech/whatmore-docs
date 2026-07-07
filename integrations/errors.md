---
title: "Errors & Conventions"
timestamp: false
---

Conventions that apply to every Core API call — [Authentication](/integrations/authentication),
[Catalog](/integrations/catalog-api), and [Order Tracking](/integrations/order-tracking).

## Base URL & auth

- Base URL: **`https://api.whatmore.live`**.
- Every call carries a bearer token: `Authorization: Bearer <access_token>`.
- The token is **long-lived — it does not expire.** Fetch it once from
  `GET /auth/access-token?store_id=<store_id>`, cache it server-side, and reuse it. There is
  no refresh flow.

## Response conventions

- **Success is any `2xx`.** Write calls (`POST`/`PUT` on catalog, order tracking) return
  **HTTP 200 with an empty body** — `{}`. Don't wait for a payload; treat `200` as done.
- **Reads** return JSON (an object or array) as documented on each endpoint.
- **Prices are strings** (e.g. `"12.500"`), not numbers. `quantity` is an integer.
- The **product identifier must be identical** everywhere — the `client_product_id` you sync
  to the catalog is the same value you send in `order_items[].product_id`. A mismatch means
  the item can't be attributed.

## Status codes

| Code | Meaning |
| ---- | ------- |
| `200` | Success. Writes return `{}`; reads return the documented JSON. |
| `401` | Missing or invalid bearer token. Re-fetch the token and retry. |
| `404` | `Order Id already exists.` — the order was already recorded (see [idempotency](#idempotency--retries)). `Brand is invalid` — the `store_id` isn't recognised. |
| `422` | Request body failed validation — e.g. a missing `client_product_id` on `POST /v2/product`, or a malformed [order-tracking](/integrations/order-tracking) payload. |

Errors are returned in FastAPI's shape: `{ "detail": "<message>" }`.

## Idempotency & retries

Order tracking is **de-duplicated by `order_id`.** Re-sending the same `order_id` is rejected
with **`404 Order Id already exists.`** rather than double-counted, so retries after a network
failure are safe — use a stable `order_id` and treat the duplicate response as success. See
[Order Tracking → idempotency](/integrations/order-tracking#idempotency--retries).

## Rate limits

The default limit is **1,000 requests per minute per store**. This can be increased per client
— ask your Whatmore contact if you expect higher volume.

Because a token is long-lived, cache it rather than re-fetching per call, and push catalog
updates **on change** rather than on a fixed schedule to stay comfortably within the limit.
